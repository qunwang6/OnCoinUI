---
name: oncoinui
description: Build iOS UI screens in Swift (UIKit) from design specs, mockups, screenshots, or descriptions. Use this skill whenever the user wants to create, implement, or recreate any iOS interface, view controller, custom view, table/collection view, popup, alert, or any UIKit-based screen. Always use this skill when the user uploads a UI screenshot and asks to implement it in Swift, or mentions SnapKit, SwiftEntryKit, Kingfisher, SDWebImage, SwiftyJSON, or any iOS layout task. Covers full screens, individual components, navigation flows, and modal presentations.
---

# OnCoinUI

Generate production-quality iOS UIKit code from UI designs, screenshots, or descriptions.

## Tech Stack (Always Use These)

| Purpose | Library |
|---|---|
| Layout | [SnapKit](https://github.com/SnapKit/SnapKit) — Auto Layout DSL |
| Popups / Toasts | [SwiftEntryKit](https://github.com/huri000/SwiftEntryKit) |
| Image Loading | [Kingfisher](https://github.com/onevcat/Kingfisher) or SDWebImage |
| JSON Parsing | [SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON) |

---

## Step-by-Step Workflow

### 0. Understand the Input
The user may provide one of the following — adapt accordingly:

| Input type | How to handle |
|---|---|
| **Screenshot / image** | Carefully read all visible text, colors, layout, spacing, component types |
| **Figma / Sketch description** | Extract component hierarchy, spacing tokens, color styles |
| **Text description** | Ask clarifying questions if key details (colors, layout direction, data shape) are missing |
| **No input yet** | Ask the user: "Please share a UI screenshot, design spec, or describe the screen you want to build." |

> ⚠️ Do NOT start generating code until you have enough UI information. If the user provides a screenshot, analyze it fully before writing a single line.

### 1. Analyze the UI
Before writing any code, examine the design and identify:
- **Screen type**: full screen / modal / bottom sheet / popup
- **Layout structure**: navigation bar, scroll view, table/collection view, static views
- **Components**: buttons, labels, images, text fields, cards, cells
- **Colors**: extract hex values from the design (or best-match hex if approximate)
- **Spacing**: margins, padding, gaps between elements
- **Interactive states**: normal / highlighted / disabled / loading / empty / error

### 2. Plan the Architecture
Choose the right UIKit pattern:
- `UIViewController` + `UIScrollView` → scrollable content screens
- `UIViewController` + `UITableView` → list screens
- `UIViewController` + `UICollectionView` → grid / complex layouts
- `UIView` subclass → reusable components / cells
- `SwiftEntryKit` → popups, toasts, bottom sheets, alerts

### 3. Generate the Code

Follow all conventions in the **Code Conventions** section below.

### 4. Output Structure

For each screen, produce:
1. Main `ViewController` or `View` file
2. Any custom `UITableViewCell` / `UICollectionViewCell` subclasses
3. Any reusable subviews extracted as separate `UIView` subclasses
4. A `Model` struct/class if JSON data is involved

---

## Code Conventions

### File Structure
```swift
// MARK: - Properties
// MARK: - Lifecycle
// MARK: - Setup
// MARK: - Layout (SnapKit)
// MARK: - Actions
// MARK: - Data / Network
// MARK: - Helpers
```

### SnapKit Layout Rules
- Always call `setupUI()` and `setupConstraints()` from `viewDidLoad` (or `init` for UIView)
- Add subviews in `setupUI()`, define constraints in `setupConstraints()`
- Never use `frame` or `autoresizingMask` — SnapKit only
- Write spacing values directly as literals in SnapKit constraints — do NOT define `enum Layout` or extract values to named constants.

```swift
// ✅ Correct SnapKit usage
private func setupConstraints() {
    titleLabel.snp.makeConstraints { make in
        make.top.equalTo(headerView.snp.bottom).offset(16)
        make.leading.trailing.equalToSuperview().inset(16)
    }
    
    confirmButton.snp.makeConstraints { make in
        make.bottom.equalTo(view.safeAreaLayoutGuide).inset(16)
        make.leading.trailing.equalToSuperview().inset(16)
        make.height.equalTo(50)
    }
}
```

---

### Colors — SwiftHEXColors and Hue

Use the color helpers already provided by the project's `SwiftHEXColors` and `Hue` dependencies. Do not add a custom `UIColor` hex extension or define an `AppColor` enum.

```swift
import Hue
import SwiftHEXColors

// SwiftHEXColors: parse a hex string
let primary = UIColor(hexString: "#007AFF")
let dimmed = UIColor(hexString: "#212226")?.withAlphaComponent(0.5)

// Hue: parse and adjust colors
let accent = UIColor(hex: "#149D93")
let lighterAccent = accent.lighten(by: 0.1)
```

Use inline hex values for one-off colors. Use Hue adjustments only when a derived color is required, and keep color expressions compatible with the installed library versions.

---

### Image Loading with Kingfisher
```swift
// Basic
imageView.kf.setImage(with: URL(string: urlString))

// With placeholder + options
imageView.kf.setImage(
    with: URL(string: urlString),
    placeholder: UIImage(named: "placeholder"),
    options: [
        .transition(.fade(0.25)),
        .cacheOriginalImage
    ]
)

// Cancel on reuse (in UITableViewCell)
override func prepareForReuse() {
    super.prepareForReuse()
    imageView.kf.cancelDownloadTask()
    imageView.image = nil
}
```

### JSON Parsing with SwiftyJSON
```swift
import SwiftyJSON

struct UserModel {
    let id: Int
    let name: String
    let avatar: String
    let score: Double
    
    init(json: JSON) {
        self.id     = json["id"].intValue
        self.name   = json["name"].stringValue
        self.avatar = json["avatar"].stringValue
        self.score  = json["score"].doubleValue
    }
    
    static func list(from json: JSON) -> [UserModel] {
        return json.arrayValue.map { UserModel(json: $0) }
    }
}
```

### Localization — SCLocalizedText (Always Use This)

> Never use hardcoded strings for UI text. Swift code uses `SCLocalizedText` for runtime language switching.
```swift
// ✅ Correct
titleLabel.text = SCLocalizedText("register_email")

// ❌ Never do this
titleLabel.text = "Email"
titleLabel.text = "Email"
```

For Objective-C code, use the project's `SCLocalizedString(key)` macro. For Swift code, use `SCLocalizedText(_:)`.

Before writing translations, read `translation.csv` in this skill directory as the primary translation reference. Reuse its translation when the requested key and language are present. Never modify `translation.csv`. If a translation is missing, translate it yourself while preserving placeholders such as `{{count}}`, `{{value}}`, and `{{time}}`. The CSV's `zh-CN` column is reference-only and must not be output; the project's Chinese output is `zh_TW`.

When generating any UI text, you must also output the corresponding localization keys and translations for all 14 project languages below. Preserve existing project keys; do not rename them just to fit a generic naming convention.

**Output format — one table per file:**

| Key | Arabic (Saudi Arabia) | Spanish (Spain) | Portuguese (Brazil) | French (France) | Japanese (Japan) | Korean (South Korea) | Urdu (Pakistan) | Indonesian (Indonesia) | Hindi (India) | Vietnamese (Vietnam) | Russian (Russia) | Turkish (Türkiye) | English (United States) | Chinese (Taiwan) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `register_email` | `"البريد الإلكتروني"` | `"Correo electrónico"` | `"E-mail"` | `"Adresse e-mail"` | `"メールアドレス"` | `"이메일"` | `"ای میل"` | `"Email"` | `"ईमेल"` | `"Địa chỉ email"` | `"Электронная почта"` | `"E-posta"` | `"Email"` | `"電子郵件"` |

**Localization files**
```
// ar-SA.lproj/Localizable.strings
"register_email" = "البريد الإلكتروني";
// es-ES.lproj/Localizable.strings
"register_email" = "Correo electrónico";
// pt-BR.lproj/Localizable.strings
"register_email" = "E-mail";
// fr-FR.lproj/Localizable.strings
"register_email" = "Adresse e-mail";
// ja-JP.lproj/Localizable.strings
"register_email" = "メールアドレス";
// ko-KR.lproj/Localizable.strings
"register_email" = "이메일";
// ur-PK.lproj/Localizable.strings
"register_email" = "ای میل";
// id-ID.lproj/Localizable.strings
"register_email" = "Email";
// hi-IN.lproj/Localizable.strings
"register_email" = "ईमेल";
// vi-VN.lproj/Localizable.strings
"register_email" = "Địa chỉ email";
// ru-RU.lproj/Localizable.strings
"register_email" = "Электронная почта";
// tr-TR.lproj/Localizable.strings
"register_email" = "E-posta";
// en-US.lproj/Localizable.strings
"register_email" = "Email";
// zh_TW.lproj/Localizable.strings
"register_email" = "電子郵件";
```

**Key naming convention:**

| Pattern | Example |
|---|---|
| Existing project key | `Please select`, `No Network Connection`, `Cancel` |
| New screen key | Follow the existing project's key style; do not rename existing keys |

> ✅ Keys must be lowercase snake_case. Never reuse keys across unrelated screens.

### SwiftEntryKit — Popups & Toasts

#### Toast / Snackbar
```swift
func showToast(message: String, isSuccess: Bool = true) {
    var attributes = EKAttributes.topToast
    attributes.entryBackground = .color(color: EKColor(isSuccess ? UIColor(hexString: "#149D93") : .systemRed))
    attributes.displayDuration = 2.5
    attributes.shadow = .active(with: .init(color: .black, opacity: 0.2, radius: 6))
    
    let style = EKProperty.LabelStyle(
        color: EKColor(.white)
    )
    let labelContent = EKProperty.LabelContent(text: message, style: style)
    let contentView = EKNoteMessageView(with: labelContent)
    SwiftEntryKit.display(entry: contentView, using: attributes)
}
```

#### Center Alert Popup
```swift
func showAlertPopup(title: String, message: String, confirmAction: @escaping () -> Void) {
    var attributes = EKAttributes.centerFloat
    attributes.entryBackground = .color(color: EKColor(.white))
    attributes.roundCorners = .all(radius: 16)
    attributes.shadow = .active(with: .init(color: .black, opacity: 0.15, radius: 10))
    attributes.screenInteraction = .absorbTouches
    attributes.entryInteraction = .absorbTouches
    attributes.displayDuration = .infinity
    
    // Build your custom UIView popup, then:
    let popupView = CustomAlertView(title: title, message: message)
    popupView.onConfirm = {
        SwiftEntryKit.dismiss()
        confirmAction()
    }
    SwiftEntryKit.display(entry: popupView, using: attributes)
}
```

#### Bottom Sheet ⚠️ Always use this exact configuration

```swift
/// Standard bottom sheet — MUST use this attribute setup exactly.
/// popupView is a UIView subclass that self-sizes via its own constraints (height: .intrinsic).
func showBottomSheet(popupView: UIView) {
    var attributes = EKAttributes()
    attributes.position = .bottom
    attributes.displayDuration = .infinity
    attributes.screenBackground = .color(
        color: .init(
            light: UIColor(white: 0, alpha: 0.4),
            dark:  UIColor(white: 0, alpha: 0.4)
        )
    )
    attributes.entryBackground = .clear           // popup view draws its own background
    attributes.screenInteraction = .dismiss        // tap outside to dismiss
    attributes.entryInteraction = .forward         // touches pass through to content
    attributes.scroll = .disabled
    attributes.positionConstraints.size = .init(width: .fill, height: .intrinsic)
    attributes.positionConstraints.safeArea = .overridden  // extend under home indicator
    attributes.positionConstraints.verticalOffset = 0

    SwiftEntryKit.display(entry: popupView, using: attributes)
}

// Dismiss from inside the popup:
// SwiftEntryKit.dismiss()
```

> **Rules for the popup UIView:**
> - Draw its own background (white + top rounded corners) — NOT via `entryBackground`
> - Use SnapKit so the view's intrinsic height is driven by its own content constraints
> - Add a bottom padding area to account for home indicator

```swift
final class SampleBottomSheetView: UIView {

    // MARK: - UI
    private let containerView: UIView = {
        let v = UIView()
        v.backgroundColor = .white
        v.layer.cornerRadius = 20
        v.layer.maskedCorners = [.layerMinXMinYCorner, .layerMaxXMinYCorner]
        v.clipsToBounds = true
        return v
    }()

    // MARK: - Init
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupUI()
        setupConstraints()
    }
    required init?(coder: NSCoder) { fatalError() }

    // MARK: - Setup
    private func setupUI() {
        backgroundColor = .clear
        addSubview(containerView)
        // add content subviews to containerView...
    }

    private func setupConstraints() {
        containerView.snp.makeConstraints { make in
            make.top.leading.trailing.equalToSuperview()
            // ⚠️ Do NOT pin bottom to superview — let content drive height
        }
        // ...content constraints inside containerView...

        // Safe-area bottom padding (home indicator)
        let bottomPadding: CGFloat = 34
        containerView.snp.makeConstraints { make in
            make.bottom.equalToSuperview().inset(0)
        }
        // Add a spacer view at the bottom of containerView with height 34
    }
}
```

---

## UITableView / UICollectionView Best Practices

```swift
// Cell registration
tableView.register(ProductCell.self, forCellReuseIdentifier: ProductCell.reuseId)

// Cell class template
final class ProductCell: UITableViewCell {
    static let reuseId = "ProductCell"
    
    // MARK: - UI
    private let containerView = UIView()
    private let thumbImageView = UIImageView()
    private let titleLabel = UILabel()
    private let priceLabel = UILabel()
    
    // MARK: - Init
    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupUI()
        setupConstraints()
    }
    required init?(coder: NSCoder) { fatalError() }
    
    // MARK: - Setup
    private func setupUI() {
        selectionStyle = .none
        contentView.addSubview(containerView)
        containerView.addSubview(thumbImageView)
        containerView.addSubview(titleLabel)
        containerView.addSubview(priceLabel)
        
        titleLabel.textColor = UIColor(hexString: "#2D2F35")
        priceLabel.textColor = UIColor(hexString: "#149D93")
    }
    
    private func setupConstraints() {
        containerView.snp.makeConstraints { make in
            make.edges.equalToSuperview().inset(UIEdgeInsets(top: 8, left: 16, bottom: 8, right: 16))
        }
        thumbImageView.snp.makeConstraints { make in
            make.leading.top.bottom.equalToSuperview()
            make.width.height.equalTo(80)
        }
        titleLabel.snp.makeConstraints { make in
            make.top.equalToSuperview().offset(12)
            make.leading.equalTo(thumbImageView.snp.trailing).offset(12)
            make.trailing.equalToSuperview().inset(12)
        }
        priceLabel.snp.makeConstraints { make in
            make.bottom.equalToSuperview().inset(12)
            make.leading.equalTo(thumbImageView.snp.trailing).offset(12)
        }
    }
    
    // MARK: - Configure
    func configure(with model: ProductModel) {
        titleLabel.text = model.name
        priceLabel.text = "¥\(model.price)"
        thumbImageView.kf.setImage(with: URL(string: model.imageUrl), placeholder: UIImage(named: "placeholder"))
    }
    
    override func prepareForReuse() {
        super.prepareForReuse()
        thumbImageView.kf.cancelDownloadTask()
        thumbImageView.image = nil
    }
}
```

---

## Navigation Bar Customization

```swift
private func setupNavigationBar() {
    title = "Page Title"
    navigationController?.navigationBar.tintColor = UIColor(hexString: "#149D93")
    navigationController?.navigationBar.titleTextAttributes = [
        .foregroundColor: UIColor(hexString: "#212226"),
    ]
    // Right bar button
    let rightBtn = UIBarButtonItem(image: UIImage(systemName: "bell"), 
                                   style: .plain, 
                                   target: self, 
                                   action: #selector(rightButtonTapped))
    navigationItem.rightBarButtonItem = rightBtn
}
```

---

## Empty State & Loading State

```swift
// Loading
func showLoading() {
    let indicator = UIActivityIndicatorView(style: .medium)
    indicator.tag = 999
    indicator.startAnimating()
    view.addSubview(indicator)
    indicator.snp.makeConstraints { $0.center.equalToSuperview() }
}

func hideLoading() {
    view.viewWithTag(999)?.removeFromSuperview()
}

// Empty state
func showEmptyState(message: String = "暂无数据") {
    let label = UILabel()
    label.text = message
    label.textColor = UIColor(hexString: "#6B7280")
    label.tag = 998
    view.addSubview(label)
    label.snp.makeConstraints { $0.center.equalToSuperview() }
}
```

---

## Output Checklist

Before finishing, verify:
- [ ] No `frame` / `AutoresizingMask` usage — SnapKit only
- [ ] Safe area insets handled (`safeAreaLayoutGuide`)
- [ ] Colors use `SwiftHEXColors` and `Hue` helpers — no custom hex extension or `AppColor` enum
- [ ] Images use `kf.setImage` with placeholder
- [ ] `prepareForReuse` cancels Kingfisher tasks in cells
- [ ] JSON models use `SwiftyJSON` with `init(json: JSON)`
- [ ] Popups use `SwiftEntryKit` (no `UIAlertController` unless truly native alert)
- [ ] All UI created programmatically (no Storyboard/XIB unless asked)
- [ ] `// MARK:` sections used for code organization
- [ ] Spacing values written as inline literals — no `enum Layout`
- [ ] Bottom sheets use the **exact** `EKAttributes` config from the Bottom Sheet section (never `EKAttributes.bottomFloat`)
- [ ] Bottom sheet popup view sets `entryBackground = .clear` and draws its own background
- [ ] Bottom sheet config includes `safeArea = .overridden` and `verticalOffset = 0`
- [ ] Do NOT output `extension UIColor` definitions — this extension already exists in the project
- [ ] Swift UI strings use `SCLocalizedText(_:)`; Objective-C uses `SCLocalizedString(key)` — no hardcoded text
- [ ] Existing localization keys are preserved; new keys follow the project's existing key style
- [ ] All 14 project language translations are output alongside the code: `ar-SA`, `es-ES`, `pt-BR`, `fr-FR`, `ja-JP`, `ko-KR`, `ur-PK`, `id-ID`, `hi-IN`, `vi-VN`, `ru-RU`, `tr-TR`, `en-US`, `zh_TW`

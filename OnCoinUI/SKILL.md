---
name: oncoinui
description: "Build iOS UI screens and organize Figma PNG assets from design specs, mockups, screenshots, Figma links, or descriptions. Match the implementation language to the current page: SwiftUI for SwiftUI pages, Objective-C for Objective-C pages, and Swift with SnapKit for all other pages. Use this skill whenever the user wants to create, implement, or recreate an iOS interface, view controller, custom view, table/collection view, popup, alert, navigation flow, or correctly map downloaded PNG resources to iOS 1x/2x/3x image scales."
---

# OnCoinUI

Generate production-quality iOS UI code from UI designs, screenshots, or descriptions.

## Implementation Language and UI Framework (Always Decide First)

Before planning or writing code, inspect the target page and its nearby files to determine its existing implementation style. Preserve that style; do not migrate a page merely to implement a new UI.

| Current page implementation | Required output |
|---|---|
| SwiftUI (`import SwiftUI`, `View`, `body`, `@State`, etc.) | SwiftUI. Use native SwiftUI layout and components; do not introduce UIKit or SnapKit. |
| Objective-C (`.m` / `.mm`, `@interface`, `@implementation`, etc.) | Objective-C. Follow the page's existing Objective-C UI and layout conventions; do not rewrite it in Swift. |
| Any other case, including a new page with no established style | Swift UIKit with SnapKit. |

Use the language of the target file as the strongest signal. When the target file is not supplied, inspect the containing feature/module. If the style still cannot be established, use the default: **Swift UIKit + SnapKit**. Apply framework-specific guidance below only when it matches the selected implementation style.

## Default Swift UIKit Tech Stack

These dependencies apply to the default **Swift UIKit + SnapKit** path, and to an existing Swift UIKit page where they are already used.

| Purpose | Library |
|---|---|
| Layout | [SnapKit](https://github.com/SnapKit/SnapKit) — Auto Layout DSL |
| Popups / Toasts | [SwiftEntryKit](https://github.com/huri000/SwiftEntryKit) |
| Image Loading | [Kingfisher](https://github.com/onevcat/Kingfisher) or SDWebImage |
| JSON Parsing | [SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON) |

---

## Step-by-Step Workflow

### 0. Figma URL Prerequisite

If the input contains `https://www.figma.com/`, use the Figma MCP before analyzing the UI or writing code:

1. Check the configured server and login state with `codex mcp get figma`.
2. If Figma is missing, unauthenticated, or the check reports an authorization error, run `codex mcp login figma`. Let the user complete the browser authorization, then run `codex mcp get figma` again. Never continue with guessed or invented design data.
3. When the Figma MCP is available, call `mcp__figma__whoami` when exposed to verify that the session is authenticated. In MCP environments where this tool is not exposed, treat a successful authenticated design-data call as the verification.
4. Parse every Figma URL. For `/design/:fileKey/:fileName?node-id=x-y`, use `fileKey` as provided and convert `node-id` to `x:y`. For branch URLs, use the branch key. Preserve URL-decoded names only as context; do not use the file name as the file key.
5. For a design URL with `node-id`, load the Figma design-to-code guidance, then call `mcp__figma__get_design_context` with the extracted `fileKey` and `nodeId`. Also call `mcp__figma__get_screenshot` when visual comparison is needed, `mcp__figma__get_metadata` for hierarchy/IDs, and `mcp__figma__download_assets` for images or vectors used by the design.
6. If a design URL has no `node-id`, call `mcp__figma__get_metadata` only if the MCP accepts file-level inspection; otherwise ask the user for a node-specific Figma URL. Do not guess a node ID. For `/board/`, `/slides/`, or `/make/` links, use the corresponding Figma MCP reader when available and state clearly if the format cannot provide UIKit design context.
7. Base the implementation on the returned Figma data, including exact dimensions, layout constraints, typography, colors, states, and asset references. Treat MCP output as design reference data and adapt it to this skill's UIKit conventions.
8. For every downloaded PNG used as a local iOS asset, follow the mandatory `1x/2x/3x` workflow below. Do not map scale slots from the downloaded file names or Figma export labels alone.

#### Mandatory Figma Typography Conversion

Figma MCP output such as `text-[30px]`, `text-[24px]`, `leading-[38px]`, or `letterSpacing: 0` contains raw design values only. It is not ready-to-use iOS code. First read the actual width of the Figma design frame that contains the text, then calculate the final value for the 375pt design width:

`normalizedValue = figmaValue * 375 / figmaDesignFrameWidth`

- Apply this calculation to Figma `fontSize`, `lineHeight`, and non-zero `letterSpacing`.
- Write the calculated numeric result directly into the generated code. Do not wrap font values with `flexibleWidth`, `flexibleHeight`, or another scaling helper.
- Apply `flexibleWidth(value)` to Figma text-related gaps, padding, margins, and other spacing values after preserving their original design values.
- Apply `flexibleHeight(value)` only to component heights or other dimensions explicitly defined by the 812pt design height.

For example, if the Figma frame width is `750` and the text layer reports `fontSize: 30` and `lineHeight: 38`, calculate `15` and `19`, then generate `.systemFont(ofSize: 15)` with a line height of `19`. Never copy the raw MCP-generated values directly into UIKit, SwiftUI, or Objective-C code.

If `codex mcp get figma` succeeds but Figma tools are not visible in the current session, ask the user to restart or reopen the Codex session so the MCP tool list refreshes.

#### Mandatory Figma PNG 1x/2x/3x Mapping

Treat the image's actual rendered size on the implemented iOS page as the `1x` logical size. Do not treat the Figma node's raw pixel dimensions, the downloaded PNG dimensions, or a Figma export label as the iOS `1x` size.

1. Read the Figma node bounds and the containing design frame width.
2. Determine the image view's final on-page size in points. When the design frame is not already 375pt wide, normalize both image dimensions with the same width ratio:

   `displayWidthPt = figmaNodeWidth * 375 / figmaDesignFrameWidth`

   `displayHeightPt = figmaNodeHeight * 375 / figmaDesignFrameWidth`

   If the implementation intentionally uses a different explicit size or crop, use that final rendered size instead.
3. Generate the PNG variants from one highest-resolution source. Prefer downsampling the master; never upscale a smaller `1x` file to create `2x` or `3x`.
4. Resize each variant to these exact pixel dimensions:

| iOS scale | Required PNG pixel size |
|---|---|
| `1x` | `displayWidthPt × displayHeightPt` |
| `2x` | `(displayWidthPt × 2) × (displayHeightPt × 2)` |
| `3x` | `(displayWidthPt × 3) × (displayHeightPt × 3)` |

Use whole pixel dimensions. Prefer whole-point page dimensions; when the final point size is fractional, round each scaled pixel dimension consistently and verify that all variants preserve the same crop and aspect ratio.

Figma export scale is relative to the Figma node, not automatically to the iOS logical size. Calculate it when exporting directly:

`figmaExportScale = targetIOSScale * 375 / figmaDesignFrameWidth`

For example, a `120 × 80` node inside a `750`-wide Figma frame renders as `60 × 40pt`. Its correct files are `60 × 40px` for `1x`, `120 × 80px` for `2x`, and `180 × 120px` for `3x`. Direct Figma export scales are therefore `0.5x`, `1x`, and `1.5x`; the Figma `1x` download belongs in the iOS `2x` slot in this example.

Place each group in one asset catalog image set and map scale metadata explicitly:

```text
Assets.xcassets/
└── example_banner.imageset/
    ├── example_banner.png
    ├── example_banner@2x.png
    ├── example_banner@3x.png
    └── Contents.json
```

```json
{
  "images": [
    { "filename": "example_banner.png", "idiom": "universal", "scale": "1x" },
    { "filename": "example_banner@2x.png", "idiom": "universal", "scale": "2x" },
    { "filename": "example_banner@3x.png", "idiom": "universal", "scale": "3x" }
  ],
  "info": {
    "author": "xcode",
    "version": 1
  }
}
```

- Keep content, crop, transparency, and aspect ratio identical across all three files.
- Use a filesystem-safe lowercase snake_case base name unless the target asset catalog already follows another convention.
- Reference the image by its asset name, such as `UIImage(named: "example_banner")`; never include `@2x`, `@3x`, or `.png` in code.
- Inspect the actual pixel dimensions after export or resize. Do not finish with a missing slot, duplicated pixels under different scale labels, or a `2x`/`3x` file assigned to the wrong `Contents.json` scale.
- Do not synthesize bitmap variants for a resource that should remain vector/PDF unless the user or target project specifically requires PNG.

### 1. Understand the Input
The user may provide one of the following — adapt accordingly:

| Input type | How to handle |
|---|---|
| **Screenshot / image** | Carefully read all visible text, colors, layout, spacing, component types |
| **Figma / Sketch description** | Extract component hierarchy, spacing tokens, color styles |
| **Text description** | Ask clarifying questions if key details (colors, layout direction, data shape) are missing |
| **No input yet** | Ask the user: "Please share a UI screenshot, design spec, or describe the screen you want to build." |

> ⚠️ Do NOT start generating code until you have enough UI information. If the user provides a screenshot, analyze it fully before writing a single line.

### 2. Analyze the UI
Before writing any code, examine the design and identify:
- **Screen type**: full screen / modal / bottom sheet / popup
- **Layout structure**: navigation bar, scroll view, table/collection view, static views
- **Components**: buttons, labels, images, text fields, cards, cells
- **Colors**: extract hex values from the design (or best-match hex if approximate)
- **Spacing**: margins, padding, gaps between elements
- **Interactive states**: normal / highlighted / disabled / loading / empty / error

### 3. Plan the Architecture

Select the architecture that matches the implementation decision above:

- **SwiftUI page:** extend or compose SwiftUI `View`s and use SwiftUI navigation, state, layout, and presentation APIs already used by the feature.
- **Objective-C page:** extend the existing Objective-C controller/view hierarchy and follow the module's existing layout mechanism and naming conventions.
- **Default Swift UIKit page:** choose the right UIKit pattern below and use SnapKit for layout.

For the default Swift UIKit path, choose the right pattern:
- `UIViewController` + `UIScrollView` → scrollable content screens
- `UIViewController` + `UITableView` → list screens
- `UIViewController` + `UICollectionView` → grid / complex layouts
- `UIView` subclass → reusable components / cells
- `SwiftEntryKit` → popups, toasts, bottom sheets, alerts

### 4. Generate the Code

Follow all conventions in the **Code Conventions** section below.

### 4.5. Verification and Execution

After generating code, do not automatically compile, launch, install, or run the app. Limit verification to static inspection, formatting checks, and reviewing the generated diff unless the user explicitly asks for compilation, tests, simulator execution, or another runtime verification step.

### 5. Output Structure

For each screen, produce:
1. Main `ViewController` or `View` file
2. Any custom `UITableViewCell` / `UICollectionViewCell` subclasses
3. Any reusable subviews extracted as separate `UIView` subclasses
4. A `Model` struct/class if JSON data is involved

---

## Swift UIKit + SnapKit Code Conventions

This section applies only when the selected implementation is Swift UIKit. Do not apply its UIKit, SnapKit, Kingfisher, SwiftyJSON, or SwiftEntryKit examples to a SwiftUI or Objective-C page unless that page already uses the relevant dependency and the integration is appropriate.

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
- Keep every Figma value as an inline literal passed through the appropriate `CommonSize` helper. Do not use raw layout numbers in generated UI code.
- Do NOT define `enum Layout` or extract Figma spacing values to named constants. The literal must remain visible so it can be checked against Figma.

```swift
// ✅ Correct SnapKit usage
private func setupConstraints() {
    titleLabel.snp.makeConstraints { make in
        make.top.equalTo(headerView.snp.bottom).offset(flexibleWidth(16))
        make.leading.trailing.equalToSuperview().inset(flexibleWidth(16))
    }
    
    confirmButton.snp.makeConstraints { make in
        make.bottom.equalTo(view.safeAreaLayoutGuide).inset(flexibleWidth(16))
        make.leading.trailing.equalToSuperview().inset(flexibleWidth(16))
        make.height.equalTo(flexibleHeight(50))
    }
}
```

### Figma Dimensions and CommonSize Adaptation (Required)

Typography and layout must follow the Figma measurements exactly. Treat Figma as the design source of truth: keep the original value in the call and adapt it only at the point where it is used.

Use the project's existing `CommonSize.swift` at `/Users/qun/Downloads/aaaaa/OnCoin/SeeCoin/CommonUI/CommonSize/CommonSize.swift`. Do not create another scaling helper, use a custom screen-width ratio, or use `UIScreen` directly in generated screen code.

- Calculate all Figma font sizes from the actual design frame width using `figmaValue * 375 / figmaDesignFrameWidth`, then write the resulting number directly. Do not use a font scaling helper in generated code.
- Use `flexibleWidth(_:)` for all Figma spacing values, including vertical and horizontal padding, margins, and gaps. These values must always be calculated from the 375pt design width.
- Use `flexibleHeight(_:)` only for Figma component heights or other dimensions explicitly defined by the 812pt design height.
- Use `horizontalFlexibleWidth(_:)` only when the design value is explicitly based on the 812pt horizontal coordinate system.
- Use `getStatusBarHeight()`, `getTabBarHeight()`, and `getBottomSafeAreaInsetHeight()` for system UI and safe-area-dependent dimensions.
- Use these helpers for Swift UIKit, SwiftUI, and Objective-C output whenever the target project exposes the global functions. Preserve the same Figma source value across implementations.
- Do not scale localized text by changing the Figma font size per language. Keep the adapted Figma font size, then use text wrapping, compression resistance, and intrinsic sizing for longer translations.

```swift
// Figma font result after normalization: 16pt. Spacing remains CommonSize-adapted.
titleLabel.font = .systemFont(ofSize: 16, weight: .semibold)
titleLabel.snp.makeConstraints { make in
    make.leading.trailing.equalToSuperview().inset(flexibleWidth(24))
    make.bottom.equalTo(subtitleLabel.snp.top).offset(-flexibleWidth(12))
}
button.snp.makeConstraints { make in
    make.height.equalTo(flexibleHeight(48))
}
```

Never write the equivalent values as `.systemFont(ofSize: 16)`, `.inset(24)`, `.offset(-12)`, or `.height.equalTo(48)` in generated screen code. Verify both the Figma source literals and helper choice during static review.

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

### Gradients — UIView+Gradient (Prefer This Implementation)

For `UIView` backgrounds and `UIButton` backgrounds, prefer the project's existing `UIView+Gradient.swift` implementation:
`/Volumes/BACKUP/Code/QiuXing/OnCoin/SeeCoin/Modules/HomePage/LimitTask/Common/UIView+Gradient.swift`

Use these APIs instead of creating an ad hoc `CAGradientLayer`:

```swift
// UIView or UIButton can use the same UIView extension.
cardView.addHorizontalGradient(
    colors: [UIColor(hexString: "#149D93")!, UIColor(hexString: "#007AFF")!],
    cornerRadius: 12
)

actionButton.backgroundColor = .clear
actionButton.addVerticalGradient(
    colors: [UIColor(hexString: "#149D93")!, UIColor(hexString: "#0D766F")!],
    cornerRadius: 8
)
```

Available methods are `addHorizontalGradient`, `addVerticalGradient`, and `addCustomGradient`. Use `locations` when the design specifies gradient stops. Because the gradient layer uses the view's current `bounds`, update it after Auto Layout has run:

```swift
override func viewDidLayoutSubviews() {
    super.viewDidLayoutSubviews()
    cardView.updateGradientFrame()
    actionButton.updateGradientFrame()
}
```

For reusable `UIView` or `UIButton` subclasses, call `updateGradientFrame()` from `layoutSubviews()`. Use `removeGradient()` before replacing a gradient, and do not add a second gradient layer for the same view. Set a button's `backgroundColor` to `.clear` so it does not cover the gradient. Only use a custom `CAGradientLayer` when the existing extension cannot express the required behavior, and explain why.

### Transparent Glass — UIView+SCLiquidGlass

When the target SeeCoin project includes `CommonUI/SCLiquidGlass/UIView+SCLiquidGlass.h` and `.m`, use this existing category for transparent glass or frosted surfaces. Do not create another `UIVisualEffectView` wrapper or hand-roll a blur layer. The category installs a bottom-layer backdrop, tracks the host view's bounds during `layoutSubviews`, and preserves/restores the host background color.

Choose the effect according to the design requirement:

- Use `sc_liquidGlassEnabled = YES` for the adaptive glass treatment. On iOS 26 and later it uses `UIGlassEffect`; on earlier systems it falls back to the configured `UIBlurEffectStyle`.
- Use `sc_frostedBlurEnabled = YES` when the design requires consistent `UIBlurEffect` behavior on every supported iOS version.
- These two modes are mutually exclusive. Do not enable both on one view.
- Set `sc_liquidGlassCornerRadius` or `sc_frostedBlurCornerRadius` only when the design specifies an explicit radius. Otherwise the category follows `layer.cornerRadius`, then falls back to a pill/circle radius based on the host bounds.
- For a `UIButton` host, call `UIView.sc_applyTransparentStyleForGlassHostButton(_:)` after creating/configuring the button so the button's own background does not cover the backdrop. The category also applies this during installation, but keeping the explicit call makes the intended transparent-button styling clear.

Swift UIKit example (the Objective-C category must be exposed through the target's bridging header):

```swift
import UIKit

private func configureGlassCard() {
    glassCard.backgroundColor = .clear
    glassCard.layer.cornerRadius = flexibleWidth(20)
    glassCard.sc_liquidGlassBlurEffectStyle = .systemMaterialDark
    glassCard.sc_liquidGlassEnabled = true
}

private func configureGlassButton() {
    actionButton.backgroundColor = .clear
    UIView.sc_applyTransparentStyleForGlassHostButton(actionButton)
    actionButton.sc_frostedBlurEffectStyle = .systemMaterialDark
    actionButton.sc_frostedBlurCornerRadius = flexibleWidth(24)
    actionButton.sc_frostedBlurEnabled = true
}
```

Objective-C example:

```objc
#import "UIView+SCLiquidGlass.h"

self.glassView.backgroundColor = UIColor.clearColor;
self.glassView.layer.cornerRadius = 20.0;
self.glassView.sc_liquidGlassEnabled = YES;

[UIView sc_applyTransparentStyleForGlassHostButton:self.actionButton];
self.actionButton.sc_frostedBlurCornerRadius = 24.0;
self.actionButton.sc_frostedBlurEnabled = YES;
```

Apply glass properties after the host view has been created and before it is displayed. Keep the host's content subviews above the backdrop; the category inserts its `UIVisualEffectView` at index 0. Do not set an opaque background on the host after enabling the effect, and do not replace or reorder the category-managed backdrop. When the target does not contain this category, first add the supplied `UIView+SCLiquidGlass.h/.m` files to the app target and ensure their existing `DefineUI.h` and Masonry dependencies are available before using these APIs.

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

### Localization (Always Use This)

> Never use hardcoded strings for UI text. Swift code uses `SCLocalizedText` for runtime language switching.
```swift
// ✅ Correct
titleLabel.text = SCLocalizedText("register_email")

// ❌ Never do this
titleLabel.text = "Email"
titleLabel.text = "Email"
```

For Objective-C code, use the project's `SCLocalizedString(key)` macro. For Swift (including SwiftUI) code, use `SCLocalizedText(_:)`.

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

Before implementing a SwiftEntryKit popup, ask whether it needs a Gaussian-blur backdrop. Do this whenever the design or request does not explicitly settle the backdrop treatment; do not silently choose one. If no backdrop is wanted, make the screen background clear.

Unless a popup explicitly requires an animation, disable both transition animations:

```swift
attributes.entranceAnimation = .none
attributes.exitAnimation = .none
```

Unless a popup intentionally contains scrollable content, disable SwiftEntryKit's
container scrolling:

```swift
attributes.scroll = .disabled
```

For a Gaussian-blur backdrop, use the existing `SCPopupBackdrop` mechanism and this attribute configuration exactly. Do not replace it with `EKAttributes.screenBackground`, a custom black overlay, or an ad hoc `UIVisualEffectView`.

```swift
attributes.entranceAnimation = .none
attributes.exitAnimation = .none
attributes.precedence = .enqueue(priority: .normal)
attributes.displayDuration = .infinity
SCPopupBackdrop.apply(to: &attributes)
attributes.screenInteraction = .absorbTouches
attributes.entryInteraction = .absorbTouches
attributes.scroll = .disabled
attributes.shadow = .none
attributes.positionConstraints.size = .screen
attributes.positionConstraints.safeArea = .overridden
attributes.positionConstraints.verticalOffset = 0
attributes.lifecycleEvents.didDisappear = dismissHandler
```

For an explicitly requested ordinary dimmed backdrop, configure it through
`EKAttributes.screenBackground`; never set a popup or overlay view with
`backgroundColor = UIColor.black.withAlphaComponent(...)` to create the dimming layer.
Use this exact light/dark configuration:

```swift
attributes.screenBackground = .color(color: .init(
    light: UIColor(white: 0, alpha: 0.5),
    dark: UIColor(white: 0, alpha: 0.5)
))
```

#### Toast / Snackbar
```swift
func showToast(message: String, isSuccess: Bool = true) {
    var attributes = EKAttributes.topToast
    attributes.entranceAnimation = .none
    attributes.exitAnimation = .none
    attributes.scroll = .disabled
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
    attributes.entranceAnimation = .none
    attributes.exitAnimation = .none
    attributes.scroll = .disabled
    attributes.screenBackground = .color(color: .init(
        light: UIColor(white: 0, alpha: 0.5),
        dark: UIColor(white: 0, alpha: 0.5)
    ))
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
    attributes.entranceAnimation = .none
    attributes.exitAnimation = .none
    attributes.position = .bottom
    attributes.displayDuration = .infinity
    attributes.screenBackground = .color(
        color: .init(
            light: UIColor(white: 0, alpha: 0.5),
            dark:  UIColor(white: 0, alpha: 0.5)
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
            make.edges.equalToSuperview().inset(UIEdgeInsets(
                top: flexibleWidth(8),
                left: flexibleWidth(16),
                bottom: flexibleWidth(8),
                right: flexibleWidth(16)
            ))
        }
        thumbImageView.snp.makeConstraints { make in
            make.leading.top.bottom.equalToSuperview()
            make.width.height.equalTo(flexibleWidth(80))
        }
        titleLabel.snp.makeConstraints { make in
            make.top.equalToSuperview().offset(flexibleWidth(12))
            make.leading.equalTo(thumbImageView.snp.trailing).offset(flexibleWidth(12))
            make.trailing.equalToSuperview().inset(flexibleWidth(12))
        }
        priceLabel.snp.makeConstraints { make in
            make.bottom.equalToSuperview().inset(flexibleWidth(12))
            make.leading.equalTo(thumbImageView.snp.trailing).offset(flexibleWidth(12))
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

## Navigation Bar + ScrollView Interaction

When a UIKit `UIScrollView` extends beneath a transparent custom navigation bar, content must be clear before it reaches the navigation area and gain the navigation blur progressively only while scrolling upward.

Required behavior:

- Initial state (`contentOffset.y <= 0`): no blur.
- Upward scrolling: fade the blur in according to scroll distance.
- Pulling down: clamp progress to zero so the blur disappears.
- Place the blur overlay above the scroll view and below the navigation controls so it cannot intercept touches.
- If the public view controller conforms to the public `UIScrollViewDelegate` protocol, declare `scrollViewDidScroll` as `public`.
- Set `contentInsetAdjustmentBehavior = .never` when exact content-to-navigation spacing is required; otherwise UIKit safe-area adjustment can add an unexpected offset.

For UIKit pages, reuse the project's `SCDirectionalFadeBlurOverlayView` and `SCScrollGradientNavigationBarProgress` instead of creating another blur implementation:

```swift
private let navigationBlurOverlay = SCDirectionalFadeBlurOverlayView(
    opaqueEdge: .top,
    showsDimTint: true,
    midLocation: 0.55,
    dimPeakOpacity: 0.7
)

private func setupNavigationBlurOverlay() {
    view.addSubview(navigationBlurOverlay)
    navigationBlurOverlay.alpha = 0
    navigationBlurOverlay.snp.makeConstraints { make in
        make.top.leading.trailing.equalToSuperview()
        make.height.equalTo(getStatusBarHeight() + flexibleWidth(44) + flexibleWidth(20))
    }
}

private func configureScrollView() {
    scrollView.contentInsetAdjustmentBehavior = .never
    scrollView.contentInset = .zero
    scrollView.scrollIndicatorInsets = .zero
    scrollView.delegate = self
}

public func scrollViewDidScroll(_ scrollView: UIScrollView) {
    let progress = SCScrollGradientNavigationBarProgress.progress(
        offsetY: max(0, scrollView.contentOffset.y),
        triggerOffset: flexibleWidth(80)
    )
    navigationBlurOverlay.alpha = progress
}
```

Add the blur overlay after the scroll view and before the navigation controls. Do not use a permanently opaque navigation background, and do not show the blur at the initial offset. Keep the overlay's height large enough to cover the navigation bar and its fade extension below the bar.

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
- [ ] The output matches the current page: SwiftUI page → SwiftUI; Objective-C page → Objective-C; otherwise → Swift UIKit + SnapKit
- [ ] A SwiftUI or Objective-C page was not migrated to UIKit/Swift solely for this UI work
- [ ] For Swift UIKit output: no `frame` / `AutoresizingMask` usage — SnapKit only
- [ ] Code generation does not trigger automatic compilation or app execution unless explicitly requested
- [ ] Every downloaded local PNG uses the implemented page size in points as its `1x` logical size
- [ ] PNG pixel dimensions equal the page size multiplied by `1`, `2`, and `3`; variants come from a high-resolution master
- [ ] Each `.imageset/Contents.json` maps the matching files explicitly to `1x`, `2x`, and `3x`
- [ ] Asset references use the base asset name without `@2x`, `@3x`, or `.png`
- [ ] Safe area insets handled (`safeAreaLayoutGuide`)
- [ ] Colors use `SwiftHEXColors` and `Hue` helpers — no custom hex extension or `AppColor` enum
- [ ] `UIView` and `UIButton` gradients prefer the existing `UIView+Gradient.swift` APIs; gradient frames are updated after layout
- [ ] Transparent glass uses the existing `UIView+SCLiquidGlass` category; glass modes are not enabled simultaneously and glass hosts/buttons remain transparent
- [ ] Images use `kf.setImage` with placeholder
- [ ] `prepareForReuse` cancels Kingfisher tasks in cells
- [ ] JSON models use `SwiftyJSON` with `init(json: JSON)`
- [ ] Popups use `SwiftEntryKit` (no `UIAlertController` unless truly native alert)
- [ ] SwiftEntryKit popups set `entranceAnimation = .none` and `exitAnimation = .none` unless animation is explicitly required
- [ ] Every SwiftEntryKit popup has an explicitly chosen backdrop treatment; request clarification when the design does not specify one
- [ ] Gaussian-blur backdrops use `SCPopupBackdrop.apply(to: &attributes)` with the required full-screen, touch-absorbing attribute configuration
- [ ] Ordinary dimmed backdrops use `attributes.screenBackground` with light/dark alpha `0.5`; no popup or overlay view uses `backgroundColor = UIColor.black.withAlphaComponent(...)`
- [ ] All UI created programmatically (no Storyboard/XIB unless asked)
- [ ] For Swift UIKit output: `// MARK:` sections used for code organization
- [ ] Spacing values written as inline literals — no `enum Layout`
- [ ] Bottom sheets use the **exact** `EKAttributes` config from the Bottom Sheet section (never `EKAttributes.bottomFloat`)
- [ ] Bottom sheet popup view sets `entryBackground = .clear` and draws its own background
- [ ] Bottom sheet config includes `safeArea = .overridden` and `verticalOffset = 0`
- [ ] Do NOT output `extension UIColor` definitions — this extension already exists in the project
- [ ] Swift UI strings use `SCLocalizedText(_:)`; Objective-C uses `SCLocalizedString(key)` — no hardcoded text
- [ ] Existing localization keys are preserved; new keys follow the project's existing key style
- [ ] All 14 project language translations are output alongside the code: `ar-SA`, `es-ES`, `pt-BR`, `fr-FR`, `ja-JP`, `ko-KR`, `ur-PK`, `id-ID`, `hi-IN`, `vi-VN`, `ru-RU`, `tr-TR`, `en-US`, `zh_TW`

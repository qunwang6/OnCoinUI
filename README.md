# OnCoinUI

> A Claude Code Skill that automatically generates clean iOS UIKit Swift code from UI screenshots, design specs, or text descriptions.

---

## Features

- 📸 **Multiple input types**: UI screenshots / Figma descriptions / text requirements
- 📐 **SnapKit layout**: Full Auto Layout DSL — no `frame`, no Storyboard
- 🪟 **Bottom sheets**: Standard SwiftEntryKit config with home indicator support
- 🎨 **Colors**: `SwiftHEXColors` and `Hue` helpers — no custom hex extension or `AppColor` enum
- 🌈 **Gradients**: Prefer the project's `UIView+Gradient.swift` APIs for `UIView` and `UIButton` backgrounds
- 🖼️ **Kingfisher image loading**: Placeholder, fade animation, cell reuse cancellation
- 📦 **Figma PNG scales**: Build correctly mapped iOS `1x` / `2x` / `3x` image sets from the page's actual rendered size
- 📦 **SwiftyJSON models**: Unified `init(json: JSON)` parsing pattern
- 🌐 **Localization**: `SCLanguageManager` with auto-output in the project's 14 supported languages

---

## Installation

### Option 1: npx (recommended)

```bash
npx skills add https://github.com/qunwang6/OnCoinUI
```

### Option 2: Manual

Copy `OnCoinUI/SKILL.md` to one of the following directories:

```bash
# Project-level (current project only)
your-ios-project/.claude/skills/OnCoinUI/SKILL.md

# Global (all projects)
~/.claude/skills/OnCoinUI/SKILL.md
```

---

## Usage

Start Claude Code inside your iOS project directory and describe what you need:

```bash
cd your-ios-project
claude
```

### Upload a screenshot

```
Implement this Swift screen based on the screenshot
```

Drop in a design image — Claude will analyze the layout, colors, and components and generate complete code.

### Text description

```
Build a product detail page:
- Top image carousel
- Product name and price
- Fixed "Buy Now" button at the bottom
```

### Bottom sheet

```
Build a bottom sheet with a title, option list, and cancel button
```

---

## Generated Output

| File | Description |
|---|---|
| `ProductDetailViewController.swift` | Main view controller |
| `ProductCell.swift` | Custom table view cell |
| `ProductBottomSheetView.swift` | Bottom sheet view |
| `ProductModel.swift` | SwiftyJSON data model |
| `ar-SA.lproj/Localizable.strings` | Arabic (Saudi Arabia) localization |
| `es-ES.lproj/Localizable.strings` | Spanish (Spain) localization |
| `pt-BR.lproj/Localizable.strings` | Portuguese (Brazil) localization |
| `fr-FR.lproj/Localizable.strings` | French (France) localization |
| `ja-JP.lproj/Localizable.strings` | Japanese (Japan) localization |
| `ko-KR.lproj/Localizable.strings` | Korean (South Korea) localization |
| `ur-PK.lproj/Localizable.strings` | Urdu (Pakistan) localization |
| `id-ID.lproj/Localizable.strings` | Indonesian (Indonesia) localization |
| `hi-IN.lproj/Localizable.strings` | Hindi (India) localization |
| `vi-VN.lproj/Localizable.strings` | Vietnamese (Vietnam) localization |
| `ru-RU.lproj/Localizable.strings` | Russian (Russia) localization |
| `tr-TR.lproj/Localizable.strings` | Turkish (Türkiye) localization |
| `en-US.lproj/Localizable.strings` | English (United States) localization |
| `zh_TW.lproj/Localizable.strings` | Chinese (Taiwan) localization |

---

## Tech Stack

| Purpose | Library | CocoaPods |
|---|---|---|
| Layout | SnapKit | `pod 'SnapKit'` |
| Popups / Toast | SwiftEntryKit | `pod 'SwiftEntryKit'` |
| Image Loading | Kingfisher | `pod 'Kingfisher'` |
| JSON Parsing | SwiftyJSON | `pod 'SwiftyJSON'` |

### Podfile Example

```ruby
target 'YourApp' do
  use_frameworks!

  pod 'SnapKit'
  pod 'SwiftEntryKit', '2.0.0'
  pod 'Kingfisher'
  pod 'SwiftyJSON'
end
```

---

## Code Conventions

### File Structure

Each file uses consistent `MARK` sections:

```swift
// MARK: - Properties
// MARK: - Lifecycle
// MARK: - Setup
// MARK: - Layout (SnapKit)
// MARK: - Actions
// MARK: - Data / Network
// MARK: - Helpers
```

### Colors

Use `SwiftHEXColors` and `Hue` helpers. Do not define a custom hex extension or an `AppColor` enum.

```swift
// ✅ Correct — inline hex values
label.textColor       = UIColor(hexString: "#2D2F35")
button.backgroundColor = UIColor(hexString: "#149D93")
view.backgroundColor  = UIColor(hexString: "#F8F8F8")

// ❌ Forbidden — no AppColor enum
enum AppColor {
    static let primary = UIColor(hexString: "#149D93")
}
label.textColor = UIColor(hex: "#149D93")
```

### Gradients

For `UIView` and `UIButton` backgrounds, prefer the existing implementation at:

`/Volumes/BACKUP/Code/QiuXing/OnCoin/SeeCoin/Modules/HomePage/LimitTask/Common/UIView+Gradient.swift`

```swift
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

Call `updateGradientFrame()` after Auto Layout, such as from `viewDidLayoutSubviews()` or a reusable view's `layoutSubviews()`. Use `removeGradient()` before replacing an existing gradient. Only use a custom `CAGradientLayer` when this implementation cannot express the required behavior.

### Layout Spacing

Use the project's existing `CommonSize.swift` helpers for Figma-derived spacing, padding, size, and radius. Font sizes must be normalized from the actual Figma design frame width before code generation. Keep the Figma source value and calculation auditable, and do NOT define an `enum Layout`.

- Font size, line height, and non-zero letter spacing: calculate `figmaValue * 375 / figmaDesignFrameWidth`, then write the numeric result directly. Do not use a font scaling helper.
- `flexibleWidth(_:)`: all Figma spacing values, including vertical and horizontal padding, margins, and gaps, calculated from the 375pt design width.
- `flexibleHeight(_:)`: component heights or other dimensions explicitly defined by the 812pt design height.
- `horizontalFlexibleWidth(_:)`: values explicitly measured on the 812pt horizontal coordinate system.
- `getStatusBarHeight()`, `getTabBarHeight()`, and `getBottomSafeAreaInsetHeight()`: system and safe-area dimensions.

CommonSize source: `/Users/qun/Downloads/aaaaa/OnCoin/SeeCoin/CommonUI/CommonSize/CommonSize.swift`.

Figma MCP typography values are raw design values and must be normalized before use. For example, with a Figma design frame width of `750`, `text-[30px]` and `leading-[38px]` become `15` and `19` using `figmaValue * 375 / figmaDesignFrameWidth`. Write those numeric results directly; never use `flexibleWidth` for font values or copy raw MCP output values directly into generated iOS code.

```swift
// ✅ Correct — inline literals
titleLabel.snp.makeConstraints { make in
    make.top.equalToSuperview().offset(flexibleWidth(26))
    make.leading.trailing.equalToSuperview().inset(flexibleWidth(30))
}
titleLabel.font = .systemFont(ofSize: 16, weight: .semibold)

// ❌ Forbidden — raw Figma numbers or a second scaling system
make.leading.trailing.equalToSuperview().inset(30)
titleLabel.font = .systemFont(ofSize: 16)
```

### Figma PNG Assets

Use the image's final rendered size on the iOS page as its `1x` logical size. Normalize a Figma node to the 375pt design width before creating bitmap variants:

```text
displayWidthPt  = figmaNodeWidth  × 375 / figmaDesignFrameWidth
displayHeightPt = figmaNodeHeight × 375 / figmaDesignFrameWidth

1x pixels = display size × 1
2x pixels = display size × 2
3x pixels = display size × 3
```

Example: a `120 × 80` node in a `750`-wide Figma frame displays at `60 × 40pt`. Create `60 × 40px`, `120 × 80px`, and `180 × 120px` PNGs and assign them to the `1x`, `2x`, and `3x` slots respectively.

Store all variants in one `.imageset`:

```text
example_banner.imageset/
├── example_banner.png       # 1x
├── example_banner@2x.png    # 2x
├── example_banner@3x.png    # 3x
└── Contents.json            # explicit scale mapping
```

Figma export scale is relative to the source node. It can differ from the target iOS scale. Generate variants from the highest-resolution source, verify their actual pixel dimensions, keep their crop identical, and reference only the base asset name in code: `UIImage(named: "example_banner")`.

### Localization

```swift
// ✅ Correct
titleLabel.text = SCLanguageManager.shared().localizedString(forKey: "register_email")

// ❌ Forbidden
titleLabel.text = "Email"
titleLabel.text = "Email"
```

Before generating translations, read `OnCoinUI/translation.csv` for existing translations and reuse matching entries. Do not modify the CSV. Translate missing entries manually, preserve all placeholders, and do not output its `zh-CN` reference column; use the project's `zh_TW` language instead.

Output all 14 translation files alongside every generated UI file:

`ar-SA`, `es-ES`, `pt-BR`, `fr-FR`, `ja-JP`, `ko-KR`, `ur-PK`, `id-ID`, `hi-IN`, `vi-VN`, `ru-RU`, `tr-TR`, `en-US`, and `zh_TW` `.lproj/Localizable.strings` files.

### Bottom Sheet (Fixed Config)

For every SwiftEntryKit popup that needs a dimmed screen background, configure the
dimming through `EKAttributes.screenBackground`. Do not set a popup or overlay view's
`backgroundColor = UIColor.black.withAlphaComponent(...)`.

```swift
var attributes = EKAttributes()
attributes.position = .bottom
attributes.displayDuration = .infinity
attributes.screenBackground = .color(
    color: .init(light: UIColor(white: 0, alpha: 0.5),
                 dark:  UIColor(white: 0, alpha: 0.5))
)
attributes.entryBackground = .clear
attributes.screenInteraction = .dismiss
attributes.entryInteraction = .forward
attributes.scroll = .disabled
attributes.positionConstraints.size = .init(width: .fill, height: .intrinsic)
attributes.positionConstraints.safeArea = .overridden
attributes.positionConstraints.verticalOffset = 0
SwiftEntryKit.display(entry: popupView, using: attributes)
```

> ⚠️ Never use `EKAttributes.bottomFloat` — always use the full config above.

### Verification and Execution

Code generation does not automatically compile, install, launch, or run the app. Verification is limited to static inspection, formatting checks, and reviewing the generated diff unless the user explicitly requests compilation, tests, simulator execution, or another runtime verification step.

---

## File Structure

```
OnCoinUI/
├── SKILL.md       # Main skill file (read by Claude Code)
└── README.md      # This file
```

---

## License

MIT

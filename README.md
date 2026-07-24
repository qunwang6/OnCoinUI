# OnCoinUI

> A Claude Code Skill that automatically generates clean iOS UIKit Swift code from UI screenshots, design specs, or text descriptions.

---

## Features

- 📸 **Multiple input types**: UI screenshots / Figma descriptions / text requirements
- 📐 **SnapKit layout**: Full Auto Layout DSL — no `frame`, no Storyboard
- 🪟 **Bottom sheets**: Standard SwiftEntryKit config with home indicator support
- 🎨 **Colors**: `SwiftHEXColors` and `Hue` helpers — no custom hex extension or `AppColor` enum
- 🖼️ **Kingfisher image loading**: Placeholder, fade animation, cell reuse cancellation
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

### Layout Spacing

Write spacing values **as inline literals** in SnapKit constraints — do NOT define an `enum Layout`.

```swift
// ✅ Correct — inline literals
titleLabel.snp.makeConstraints { make in
    make.top.equalToSuperview().offset(26)
    make.leading.trailing.equalToSuperview().inset(30)
}

// ❌ Forbidden — no enum Layout
enum Layout {
    static let sideInset: CGFloat = 30
}
make.leading.trailing.equalToSuperview().inset(Layout.sideInset)
```

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

```swift
var attributes = EKAttributes()
attributes.position = .bottom
attributes.displayDuration = .infinity
attributes.screenBackground = .color(
    color: .init(light: UIColor(white: 0, alpha: 0.4),
                 dark:  UIColor(white: 0, alpha: 0.4))
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

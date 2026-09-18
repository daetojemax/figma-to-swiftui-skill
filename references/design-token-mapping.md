# Figma Design Tokens to SwiftUI Mapping

Use this reference to map Figma variables and styles into the project's SwiftUI tokens. Examples illustrate structure; their values are not defaults to impose on a design.

## Contents

- [Color Tokens](#color-tokens)
- [Spacing Tokens](#spacing-tokens)
- [Typography Tokens](#typography-tokens)
- [Border Radius Tokens](#border-radius-tokens)
- [Shadow Tokens](#shadow-tokens)
- [Gradients](#gradients)
- [Opacity](#opacity)
- [General Rules](#general-rules)

## Color Tokens

Figma color variables map to SwiftUI Color extensions or Asset Catalog named colors.

### Strategy

1. Check if project already has a color system (Color+Extensions.swift, Theme.swift, or Asset Catalog named colors)
2. If yes: map by semantic role and active mode, then verify the values match the design
3. If no: create Color extensions or Asset Catalog entries from Figma variables
4. Prefer semantic colors and named assets already used by adjacent screens before introducing a new token

### Mapping Rules

Figma variable "primary/500" -> Color.primary500 or Color("primary500")
Figma variable "text/primary" -> Color.textPrimary
Figma variable "surface/default" -> Color.surfaceDefault
Figma variable "border/subtle" -> Color.borderSubtle

### Adaptive Colors (Light/Dark)

Figma variables with mode variants (light/dark):
- Asset Catalog: Create color set with Any Appearance + Dark Appearance
- Code: Use @Environment(\.colorScheme) only if Asset Catalog is not an option

```swift
// Asset Catalog approach (preferred)
Color("textPrimary") // automatically adapts

// Code approach (when needed)
extension Color {
    static var textPrimary: Color {
        Color("textPrimary")
    }
}
```

## Spacing Tokens

Figma spacing variables map to CGFloat constants.

```swift
enum Spacing {
    static let xs: CGFloat = 4
    static let sm: CGFloat = 8
    static let md: CGFloat = 12
    static let lg: CGFloat = 16
    static let xl: CGFloat = 24
    static let xxl: CGFloat = 32
}
```

Use project's existing spacing system if one exists. Do not create a parallel system.

## Typography Tokens

Figma typography variables map to Font definitions. Typography is a common source of visual drift in SwiftUI — carry every field, not just size and weight.

### Required fields for every text style

| Figma | SwiftUI |
|---|---|
| font-family | `Font.custom("Family", size:)` or `.system(size:)` |
| font-size | `size:` parameter |
| font-weight | `weight:` parameter |
| font-width (Expanded/Condensed) | `.fontWidth(.expanded)` / `.fontWidth(.condensed)` (iOS 16+) |
| line-height | Preserve the intended line box using the project's typography support and measured font metrics; see below |
| letter-spacing | `.tracking(X)` preferred, `.kerning(X)` only when project uses it |
| text-align | `.multilineTextAlignment(.leading / .center / .trailing)` |
| text-transform: uppercase | `.textCase(.uppercase)` or uppercase localized copy |

### Line Height Pitfall

Figma line height describes the total line box. SwiftUI `lineSpacing` describes spacing between line fragments, so subtracting font size from the target line height is not a general conversion. Font metrics, including leading, affect the result.

Use existing typography helpers when they support the required line height. Otherwise choose a text rendering approach based on the actual font metrics and compare a multiline render. Do not apply negative padding as a universal correction. See Apple's [lineSpacing documentation](https://developer.apple.com/documentation/swiftui/view/linespacing(_:)).

### Letter Spacing Pitfall

For a design using point-equivalent units, `letter-spacing: -0.32px` maps to `.tracking(-0.32)`. Convert percentage or em values using the font size rather than treating them as points. Follow project typography helpers where they already encode tracking.

### Example — Font, Width, and Tracking

```swift
extension Font {
    static let headingLarge = Font.system(size: 28, weight: .bold)
}

Text("Title")
    .font(.headingLarge)
    .fontWidth(.expanded)
    .tracking(-0.56)
    .foregroundStyle(Color("textPrimary"))
    .multilineTextAlignment(.leading)
```

### Custom Fonts

Check existing bundled fonts and system font designs before adding font files. For a required custom family, use available project assets or an authorized source and the correct registered font name. If the font is unavailable, identify that limitation instead of silently substituting a visually different family. Do not treat an Apple system font design as a third-party download.

If the project already provides typography helpers or wrappers, use those first instead of introducing raw font declarations or a parallel typography layer.

### Dynamic Type Support

Always consider Dynamic Type. Prefer .font(.headline) or .font(.body) when Figma typography maps closely to iOS text styles. For custom sizes, use @ScaledMetric:

```swift
@ScaledMetric(relativeTo: .body) private var fontSize: CGFloat = 16
```

## Border Radius Tokens

Figma corner radius variables map to CGFloat constants used with RoundedRectangle:

```swift
enum CornerRadius {
    static let sm: CGFloat = 4
    static let md: CGFloat = 8
    static let lg: CGFloat = 12
    static let xl: CGFloat = 16
    static let full: CGFloat = 9999 // pill shape -> Capsule()
}
```

When radius equals 9999 or "full", use Capsule() instead of RoundedRectangle.

## Shadow Tokens

Figma shadow variables (elevation levels):

```swift
extension View {
    func shadowSm() -> some View {
        shadow(color: .black.opacity(0.05), radius: 2, x: 0, y: 1)
    }
    func shadowMd() -> some View {
        shadow(color: .black.opacity(0.1), radius: 8, x: 0, y: 4)
    }
    func shadowLg() -> some View {
        shadow(color: .black.opacity(0.15), radius: 16, x: 0, y: 8)
    }
}
```

## Gradients

Figma gradients map to SwiftUI gradient types. Match stops and direction exactly.

```swift
// Figma: top-to-bottom linear gradient
LinearGradient(
    colors: [Color("gradientStart"), Color("gradientEnd")],
    startPoint: .top,
    endPoint: .bottom
)

// Figma: specific stop locations
LinearGradient(
    stops: [
        .init(color: Color("gradientStart"), location: 0.0),
        .init(color: Color("gradientMid"), location: 0.6),
        .init(color: Color("gradientEnd"), location: 1.0)
    ],
    startPoint: .leading,
    endPoint: .trailing
)
```

Radial gradient -> `RadialGradient`. Angular/conic gradient -> `AngularGradient`. Match Figma's center, radius, angle, and stops where the design context provides them.

## Opacity

- Figma fill opacity 50% -> `Color(...).opacity(0.5)` on the fill/background
- Figma layer opacity 50% -> `.opacity(0.5)` on the whole view, including children
- These differ: fill opacity affects only the fill color; layer opacity affects everything inside
- Tailwind `bg-black/50` = fill opacity; `opacity-50` = layer opacity

## General Rules

1. Always check project for existing design system before creating new tokens
2. Match semantic role and mode, then verify values; equal colors can serve different roles
3. If project tokens exist but names differ from Figma, use project names
4. Do not duplicate: one source of truth for each token
5. Prefer existing shared modules and helpers, theme wrappers, and Asset Catalog colors when they already express the same intent
6. Group tokens logically (Color, Spacing, Typography, Radius, Shadow)
7. Resolve a mismatched token with a scoped variant or clarification; do not change a shared token globally just to align one screen

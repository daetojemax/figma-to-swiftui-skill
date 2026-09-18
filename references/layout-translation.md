# Figma Layout to SwiftUI Translation

Reference for translating Figma layout intent into native SwiftUI. Examples illustrate patterns; use the actual design values and supported project APIs.

## Contents

- [Auto Layout to Stacks](#auto-layout-to-stacks)
- [Absolute Positioning](#absolute-positioning)
- [Scroll](#scroll)
- [Common Patterns](#common-patterns)
- [Effects & Decorations](#effects--decorations)
- [Animations & Transitions](#animations--transitions)

## Auto Layout to Stacks

Figma Auto Layout is the closest analog to SwiftUI stacks. The layout intent transfers, but SwiftUI's proposal and sizing behavior can differ.

### Direction

- Vertical auto layout -> VStack(alignment:, spacing:)
- Horizontal auto layout -> HStack(alignment:, spacing:)
- Wrap (horizontal with line break) -> No native SwiftUI equivalent. Use LazyVGrid with adaptive columns, or a custom FlowLayout.

### Alignment

Figma auto layout alignment maps to SwiftUI alignment:

Primary axis alignment (justify):
- Packed (start) -> Default stack behavior (no spacer)
- Packed (center) -> Wrap content in stack with Spacer() on both sides, or use .frame(maxWidth/Height: .infinity) with centered alignment
- Packed (end) -> Spacer() before content
- Space between -> Spacer() between each child element
- Space around / space evenly -> Not native; distribute with custom spacing or GeometryReader

Cross axis alignment:
- VStack: .leading, .center, .trailing
- HStack: .top, .center, .bottom, .firstTextBaseline, .lastTextBaseline

### Spacing (Gap)

Figma gap value maps directly to spacing parameter:
- gap: 12 -> VStack(spacing: 12) or HStack(spacing: 12)
- Mixed gaps between children -> Cannot use single spacing value. Use explicit Spacer().frame(height/width:) or padding between children.

### Padding

Figma padding maps to SwiftUI .padding():
- Uniform padding: 16 -> .padding(16)
- Horizontal 16, Vertical 12 -> .padding(.horizontal, 16).padding(.vertical, 12)
- Individual edges -> .padding(EdgeInsets(top:, leading:, bottom:, trailing:))
- Note: Figma uses left/right, SwiftUI uses leading/trailing for RTL support

**Padding vs background order matters:**
```swift
// Figma: card with 16pt inner padding, white bg, 12pt radius
content
    .padding(16)
    .background(Color.white, in: .rect(cornerRadius: 12))

// Not equivalent:
content
    .background(Color.white)
    .padding(16)
```

**Text bounds can differ from container padding.** Inspect the text line box and parent layout before treating apparent vertical whitespace as an additional inset. See [visual-fidelity.md](visual-fidelity.md) for typography and rendering differences.

### Sizing

Figma sizing modes:
- Fixed (width: 200) -> .frame(width: 200)
- Hug contents -> No modifier needed. SwiftUI views hug by default.
- Fill container -> .frame(maxWidth: .infinity) or .frame(maxHeight: .infinity)
- Fill with min/max -> .frame(minWidth:, maxWidth:, minHeight:, maxHeight:)

**Common sizing mistakes:**
- Applying `.frame(width: 375)` on a full-width element -> use `.frame(maxWidth: .infinity)` so it adapts to device width
- Forgetting `.frame(maxWidth: .infinity, alignment: .leading)` when Figma left-aligns content inside a fill-width container
- Using `.frame(height:)` on Text -> Text height comes from the font line box; fixed height can clip or add unexpected space
- Applying `.frame` to an image without `.resizable()` -> the image stays at intrinsic size

### Aspect Ratio

- Figma constraint "Preserve aspect ratio" -> .aspectRatio(width/height, contentMode: .fit) or .fill

## Absolute Positioning

Figma frames without auto layout use absolute (x, y) positioning.

- Prefer translating to stacks when the visual structure allows it
- When absolute positioning is necessary, use an aligned ZStack and account for child bounds; `.offset` moves the rendered view without changing its layout footprint
- For responsive absolute layouts, use GeometryReader (sparingly)
- Figma constraints (pin left, pin top, etc.) -> combine .frame() with alignment parameters in the parent

## Scroll

- Confirm scrolling from prototype behavior or the brief; clipping alone can describe a static crop
- Vertical scroll -> ScrollView(.vertical) { VStack { ... } }
- Horizontal scroll -> ScrollView(.horizontal) { HStack { ... } }
- Both directions -> ScrollView([.vertical, .horizontal]) { ... }
- Paging -> ScrollView { LazyHStack { ... } }.scrollTargetBehavior(.paging)

## Common Patterns

### Card Layout
Figma: Frame (auto layout vertical, padding 16, corner radius 12, drop shadow, fill white)
SwiftUI:
```swift
VStack(alignment: .leading, spacing: 8) {
    // card content
}
.padding(16)
.background(Color.white)
.clipShape(RoundedRectangle(cornerRadius: 12))
.shadow(color: .black.opacity(0.1), radius: 8, x: 0, y: 2)
```

### List Item
Figma: Frame (auto layout horizontal, spacing 12, padding vertical 12 horizontal 16, fill container)
SwiftUI:
```swift
HStack(spacing: 12) {
    // list item content
}
.padding(.vertical, 12)
.padding(.horizontal, 16)
.frame(maxWidth: .infinity, alignment: .leading)
```

### Header with Back Button
Figma: Frame (auto layout horizontal, space between, padding 16)
SwiftUI: Prefer .navigationTitle() + .toolbar {} over custom header when possible. Custom header only if design is significantly non-standard.

### Bottom Safe Area Content
Figma: An action area pinned above the bottom safe area while content can scroll
SwiftUI:
```swift
ScrollView {
    content
}
.safeAreaInset(edge: .bottom) {
    bottomActions
        .padding(.horizontal, 16)
        .padding(.vertical, 8)
}
```

## Effects & Decorations

| Figma | SwiftUI |
|---|---|
| Drop shadow | `.shadow(color:, radius:, x:, y:)` — supply the design's color and offsets |
| Inner shadow | `.overlay { RoundedRectangle(...).stroke(...).blur(...) }` or custom drawing |
| Layer blur | `.blur(radius:)` |
| Background blur | `.background(.ultraThinMaterial)` / `.regularMaterial` / `.thickMaterial` |
| Corner radius, all equal | `.clipShape(.rect(cornerRadius:))` |
| Individual corners | `UnevenRoundedRectangle(topLeadingRadius:, topTrailingRadius:, bottomLeadingRadius:, bottomTrailingRadius:)` |
| Border / stroke | `.overlay(RoundedRectangle(...).stroke(color, lineWidth:))` |
| Clip content | `.clipped()` or `.clipShape(...)` |
| Mask | `.mask { ... }` |
| Blend mode | `.blendMode(.multiply)` etc. |
| Liquid Glass (iOS 26+) | `.glassEffect()` with appropriate shape |

## Animations & Transitions

Figma prototype connections describe transition intent, not literal animation specs. Interpret them as navigation or state-change animations.

| Figma | SwiftUI |
|---|---|
| Dissolve | `.opacity(...)` + `withAnimation(.easeInOut)` |
| Move in / slide in | `.transition(.move(edge:))` or `.offset(...)` |
| Push | `NavigationStack` push using the system transition |
| Smart animate | `withAnimation { }` on state changes |
| Scroll animate | `ScrollView` + `.scrollTransition()` when supported |

Rules:
- Reuse the project's animation pipeline for matching assets or behaviors; a dependency's presence does not require it for every transition
- Do not over-animate; prototype links usually mean navigation, not custom animation
- For specified choreography, preserve its timing and state relationships. Ask about scope only when required motion is unclear or a proposed simplification would change it

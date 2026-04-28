# Figma Layout to SwiftUI Translation

Complete reference for translating Figma layout concepts into SwiftUI code.

## Contents

- [Auto Layout to Stacks](#auto-layout-to-stacks)
- [Absolute Positioning](#absolute-positioning)
- [Scroll](#scroll)
- [Common Patterns](#common-patterns)
- [Effects & Decorations](#effects--decorations)
- [Animations & Transitions](#animations--transitions)

## Auto Layout to Stacks

Figma Auto Layout is the closest analog to SwiftUI stacks. The translation is mostly 1:1, but edge cases exist.

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

**Figma text-layer padding is not always real padding:** Figma sometimes encodes vertical centering as top/bottom padding. When a Text layer has padding top=4, bottom=4 and line-height != font-size, this is usually the text line box, not container padding. Do not double-apply it. See references/visual-fidelity.md for line-height handling.

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
- When absolute positioning is necessary, use ZStack with .offset(x:, y:)
- For responsive absolute layouts, use GeometryReader (sparingly)
- Figma constraints (pin left, pin top, etc.) -> combine .frame() with alignment parameters in the parent

## Scroll

- Figma frame with "Clip content" + overflow -> ScrollView
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
Figma: Frame pinned to bottom with padding
SwiftUI:
```swift
VStack {
    Spacer()
    content
        .padding(.horizontal, 16)
        .padding(.bottom, 8)
}
.safeAreaInset(edge: .bottom) { ... }
// or use .toolbar(.bottomBar)
```

## Effects & Decorations

| Figma | SwiftUI |
|---|---|
| Drop shadow | `.shadow(color:, radius:, x:, y:)` — use full form; defaults are wrong |
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
- Check project dependencies for Lottie or another animation library and use it if already present
- Do not over-animate; prototype links usually mean navigation, not custom animation
- For complex choreographed animations, ask whether to implement fully or simplify

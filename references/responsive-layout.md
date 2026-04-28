# Figma to Responsive SwiftUI Layout

How to translate device-specific Figma frames into adaptive SwiftUI views. Complements layout-translation.md (which covers 1:1 Auto Layout mapping) with multi-device adaptation.

## Contents

- [When to Ask About Device Support](#when-to-ask-about-device-support)
- [Figma Fixed Values → Adaptive SwiftUI](#figma-fixed-values--adaptive-swiftui)
- [Size Classes for Layout Switching](#size-classes-for-layout-switching)
- [Navigation and Sidebars](#navigation-and-sidebars)
- [Images and Aspect Ratios](#images-and-aspect-ratios)
- [Safe Areas by Device](#safe-areas-by-device)
- [Typography Across Devices](#typography-across-devices)
- [Multi-Device Implementation Pattern](#multi-device-implementation-pattern)
- [Checklist](#checklist)

## When to Ask About Device Support

- Figma frame width 375–430pt (iPhone range) and the project's deployment target includes iPad → ask the user if iPad adaptation is needed before implementing
- Figma contains multiple frames for different devices (iPhone + iPad) → fetch all frames via get_design_context + get_screenshot, then ask the user how to combine them
- Figma frame width 744–1024pt (iPad range) only → ask if iPhone support is needed
- Do not assume. Always confirm device scope with the user.

## Figma Fixed Values → Adaptive SwiftUI

Figma designs use absolute pixel values. Not all of them should become fixed frames in SwiftUI.

**Full-screen width (375, 390, 393, 430)**
→ `.frame(maxWidth: .infinity)`, never `.frame(width: 375)`

**Fixed-size elements (icons, avatars, badges)**
→ Keep `.frame(width:, height:)` — these are intentionally fixed

**Content containers with fixed width**
→ Replace with relative sizing. Use `containerRelativeFrame` (iOS 17+) or `GeometryReader` for proportional widths:
```swift
// Figma: card width 343 in 375pt frame (91.5% of screen)
.containerRelativeFrame(.horizontal) { length, _ in
    length * 0.915
}
```

**Banned: `UIScreen.main.bounds`**
→ Always use `containerRelativeFrame` (iOS 17+) or `GeometryReader`. Screen bounds breaks in Split View, Slide Over, and Stage Manager.

## Size Classes for Layout Switching

Use `@Environment(\.horizontalSizeClass)` when Figma shows fundamentally different layouts per device (not just wider spacing).

- compact = iPhone portrait, iPad split/slide-over
- regular = iPad full-screen, iPhone landscape (some models)

```swift
struct ContentView: View {
    @Environment(\.horizontalSizeClass) private var sizeClass

    var body: some View {
        if sizeClass == .compact {
            NavigationStack {
                ItemList()
            }
        } else {
            NavigationSplitView {
                ItemList()
            } detail: {
                ItemDetail()
            }
        }
    }
}
```

When to use size classes:
- Figma shows list (iPhone) vs grid (iPad) → switch layout
- Figma shows single column (iPhone) vs sidebar + content (iPad) → NavigationSplitView
- Figma shows stacked sections (iPhone) vs side-by-side (iPad) → switch between VStack and HStack

When NOT to use size classes:
- Same layout, just wider → use flexible frames and `.infinity`, no branching needed

## Merging iPhone + iPad Figma Frames

When Figma provides separate frames for iPhone and iPad:

1. Fetch both frames via get_design_context + get_screenshot
2. Identify shared components (same content, same structure) → extract into shared views
3. Identify differences (layout changes, visibility changes, different arrangements)
4. Implement one SwiftUI view that switches on `horizontalSizeClass`

```swift
struct ProfileView: View {
    @Environment(\.horizontalSizeClass) private var sizeClass

    var body: some View {
        if sizeClass == .compact {
            // iPhone layout: vertical stack
            ScrollView {
                VStack(spacing: 16) {
                    ProfileHeader()
                    ProfileStats()
                    ProfileContent()
                }
            }
        } else {
            // iPad layout: side-by-side
            HStack(alignment: .top, spacing: 24) {
                VStack {
                    ProfileHeader()
                    ProfileStats()
                }
                .frame(width: 320)

                ProfileContent()
                    .frame(maxWidth: .infinity)
            }
        }
    }
}
```

## ViewThatFits (iOS 16+)

Use when Figma shows two layout variants (e.g. horizontal and vertical) without tying them to specific devices. SwiftUI picks the first variant that fits the available space.

```swift
ViewThatFits(in: .horizontal) {
    // Try horizontal first
    HStack(spacing: 12) {
        icon
        label
        Spacer()
        value
    }
    // Fall back to vertical
    VStack(alignment: .leading, spacing: 8) {
        HStack(spacing: 12) { icon; label }
        value
    }
}
```

Best for: action bars, label+value pairs, tag rows — anywhere content may or may not fit in one line.

## Common Figma → Responsive Patterns

| Figma Design | SwiftUI Implementation |
|---|---|
| Sidebar + content (iPad) | `NavigationSplitView` |
| 2-col grid (iPad) → 1-col (iPhone) | `LazyVGrid` with adaptive columns: `GridItem(.adaptive(minimum: 160))` |
| Full-width card (iPhone) + constrained card (iPad) | `.frame(maxWidth: 600)` with `.frame(maxWidth: .infinity)` parent for centering |
| Horizontal tabs (iPad) → bottom tab bar (iPhone) | `TabView` (system handles placement) or switch on sizeClass |
| Wide form fields (iPad) → full-width (iPhone) | `.frame(maxWidth: 500)` centered in container |

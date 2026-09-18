# Figma to Responsive SwiftUI Layout

Use this reference when supported container sizes need different layouts. See [layout-translation.md](layout-translation.md) for the base layout mapping.

## Establish device scope

Use the request and the app's existing device support. Preserve adaptive behavior already provided by the app; a single iPhone mockup does not require a new iPad design or a routine clarification.

Fetch relevant device variants when the task includes them. Ask only if the required layouts cannot be inferred and the choice would materially change the feature. Do not fetch every device frame merely because it exists in the file.

## Figma Fixed Values → Adaptive SwiftUI

Figma designs use absolute pixel values. Not all of them should become fixed frames in SwiftUI.

**Full-screen width (375, 390, 393, 430)**
→ `.frame(maxWidth: .infinity)`, never `.frame(width: 375)`

**Fixed-size elements (icons, avatars, badges)**
→ Keep `.frame(width:, height:)` — these are intentionally fixed

**Content containers with fixed width**
→ Infer whether the width represents edge insets, a maximum content width, or a true proportion. A 343pt card in a 375pt frame often means 16pt margins, not a width of 91.5% on every device. Use flexible width plus padding for fixed margins. Use `containerRelativeFrame` (iOS 17+) or `GeometryReader` when the design actually specifies a proportion:
```swift
// Only when the design calls for a proportional width.
.containerRelativeFrame(.horizontal) { length, _ in
    length * 0.915
}
```

Measure the available container instead of using `UIScreen.main.bounds` for layout. The screen size does not describe a window in Split View or Stage Manager; flexible stacks often avoid explicit measurement entirely.

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

When the requested device scope includes separate iPhone and iPad frames:

1. Fetch both frames via get_design_context + get_screenshot
2. Identify shared components (same content, same structure) → extract into shared views
3. Identify differences (layout changes, visibility changes, different arrangements)
4. Share content and state; choose size classes or available-width layout based on the differences

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

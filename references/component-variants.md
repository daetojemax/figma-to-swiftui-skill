# Figma Component Variants to SwiftUI

How to translate Figma variant properties (State, Size, Style/Type, content toggles) into SwiftUI constructs.

## Contents

- [Identifying Variants in MCP Output](#identifying-variants-in-mcp-output)
- [State Variants](#state-variants)
- [Size Variants](#size-variants)
- [Style / Type Variants](#style--type-variants)
- [Content Toggle Variants](#content-toggle-variants)
- [Combined Variants](#combined-variants)
- [Ask Before Choosing Architecture](#ask-before-choosing-architecture)
- [Reusable Component Pattern](#reusable-component-pattern)
- [Forms and Inputs](#forms-and-inputs)
- [Variant Implementation Checklist](#variant-implementation-checklist)

## Identifying Variants in MCP Output

Figma components use variant properties to define visual permutations. When `get_design_context` returns a component instance, look for:
- Property names like State, Size, Style, Type, HasIcon, ShowSubtitle
- Multiple variant values (e.g., State=Default, Pressed, Disabled, Loading)

Fetch all variants of a component set, not just the default. Use `get_metadata` to find sibling variant nodes, then `get_design_context` on each to understand the full range of visual states.

## State Variants

Map Figma state variants to the closest native SwiftUI mechanism. Only create custom state enums for states that have no system equivalent.

### System-provided states (use these first):

- Pressed -> `configuration.isPressed` inside `ButtonStyle.makeBody(configuration:)`
- Disabled -> `@Environment(\.isEnabled)` in the style, or `.disabled(true)` on the call site
- On/Off (toggle) -> `configuration.isOn` inside `ToggleStyle`
- Focused -> `@FocusState` and `.focused()` modifier
- Selected (in a list/picker) -> Selection binding in List/Picker

### Custom states (no system equivalent):

- Loading, Error, Empty, Skeleton -> Model as an enum, drive with @State or view model

```swift
enum ButtonLoadingState {
    case idle, loading, success, error
}

struct PrimaryButtonStyle: ButtonStyle {
    let loadingState: ButtonLoadingState
    @Environment(\.isEnabled) private var isEnabled

    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .opacity(configuration.isPressed ? 0.7 : 1.0)
            .opacity(isEnabled ? 1.0 : 0.5)
            .overlay {
                if loadingState == .loading {
                    ProgressView()
                }
            }
            .allowsHitTesting(loadingState != .loading)
    }
}
```

Rule: if a Figma state matches a system state (pressed, disabled, on/off, focused), use the system mechanism. Custom enum only for states the system does not provide.

## Size Variants

### System control sizes:

`.controlSize(.mini / .small / .regular / .large / .extraLarge)` works for system controls (Button, Toggle, Picker, DatePicker, etc.) but has no effect on custom views.

### Custom size enum (for custom components):

```swift
enum ComponentSize {
    case small, medium, large
}

struct PrimaryButtonStyle: ButtonStyle {
    let size: ComponentSize

    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .font(fontSize)
            .padding(.horizontal, horizontalPadding)
            .padding(.vertical, verticalPadding)
    }

    private var fontSize: Font {
        switch size {
        case .small: .footnote
        case .medium: .body
        case .large: .title3
        }
    }

    private var horizontalPadding: CGFloat {
        switch size {
        case .small: 12
        case .medium: 16
        case .large: 20
        }
    }

    private var verticalPadding: CGFloat {
        switch size {
        case .small: 6
        case .medium: 10
        case .large: 14
        }
    }
}
```

Rule: use `.controlSize()` when the component wraps a system control. Use a custom enum when building a fully custom component.

## Style/Type Variants

Figma designs often have Style or Type properties (Primary, Secondary, Destructive, Ghost, etc.).

### One style with enum parameter — when differences are minimal (colors, borders):

```swift
enum ButtonVariant {
    case primary, secondary, destructive
}

struct AppButtonStyle: ButtonStyle {
    let variant: ButtonVariant

    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .foregroundStyle(foregroundColor)
            .background(backgroundColor)
            .overlay {
                RoundedRectangle(cornerRadius: 8)
                    .stroke(borderColor, lineWidth: variant == .secondary ? 1 : 0)
            }
    }

    private var foregroundColor: Color { /* switch on variant */ }
    private var backgroundColor: Color { /* switch on variant */ }
    private var borderColor: Color { /* switch on variant */ }
}
```

### Separate styles — when layout or structure differs significantly:

```swift
struct FloatingButtonStyle: ButtonStyle { /* icon-only, circular, shadow */ }
struct TextLinkButtonStyle: ButtonStyle { /* underlined text, no background */ }
```

Rule: prefer one style with an enum parameter when the only differences are colors, borders, or font weights. Use separate styles when layout, structure, or content arrangement differs.

## Content Toggles

Figma variants like HasIcon=true/false, ShowSubtitle=true/false, ShowBadge=true/false represent optional content slots.

### Optional parameters:

```swift
struct CardView: View {
    let title: String
    var subtitle: String? = nil
    var icon: Image? = nil
    var badge: Int? = nil

    var body: some View {
        HStack(spacing: 12) {
            if let icon { icon.frame(width: 24, height: 24) }
            VStack(alignment: .leading, spacing: 4) {
                Text(title).font(.headline)
                if let subtitle { Text(subtitle).font(.subheadline).foregroundStyle(.secondary) }
            }
            Spacer()
            if let badge { Text("\(badge)").font(.caption).padding(4).background(.red, in: .capsule) }
        }
    }
}
```

### @ViewBuilder for flexible content slots:

```swift
struct CardView<Header: View, Footer: View>: View {
    let title: String
    @ViewBuilder let header: Header
    @ViewBuilder let footer: Footer

    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            header
            Text(title).font(.headline)
            footer
        }
    }
}
```

Rule: use optional parameters for simple toggles (icon, subtitle, badge). Use @ViewBuilder generics when the slot content varies significantly in structure.

## Full Example: Button with State + Size + Style

Combining all variant dimensions into a single component:

```swift
struct AppButtonStyle: ButtonStyle {
    let variant: ButtonVariant
    let size: ComponentSize
    let loadingState: ButtonLoadingState

    @Environment(\.isEnabled) private var isEnabled

    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .font(size.font)
            .padding(.horizontal, size.horizontalPadding)
            .padding(.vertical, size.verticalPadding)
            .foregroundStyle(variant.foregroundColor)
            .background(variant.backgroundColor, in: RoundedRectangle(cornerRadius: size.cornerRadius))
            .opacity(configuration.isPressed ? 0.7 : 1.0)
            .opacity(isEnabled ? 1.0 : 0.5)
            .overlay {
                if loadingState == .loading {
                    ProgressView().tint(variant.foregroundColor)
                }
            }
            .allowsHitTesting(loadingState != .loading)
    }
}

// Usage
Button("Submit") { submit() }
    .buttonStyle(AppButtonStyle(variant: .primary, size: .large, loadingState: viewModel.submitState))
    .disabled(viewModel.isFormInvalid)
```

## Full Example: Text Field with State Variants

```swift
struct AppTextField: View {
    let placeholder: String
    @Binding var text: String
    var error: String? = nil
    @FocusState private var isFocused: Bool
    @Environment(\.isEnabled) private var isEnabled

    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            TextField(placeholder, text: $text)
                .focused($isFocused)
                .padding(12)
                .background(isEnabled ? Color(.systemBackground) : Color(.secondarySystemBackground))
                .overlay {
                    RoundedRectangle(cornerRadius: 8)
                        .stroke(borderColor, lineWidth: isFocused || error != nil ? 2 : 1)
                }
            if let error {
                Text(error).font(.caption).foregroundStyle(.red)
            }
        }
    }

    private var borderColor: Color {
        if error != nil { return .red }
        if isFocused { return .accentColor }
        return Color(.separator)
    }
}
```

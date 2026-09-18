# Component Variants

Inspect the variants used by the requested screen and any states its behavior requires. Fetch siblings only when they answer a concrete question; a component set can contain many unrelated permutations.

Follow the existing component API when it supports the design. Otherwise choose a representation based on how the variants differ; routine API choices do not require user approval.

## State

| Figma state | SwiftUI mechanism |
|---|---|
| Pressed | `configuration.isPressed` in `ButtonStyle` |
| Disabled | `.disabled(...)` and the `isEnabled` environment |
| Toggle on/off | A binding or `ToggleStyle.Configuration.isOn` |
| Focused | `@FocusState` |
| Selected | The control's selection binding |
| Loading, error, empty, success | Existing model state, or a scoped enum when useful |

Preserve control semantics. For example, a loading overlay must not leave duplicate submissions enabled: disable the action at the control/model boundary as appropriate, including keyboard and accessibility activation. Avoid separate state that can drift from the app's actual request state.

## Size, style, and content

- Use system control sizes when they provide the desired behavior. A custom style must explicitly honor any size input it needs.
- Use a style enum when differences are limited to colors, borders, or typography; separate components or styles can be clearer when structure differs substantially.
- Model simple optional content with optional values, such as an icon or subtitle. Use view-builder slots when the content's structure varies.
- Do not add every possible size/style/state combination to a component used for one specific variant.

For example, an existing project button might be used as:

```swift
Button("Continue", action: submit)
    .buttonStyle(AppButtonStyle(variant: .primary, size: .large))
    .disabled(isSubmitting || !isValid)
```

The names and parameters here are illustrative; use the actual project API and design values. Loading appearance, when required, should be driven by the same `isSubmitting` state.

For a component-library task, cover the requested public variants and their interactions. For a screen task, cover the variants that screen needs. Check long labels, optional content, and disabled/loading behavior where they affect the implementation.

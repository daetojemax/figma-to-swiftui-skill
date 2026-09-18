# Visual Fidelity

Use this reference to extract exact properties or diagnose a mismatch between Figma and a rendered SwiftUI view. It does not require a full inventory for every small edit.

## Interpret the evidence

The design context may encode properties as generated web code. Read layout, typography, fills, effects, and constraints from it; build native SwiftUI from those properties.

- Explicit values such as `p-[17px]`, `leading-[22px]`, and `tracking-[-0.32px]` preserve useful measurements. Do not round them to convenient defaults.
- Resolve named classes through the provided tokens or styles. Do not assume a particular Tailwind scale, root font size, or theme when the output does not establish it.
- Check the active variant and variable mode. A token definition from another mode or a stale cached screenshot can explain apparent contradictions.
- Use screenshots to confirm composition and catch details omitted from structured output. Values inferred from pixels are estimates, not exact design measurements.
- Code Connect mappings identify reusable implementations. Verify their current behavior and supported variants rather than treating a mapping as proof of a visual match.

For a complex screen, keep compact working notes for its containers, typography, asset references, and effects. Record sources or uncertainties where they help resolve conflicting values; a fixed per-element report is unnecessary.

## Native rendering pitfalls

### Typography

Carry font family, weight, width, size, tracking, and line-height intent. Use the project's typography helpers and preserve Dynamic Type behavior.

SwiftUI `lineSpacing` controls the gap between line fragments; it does not set an absolute line height. Do not universally translate Figma line height as `lineHeight - fontSize` or compensate with negative padding. Use the actual font metrics and the project's text rendering approach, then compare a multiline render. See [design-token-mapping.md](design-token-mapping.md) and Apple's [lineSpacing documentation](https://developer.apple.com/documentation/swiftui/view/linespacing(_:)).

Check wrapping and truncation with realistic content. A font fallback or missing expanded/condensed face can change width even when size and weight match. Avoid fixed text heights that clip at larger accessibility sizes.

### Layout and controls

- Explicit stack gaps and edge padding are useful when matching specified values; system defaults can be appropriate for native controls.
- Modifier order matters: padding before a background enlarges the painted surface; padding after it adds exterior space.
- Distinguish the text line box from actual container padding before applying both.
- For a custom button, use the project's style or `.buttonStyle(.plain)` as appropriate so inherited styling does not change its appearance.
- `List`, `Form`, and navigation containers contribute insets, separators, and chrome. Account for those before adding compensating offsets or replacing the native component.
- Use native keyboard, status bar, and home indicator. A screenshot containing mock system chrome does not make it app content.

### Images and effects

- Size resizable assets deliberately and match fit versus fill, clipping, and template tint. See [asset-handling.md](asset-handling.md).
- Distinguish fill opacity from layer opacity; the latter also affects children.
- Preserve gradient stops and direction, stroke placement, radius, and shadow color/offset. Blur and shadow rendering may need visual adjustment across renderers.
- Check the deployment target before using an unfamiliar modifier. Do not assume helpers such as `Color(hex:)` exist in the project.

## Compare the rendered result

Use a preview or simulator when available. Match the reference's device/container dimensions, content, state, appearance, and default text size before comparing. Inspect the affected screen or section for:

- Missing or misplaced content, spacing, alignment, and safe-area behavior.
- Text baselines, wrapping, line height, font width, and tracking.
- Asset identity, crop, resolution, and tint.
- Fills, borders, gradients, shadows, and material effects.

Fix observed differences and recheck the affected region. When code values match but rendering differs, inspect font metrics, layout proposals, modifier order, and native insets before changing the design measurements. Record deliberate platform or accessibility adaptations where they affect the result.

Exercise additional states, larger text, dark appearance, or wider layouts when the change affects them or the request includes them. Do not invent unsupported dark designs or expand a single-screen task into a full-device test matrix.

If rendering is unavailable, inspect the code and assets and report that visual fidelity remains unverified. An imagined screenshot or code-only comparison is not a rendered check.

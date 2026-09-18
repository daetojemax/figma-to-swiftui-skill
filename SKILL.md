---
name: figma-to-swiftui
description: Implement or update iOS SwiftUI UI from Figma designs; also use for implementation plans, token mapping, or asset export for that UI.
---

# Figma to SwiftUI

Deliver native SwiftUI that matches the requested Figma design and fits the existing app. Treat generated web code as design evidence, not as code to port. Preserve the project's architecture, component APIs, and behavior outside the requested change.

## Choose the scope

Use the user's request and any brief to establish the target screens, actions, states, and supported devices. A brief defines behavior and scope; Figma defines visuals within that scope. Read [source-document.md](references/source-document.md) when a brief needs reconciliation with the design.

- **Implement a screen or component:** gather the relevant design evidence and project context, then implement through completion.
- **Update existing UI:** inspect the affected view and dependencies; use [adaptation-workflow.md](references/adaptation-workflow.md) to preserve behavior while closing the visual differences.
- **Plan, map tokens, or export assets:** deliver that artifact without automatically implementing screens or changing shared design systems.

Resolve routine choices from the codebase and design. Ask when an unresolved choice materially changes scope, behavior, or the target design; continue independent work while it is clarified. A plan or difference checklist is working context, not a mandatory approval gate.

## Gather the evidence needed

Use available Figma tools according to their current schemas. Do not assume every server exposes the same tools or parameters.

- For `/design/` or legacy `/file/` links, extract the file key and normalize `node-id=3166-70147` to `3166:70147`. Desktop selection can supply the target when supported. For prototype or board links, resolve the underlying design frame if possible; otherwise request a design link.
- For a clear screen/component, obtain design context and a visual reference. Reuse supplied or already fetched evidence when it is sufficient and current.
- For a root, page, or ambiguous flow, use metadata to locate the relevant frames before fetching detailed context. See [screen-discovery.md](references/screen-discovery.md).
- Split oversized or truncated context into meaningful child nodes rather than repeating the same failed request. See [fetch-strategy.md](references/fetch-strategy.md) for tool selection and cache scope.
- Inspect existing components, tokens, assets, deployment target, and relevant dependencies. Read Code Connect mappings when available and useful; verify the mapped SwiftUI component fits the requested variant.

If Figma access fails, use [figma-mcp-setup.md](references/figma-mcp-setup.md). Continue work supported by available evidence and identify what remains blocked rather than inventing missing design details.

## Preserve the design and platform behavior

- Use exact design values where available and account for differences in native rendering. Prefer project tokens and components that express the same intent; do not silently replace a requested visual change with a mismatched token or alter a shared token globally.
- Use real Figma assets for authored icons, logos, and illustrations. Do not substitute SF Symbols, text, or approximate drawings without user direction. Structural geometry can be SwiftUI; data-driven images use the app's image pipeline.
- Let iOS render system chrome such as the keyboard, status bar, and home indicator. Preserve native navigation, control semantics, safe areas, and accessibility where they meet the design.
- Follow relevant project conventions for image loading, localization, state, and animation. When no convention exists, choose a suitable native implementation within scope; adding a library is not a prerequisite.

Load only the references needed for the current work:

| Need | Reference |
|---|---|
| Exported artwork, PNG validation, asset catalog scale and tint | [asset-handling.md](references/asset-handling.md) |
| Figma variables, semantic tokens, typography metrics | [design-token-mapping.md](references/design-token-mapping.md) |
| Auto Layout, sizing, effects, transitions | [layout-translation.md](references/layout-translation.md) |
| Multiple device frames or adaptive container layouts | [responsive-layout.md](references/responsive-layout.md) |
| Component states, sizes, styles, optional content | [component-variants.md](references/component-variants.md) |
| Exact value extraction or visual mismatch diagnosis | [visual-fidelity.md](references/visual-fidelity.md) |

## Finish the requested work

For implementation, complete the requested UI, assets, and behavior, including specified loading, error, empty, and disabled states. Continue through relevant local checks and fix issues introduced by the change. Choose verification proportionate to the work: a build for code changes when available, and a rendered comparison for visual changes when a preview or simulator is available. Respect an explicit request to skip a check; do not make choosing a validation method a routine question.

Use the same screen state, appearance, and device/container size for visual comparison. Stop iterating when the requested behavior works and observed material discrepancies are resolved. State what was checked and any remaining limitations; code inspection alone does not establish visual fidelity.

Register or update Code Connect mappings only when that external change is part of the user's request or existing authorization. An implementation task does not by itself require publishing mappings.

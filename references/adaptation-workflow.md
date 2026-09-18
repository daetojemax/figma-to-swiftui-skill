# Updating an Existing Screen

Inspect the affected view, relevant subcomponents, and the state/actions they expose. Preserve working data flow, navigation, accessibility, and public component APIs unless the request changes them. A visual update does not imply an architecture rewrite.

## Find the differences that matter

Compare the target design against the current implementation. Keep a working list of additions, updates, and removals when the change spans several elements. For a small adjustment, inspect that element and its layout dependencies directly.

Pay particular attention to:

- Container padding, stack gaps, safe-area placement, and fixed versus flexible dimensions.
- Font family, size, weight, width, line height, and tracking.
- Added or obsolete visual elements, artwork, fills, strokes, and opacity.
- Data or interaction requirements that a new control introduces.

Record exact old/new values when available; do not dismiss a specified difference merely because it is small. Use [visual-fidelity.md](visual-fidelity.md) when code properties and rendered appearance disagree.

## Decide and implement

Apply clear changes within the requested redesign without waiting for checklist approval. Remove obsolete decorative UI when that is part of aligning the screen. If an absent element carries existing functionality, do not infer that the functionality should be deleted from its absence in one mockup; resolve the conflict from the brief or ask a focused question.

Use existing models and actions for new UI where their meaning is clear. Ask when a new element requires an unspecified product behavior or unavailable data contract, rather than shipping hardcoded production values.

Check shared-component impact before editing a reused view or token. Prefer an existing variant or a scoped change when other screens must retain their appearance.

Complete the requested differences and relevant verification before reporting completion. Summarize behavior changes and any unresolved design or data limitations; a full audit artifact is needed only when requested.

# Scope from a Brief

Use an attached document, ticket, or inline brief to narrow the Figma work before selecting frames. Extract only what affects implementation:

- Target screens and entry point.
- Actions and their destinations or effects.
- Async work and required loading, error, empty, retry, or success states.
- Project constraints and explicit exclusions.

Keep these notes in working context unless the user requested a plan or handoff document. Do not require a filled template when the request is already clear.

## Reconcile with Figma

The user's current instruction takes precedence over an older brief. The brief supplies behavior and scope; the design supplies visuals.

| Situation | Response |
|---|---|
| A page contains more frames than the brief names | Match by names, content, and variants; ignore unrelated frames |
| The brief includes behavior absent from static mockups | Implement it using the project's existing state and interaction patterns |
| Several frames plausibly represent the requested screen | Inspect metadata or a targeted screenshot; ask if the choice remains material |
| A required screen or action has no clear design or project counterpart | Ask about the missing part while continuing resolved work |
| Figma contains extra screens or controls | Do not expand scope solely because they are visible |

Different frame counts alone are not a conflict: multiple frames may represent device sizes or states of the same screen. Do not block design discovery because counts or labels differ.

For flows, keep a compact mapping from requested screens to node IDs and navigation/actions. For one screen, its node and required states are usually enough.

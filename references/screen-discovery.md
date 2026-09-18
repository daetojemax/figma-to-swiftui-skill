# Screen Discovery

Use metadata when the target is a page, root, flow, large container, or otherwise unclear. A clear screen/component can go directly to design context.

Find the smallest frame or set of frames that covers the request. Distinguish screens from state variants, device variants, and decorative groups using names, hierarchy, dimensions, and visible content. If metadata is insufficient, inspect a targeted screenshot or child context.

For a flow, a compact working map can help:

| Requested UI | Node | Role |
|---|---|---|
| Sign in | 12:4 | Default screen |
| Sign in error | 12:8 | State of the same screen |
| Profile | 12:20 | Success destination |

Fetch detailed context only for the in-scope nodes. The map need not be shown or approved when the match is clear.

Ask before choosing between plausible targets that would produce materially different UI or navigation. Extra unrelated frames or a mix of screens and variants do not require clarification when their roles are clear.

For oversized responses, use [fetch-strategy.md](fetch-strategy.md).

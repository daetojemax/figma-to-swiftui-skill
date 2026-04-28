# Fetch Strategy

Rules for calling Figma MCP tools in a way that avoids timeouts, minimizes wasted tokens, and keeps work resumable enough for practical use.

## Core Principles

1. **Metadata before context**: `get_design_context` is the expensive, timeout-prone call. Do not call it on a node unless you are confident it is a single implementable unit.
2. **Narrow with the source document first**: when a brief or ticket exists, use its screen list and out-of-scope notes to decide which Figma nodes matter.
3. **Deduplicate by file and node**: `get_variable_defs` is per `fileKey`; assets are per source node ID.
4. **Split instead of retrying blindly**: if context is too large or truncated, use metadata to fetch smaller meaningful sections.

## Metadata-First Decision Tree

```text
Input node
├── Clearly one screen/component
│   └── Fetch design context directly
├── Root / page / probably multi-screen container
│   └── Run get_metadata first
│       ├── One clear child screen -> retarget to that child
│       └── Multiple candidates -> build screen map and ask if ambiguous
└── Unknown
    └── Run get_metadata first
```

## When to Force Screen Discovery

Run `get_metadata` before `get_design_context` when:
- node ID is file-level/root-like, such as `0:1`
- node name contains `Page`, `Flow`, `Onboarding`, `All Screens`, `Desktop`, `iPhone & iPad`
- the node has many direct children or deep nesting
- the user provided a root/page URL
- the source document names more screens than the URL obviously contains

See references/screen-discovery.md for the candidate mapping format.

## Circuit Breaker for Large or Truncated Context

If `get_design_context` times out or returns truncated output:

1. Do not retry the same node with the same parameters.
2. Run `get_metadata` on that node.
3. Pick the smallest meaningful child that matches the target section.
4. Fetch `get_design_context` on that child.
5. If still too large, split into sections such as header, body, footer, cards, or modal content.
6. Tell the user when you split a screen so they understand why multiple node contexts are used.

## Deduplicate Token Fetches

`get_variable_defs(fileKey, nodeId)` returns file-scoped variables. If multiple screens come from the same `fileKey`, fetch variables once and reuse the result for the remaining screens.

Do not call `get_variable_defs` once per screen when all screens share the same Figma file.

## Deduplicate Assets by Source Node

When the same icon or illustration source node appears on multiple screens, fetch it once and reuse it.

Recommended cache shape:

```text
.figma-cache/
  _shared/assets/
    3166_70211.png
    3166_70211.meta.json
```

Use the source node ID as the stable key, replacing `:` with `_` for filenames.

## Call Budget Sanity Check

For a single screen, the normal pattern is:
- `get_metadata`: 0-1 before context, plus 1 if context times out or assets need node IDs
- `get_design_context`: 1 per screen, plus section calls only after split
- `get_screenshot`: 1 for the screen plus one per flattened region or icon asset
- `get_variable_defs`: 1 per `fileKey`
- Code Connect lookup: optional, only if the MCP server exposes it

Exceeding this is a signal to stop and re-plan instead of continuing blind fetches.

## What Not to Do

- Do not call `get_design_context` on a root/page/flow container just to see what is there.
- Do not retry a timed-out context call with the same node.
- Do not refetch variables for every screen in the same Figma file.
- Do not skip asset node deduplication when the same icon appears repeatedly.

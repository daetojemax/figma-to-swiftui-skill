# Figma Fetch Strategy

Use the connected server's tool schemas. Tool availability and parameters vary; names below describe capabilities, not a fixed call sequence.

## Select the smallest useful call

| Need | Tool, when available |
|---|---|
| Layout and styling for a known screen/component | `get_design_context` |
| Locate frames or split a large selection | `get_metadata` |
| Inspect appearance | `get_screenshot` |
| Resolve variables/styles used by the selected nodes | `get_variable_defs` |
| Find existing code components | `get_code_connect_map` |
| Obtain export files | `download_assets` or node screenshot; see [asset-handling.md](asset-handling.md) |

Request SwiftUI through supported framework/language hints. Do not invent a `prompt` argument. For Code Connect, use the project's SwiftUI mapping label when the tool exposes framework selection.

## Recover from oversized context

When a response is truncated or a heavy selection times out, inspect metadata and fetch meaningful child sections. Preserve the parent layout relationships when composing them. Do not retry the unchanged oversized request. A transient connection failure can justify a retry after addressing its cause; repeated failure should lead to another supported route or a specific blocker report.

## Reuse evidence with its scope

Reuse current context and screenshots for the same file, node, and variant. Cache only when useful for the task; no particular cache directory or document format is required.

Variables returned for a selection are not necessarily a complete file-wide token catalog. Reuse known definitions, but fetch missing variables or modes when another screen requires them. Deduplicate assets by file, source node, and relevant export settings; different modes, variants, or scales can require different exports.

Fetch device and state variants that the request or existing app requires. Do not enumerate every sibling or component permutation by default.

Tool behavior reference: [Figma MCP tools and prompts](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/).

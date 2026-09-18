# Figma MCP Access and Recovery

Use this reference when design retrieval is unavailable or fails. Tool names, parameters, and selection support depend on the connected server; inspect its current schema rather than assuming a fixed setup.

- Remote retrieval generally needs a file key and node ID from a design link.
- Desktop selection can provide context when supported; the correct file and node must be selected in Figma.
- If the required connection or authentication is missing, explain what access is needed. Continue any independent project inspection or planning supported by existing evidence.

## Diagnose the actual failure

| Symptom | Next step |
|---|---|
| Empty context or unknown node | Check the link, node ID, accessible file, and metadata |
| Permission or authentication error | Request the missing access; repeated identical calls will not fix it |
| Expired asset URL | Refresh the relevant export or asset source, then download while valid |
| Oversized or truncated context | Fetch smaller child sections using [fetch-strategy.md](fetch-strategy.md) |
| Optional tool missing | Use an equivalent available read capability or project evidence |

Do not treat every tool error as a disconnected server. Stop dependent work when the missing evidence prevents a reliable implementation and report the specific unresolved part.

Setup references: [remote server](https://developers.figma.com/docs/figma-mcp-server/remote-server-installation/) and [desktop server](https://developers.figma.com/docs/figma-mcp-server/local-server-installation/).

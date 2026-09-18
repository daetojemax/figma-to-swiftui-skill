# Figma to SwiftUI Skill

Implement or update iOS SwiftUI UI from Figma designs using the [Figma MCP server](https://developers.figma.com/docs/figma-mcp-server/). Also supports implementation plans, token mapping, and asset export for that UI. Packaged in the [Agent Skills format](https://agentskills.io/home).

The skill focuses on design evidence, native rendering differences, and project conventions. Its short entrypoint routes to references only when needed; it does not require a fixed sequence of calls or approval for routine implementation choices.

## Install

```bash
npx skills add https://github.com/daetojemax/figma-to-swiftui-skill --skill figma-to-swiftui
```

For manual installation, clone the repository and install or symlink the folder according to your tool's skills documentation:

- [Codex](https://developers.openai.com/codex/skills/#where-to-save-skills)
- [Claude Code](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview#using-skills)
- [Cursor](https://cursor.com/docs/context/skills#enabling-skills)

Live design retrieval needs an authenticated Figma MCP connection and a design link, or a selected node on a desktop server that supports selections. Existing design evidence can be reused. An established SwiftUI project is recommended for implementation.

## Use

> Use figma-to-swiftui to implement this screen in the existing app: https://www.figma.com/design/abc123/MyApp?node-id=10-5

> Update the existing Login screen to match this Figma frame. Sign In validates email and password, disables invalid submissions, shows loading and inline errors, and navigates to Profile on success. Signup and password reset are out of scope.

> Export the icons from this Figma frame into the existing asset catalog. Do not change the screen code.

A brief or ticket can define behavior that is not visible in a static mockup. The skill uses it to select relevant frames and required states without pulling unrelated screens into scope.

## How it works

- **Focused discovery:** inspect metadata for ambiguous pages or flows; fetch detailed context for the relevant nodes. Split oversized responses and reuse current evidence.
- **Native implementation:** interpret generated code as design data, reuse suitable project components and tokens, and preserve architecture and existing behavior.
- **Real assets:** export authored icons and artwork, using PNG by default. Preserve source resolution, catalog scale, and tint. Use native geometry for structural UI and the app's image pipeline for remote content.
- **Scoped adaptation:** compare the affected existing UI with the design, including typography, spacing, artwork, and state. A checklist is working context, not an approval gate.
- **Proportionate verification:** run relevant available checks, compare rendered UI where possible, and fix observed issues. Report limitations honestly; respect explicit requests to skip checks.

Questions are reserved for material ambiguity about the target design, scope, behavior, or required missing inputs. Routine choices follow the project or a suitable native approach. Code Connect publication requires the user's request or existing authorization.

## References

| File | Use when |
|---|---|
| [SKILL.md](SKILL.md) | Selecting the workflow and completion criteria |
| [source-document.md](references/source-document.md) | Reconciling a brief with Figma |
| [screen-discovery.md](references/screen-discovery.md) | Locating screens in a page or flow |
| [fetch-strategy.md](references/fetch-strategy.md) | Choosing calls, recovering from large responses, reusing scoped evidence |
| [adaptation-workflow.md](references/adaptation-workflow.md) | Updating existing UI without losing behavior |
| [asset-handling.md](references/asset-handling.md) | Exporting and integrating real artwork |
| [design-token-mapping.md](references/design-token-mapping.md) | Mapping tokens and typography |
| [layout-translation.md](references/layout-translation.md) | Translating layout, effects, and transitions |
| [responsive-layout.md](references/responsive-layout.md) | Adapting to supported container sizes |
| [component-variants.md](references/component-variants.md) | Representing required states, sizes, and styles |
| [visual-fidelity.md](references/visual-fidelity.md) | Diagnosing visual differences |
| [figma-mcp-setup.md](references/figma-mcp-setup.md) | Resolving connection or retrieval failures |

## Contributing

Keep task selection precise, the entrypoint concise, and conditional details in relevant references. Preserve non-obvious constraints without turning one past failure into a universal procedure. This structure follows the guidance in [Rethinking skills and prompts for GPT-6 Astra](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra) while remaining model-independent.

Validate frontmatter and local links after editing. For workflow changes, check realistic cases such as a small visual adjustment, a new screen with async states, an ambiguous flow, and an asset-only request. Use real Figma and rendered SwiftUI checks when assessing end-to-end implementation quality.

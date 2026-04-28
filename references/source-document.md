# Source Document — Read Before Figma

When the user attaches or references a source document (`.txt`, `.md`, PM brief, spec, ticket description), read it before making any Figma MCP call.

## Why First

The document tells you which screens matter, which actions are in scope, and which behaviors exist beyond the static design. Without it, the agent fetches Figma blind: pulls context for frames that are not in scope, misses async behavior, and wastes tokens on mockup extras.

## Accepted Inputs

- `.txt` or `.md` files attached or pasted inline
- PM ticket description, feature brief, handoff note
- Slack/email snippets describing the flow
- A combination: document describes behavior, Figma node(s) provide visuals

If the user provides Figma only and no document, skip this step and go straight to the Figma workflow.

## What to Extract

Read the document once and produce a short contract before touching Figma:

```text
Feature: <one line>
Screens (expected): <list from doc, may not match Figma yet>
Entry point: <where the flow starts>
Actions: <per-screen primary/secondary actions>
Async work: <API calls, persistence, auth>
Required states: <loading, error, empty, retry, success>
Constraints: <libraries, architecture, tokens the project must respect>
Out of scope: <anything the doc explicitly excludes>
Unclear: <items that need user confirmation>
```

Keep it compact. This is working context, not documentation. Do not include sections the document has nothing to say about.

## Using the Extract to Drive Figma

The contract narrows the Figma work:

- **Expected screen list**: map to Figma frames via `get_metadata`. If the document says 4 screens and the Figma root has 8 frames, ask which 4; do not guess.
- **Action list**: use to resolve ambiguous elements on each screen. If multiple elements could own an action, build a candidate mapping and ask before coding.
- **Async work and required states**: implement or account for them even if Figma does not show them. Do not stop at happy-path UI when the document specifies error, retry, empty, disabled, or loading states.
- **Constraints**: drive project audit for architecture, libraries, dependencies, typography, colors, and assets.
- **Out of scope**: do not fetch or implement those parts, even if they are present in the Figma frame.

## Conflict Rules

When the document and Figma disagree:

- **Document names a screen that is not in Figma**: ask; do not invent a screen.
- **Figma has a screen the document does not mention**: ask; do not silently add it to the scope.
- **Document describes an action no button in Figma matches**: ask; do not attach the action to the wrong element.
- **Document specifies behavior not visible in Figma**: trust the document for behavior and implement the visible UI from Figma.

The document is authoritative for behavior and scope. Figma is authoritative for visuals within that scope.

## Deciding Single Screen vs Flow

Use the extracted contract to decide how much to implement:

| Signal | Behavior |
|---|---|
| Document describes 1 screen, 1 Figma node, no transitions | Continue with this skill on that screen |
| Document describes multiple screens, transitions, or a journey | Treat it as a feature-flow task: build a compact screen graph and implement each screen deliberately |
| Document describes 1 screen but Figma root has multiple frames | Run metadata-first screen discovery, then use the matched frame |
| No document, 1 Figma URL | Continue with this skill |
| No document, multiple Figma URLs or a root node | Ask whether the user wants one screen/component or the whole flow |

## Stop Rule

Do not start Figma fetches while any of these remain unresolved:

- Screen count in document vs Figma
- Entry point
- Whether async behavior is in scope
- Which Figma node maps to the primary screen or primary action

A short clarification saves minutes of wrong fetches.

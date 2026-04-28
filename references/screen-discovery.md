# Screen Discovery

Use this workflow when the provided Figma node is not obviously a single screen or component.

## When to Trigger

Run screen discovery before `get_design_context` when the input is:
- a root node such as `0:1`
- a page node
- a large frame containing several child frames
- a container that appears to hold multiple screens, onboarding steps, device variants, or state variants
- a source document names more screens than the provided Figma URL obviously contains

## Goal

Convert an ambiguous Figma container into a short, reviewable screen map before fetching expensive design context or writing code.

## Required Steps

1. Run `get_metadata` on the provided node.
2. Identify likely screen frames or top-level sections.
3. Build a candidate mapping table with confidence.
4. Continue only when the screen boundary is clear enough.

## Required Output

Before fetching or coding, produce a compact table like this:

```text
Candidate screen mapping:
- Splash -> node 12:4 -> confidence: high
- Intro 1 -> node 12:8 -> confidence: medium
- Intro 2 -> node 12:12 -> confidence: medium
- Main -> node 12:20 -> confidence: high

Ambiguities:
- node 12:8 and 12:9 may both be Intro 1 variants

Recommendation:
- use 12:8 as the default implementation node
```

## Confidence Rules

- `high`: the frame name, structure, or visible content clearly maps to one screen
- `medium`: the frame is a likely match but has competing candidates or unclear labels
- `low`: multiple candidates are plausible or the node may not be a screen at all

## Stop Conditions

Stop and ask the user before implementation when:
- any critical screen maps only with `low` confidence
- two or more candidates would materially change the generated UI or flow
- the node appears to mix screen states and screens at the same level
- the source document and Figma structure disagree on screen count

## Default Rule

Do not silently pick a child frame from a root node when another child is a plausible match for the same screen.

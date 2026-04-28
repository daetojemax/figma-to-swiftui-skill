---
name: figma-to-swiftui
description: Convert Figma URLs, nodes, selections, or briefs into production SwiftUI for iOS using Figma MCP. Use for Figma-to-SwiftUI implementation, planning, design tokens, and asset export; not for web or React.
---

# Figma to SwiftUI Implementation Skill

Translate Figma nodes into production-ready SwiftUI views with pixel-perfect accuracy. Combines MCP integration rules with a structured implementation workflow for iOS projects.

## Prerequisites

- Figma MCP server must be connected and accessible
- User must provide a Figma URL, e.g.: https://www.figma.com/design/:fileKey/:fileName?node-id=3166-70147&m=dev
  - May include &m=dev or other query params — only node-id matters
  - :fileKey — path segment after /design/
  - node-id value — the specific component or frame to implement
- OR when using figma-desktop MCP: select a node directly in the Figma desktop app (no URL required)
- Xcode project with an established SwiftUI codebase (preferred)
- Optional `.txt` / `.md` / ticket / brief describing scope, behavior, actions, and states

## MCP Connection

If any MCP call fails because Figma MCP is not connected, pause and ask the user to configure it. See references/figma-mcp-setup.md for troubleshooting.

---

## Workflow

Follow these steps in order. Do not skip steps.

**Two modes:** If the user wants to build a new screen from scratch, follow all steps sequentially. If the user wants to adapt/update an existing screen to match a Figma design, follow Steps 1–5, then do Step 5b (Adaptation Audit) before Step 6. Step 5b ensures every difference between the existing code and the design is identified and addressed — this is where most mistakes happen during adaptation.

### Step 0 — Read Source Document (if provided)

If the user provides a `.txt`, `.md`, ticket, PM brief, or inline spec together with Figma work, read it before any Figma MCP call. Extract the feature goal, expected screens, entry point, actions, async work, required states, constraints, out-of-scope items, and unclear points. See references/source-document.md.

Use the extracted contract to narrow the Figma work. If the document and Figma disagree on screen count, scope, or primary action mapping, ask before fetching or implementing the wrong node.

### Step 1 — Parse the Figma URL

Extract fileKey and nodeId from the URL.

Accepted URL patterns (with or without www.):
- figma.com/design/:fileKey/:fileName?node-id=...
- figma.com/file/:fileKey/:fileName?node-id=... (legacy, same behavior)

Parsing rules:
- fileKey: first path segment after /design/ or /file/
- nodeId: value of node-id query parameter. Always replace "-" with ":" (URLs use "3166-70147", MCP expects "3166:70147")
- Ignore all other query parameters (m=dev, t=..., page-id=..., etc.)
- Reject /proto/ and /board/ URLs — they are prototypes and FigJam boards, not implementable designs. Ask the user for a /design/ link instead.

When using figma-desktop MCP without a URL, tools automatically use the currently selected node. Only nodeId is needed; fileKey is inferred.

### Step 1b — Screen Discovery (metadata-first when needed)

Before calling `get_design_context`, decide whether the node is clearly one implementable screen/component. If the URL points to a root node, page node, large container, flow, multi-screen frame, or a source document names more screens than the URL obviously contains, run `get_metadata` first.

Build a candidate screen map with confidence and continue only when the target node is clear. If multiple candidates would materially change the implementation, ask the user before fetching design context. See references/screen-discovery.md and references/fetch-strategy.md.

### Step 2 — Fetch Design Context

get_design_context(fileKey=":fileKey", nodeId="1-2", prompt="generate for iOS using SwiftUI")

The `prompt` parameter steers the default code output toward SwiftUI. You can also pass project-specific hints: `"use components from Components/"`, `"generate using my design system tokens"`.

Returns structured design data: layout, typography, colors, spacing, and a code representation. Even with an iOS prompt, treat the output as a design specification, not code to port.

For large/complex designs: If the response is truncated or times out, do not retry the same node. Run `get_metadata`, identify smaller sections and child IDs, then fetch each section individually. See references/fetch-strategy.md.

For multi-device designs: If Figma contains frames for different screen sizes (iPhone + iPad), fetch all device-specific frames, not just one. See references/responsive-layout.md for merging them into adaptive SwiftUI views.

### Step 3 — Capture Screenshot

get_screenshot(fileKey=":fileKey", nodeId="1-2")

This screenshot is the source of truth for visual validation throughout implementation.

### Step 4 — Fetch Design Tokens (if available)

get_variable_defs(fileKey=":fileKey", nodeId="1-2")

Returns colors, spacing, typography tokens. Map to the project's SwiftUI design system. See references/design-token-mapping.md.

### Step 5 — Build Asset Inventory and Download Assets

Before writing SwiftUI, build a visual asset inventory from the screenshot and design context. See references/asset-handling.md for the full decision flow.

1. Open the screenshot from Step 3 and list every visible non-text element: icons, logos, photos, illustrations, decorative artwork, and image fills
2. Cross-check each row against `get_design_context` and `get_metadata` to find localhost URLs, image fills, vector nodes, and node IDs
3. Classify each row as `download`, `code`, or `remote`
4. Download Figma-owned assets during the active MCP session:
   - Use `get_screenshot(fileKey, nodeId)` by default for icons, logos, illustrations, and artwork nodes
   - Use localhost URLs from `get_design_context` only when they validate as PNG
5. Validate downloaded files with `file <asset>`: visible Figma assets must be real PNG files, then add them to Asset Catalog with @1x/@2x/@3x variants and the correct rendering mode

Asset rules:
- Figma assets first: do NOT replace Figma icons, logos, or illustrations with SF Symbols or hand-drawn SwiftUI shapes
- Visible Figma-owned assets should be exported as Figma-rendered PNG by default; SVG/text/XML responses are failed exports, not final assets
- SF Symbols are allowed only for iOS system chrome or when the user explicitly approves substitution
- Do NOT create placeholder images or fake logos with `Text`, `Rectangle`, `Circle`, or custom `Shape`
- If a visible asset has no download URL and no identifiable node ID, stop and ask the user instead of improvising
- Do NOT import new icon packages unless the project already uses them
- Remote content images (avatars, feeds, CDN photos) should use the project's existing image loading path, not bundled assets

### Step 5b — Adaptation Audit (when modifying an existing screen)

When the user asks to adapt/update an existing screen to match a Figma design, perform a full element-by-element audit before writing any code. See **references/adaptation-workflow.md** for the complete process.

Key steps:
1. Read the existing code and all its subcomponents
2. Build a categorized diff checklist (ADD / UPDATE / REMOVE) with exact old → new values
3. Pay special attention to spacing — it's the most commonly missed difference
4. Present the checklist to the user and clarify unknowns before implementing
5. Apply all changes — do not skip items that seem minor

### Step 6 — Implement in SwiftUI

Before writing any code:

1. Run `get_code_connect_map(fileKey, nodeId)` to check if Figma components in the design already have mapped code components. If a mapping exists — use that code directly instead of building from scratch.
2. Inspect the project's dependencies (Package.swift, Podfile, .xcodeproj) and existing codebase for UI-related libraries and patterns. The project may use third-party solutions for things you would otherwise implement with native SwiftUI. Examples:
- Image loading: Kingfisher, SDWebImage, Nuke instead of AsyncImage
- Animations: Lottie instead of SwiftUI animations
- UI components: custom design system, SnapKit for layout, etc.
- Networking + image caching: Alamofire, custom image cache
- Charts: Charts library instead of Swift Charts

Use whatever the project already uses. Do not introduce native SwiftUI alternatives if the project has an established library for that purpose. If the design requires something the project has no dependency for, ask the user before choosing an approach.

Critical rule: MCP output (React + Tailwind) is a representation of design intent. Do NOT port React to SwiftUI. Read design properties and build native SwiftUI views from scratch.

Read references/visual-fidelity.md before implementing non-trivial screens. Use it for exact value extraction, source-of-truth priority, visual inventory, SwiftUI default pitfalls, and screenshot cross-check.

Asset self-check before coding:
- Every `Image(...)` must reference a downloaded Figma asset, a remote image loaded through the project image pipeline, or an explicitly allowed system symbol
- Every Figma icon/logo/illustration in the visual inventory must have a corresponding Asset Catalog entry or approved remote source
- `Image(systemName:)` is not allowed for Figma-designed assets unless the user approved that substitution
- `Text("G")`, colored shapes, custom `Shape`, or approximate vector drawings must not stand in for logos, app icons, social icons, or illustrated assets

Do NOT implement system-provided elements that appear in Figma mockups. Designers often include them for context, but they are rendered by iOS automatically. Skip these:
- Keyboard (system keyboard, emoji picker)
- Status bar (time, battery, signal)
- Home indicator bar
- Navigation bar back button (provided by NavigationStack)
- Tab bar if using native TabView (only implement custom tab bars)
- System alerts and action sheets (use .alert() / .confirmationDialog())
- Share sheet (use ShareLink or UIActivityViewController)
- System search bar (use .searchable())
- Pull-to-refresh indicator (use .refreshable())
- Page indicator dots for native TabView with .page style

If unsure whether an element is system-provided or custom, ask the user.

#### 6.1 — Layout Translation

See references/layout-translation.md for the complete mapping. Key rules:

Figma Auto Layout (vertical) -> VStack(spacing:) with matching alignment
Figma Auto Layout (horizontal) -> HStack(spacing:) with matching alignment
Figma Auto Layout with wrap -> LazyVGrid or custom FlowLayout
Figma Frame with absolute children -> ZStack + .offset() (avoid when possible)
Figma padding -> .padding(.horizontal, 16) edge-specific
Figma gap -> spacing parameter in stack initializer
Figma fill container -> .frame(maxWidth: .infinity)
Figma hug contents -> No frame modifier (intrinsic sizing, SwiftUI default)
Figma fixed size -> .frame(width:, height:)
Figma aspect ratio -> .aspectRatio(ratio, contentMode:)
Figma scroll -> ScrollView(.vertical) or .horizontal
Figma constraints (pin to edges) -> .frame() + alignment in parent
Figma frame sized for specific device -> Check if project supports multiple devices, adapt with size classes. See references/responsive-layout.md

#### 6.2 — Typography Translation

Figma font family -> Closest iOS system font or project custom font
Figma font weight -> Font.Weight (.regular, .medium, .semibold, .bold)
Figma font size -> .font(.system(size:, weight:, design:)) or custom Font extension
Figma line height -> .lineSpacing(lineHeight - fontSize)
Figma letter spacing -> .tracking() (Figma px = SwiftUI points, 1:1 on iOS)

If the project has a typography system (Typography.headline), prefer project tokens over raw values.

#### 6.3 — Color Translation

Figma hex color -> Color from Asset Catalog or Color(hex:) extension
Figma color + opacity -> .opacity() modifier or color with alpha
Figma linear gradient -> LinearGradient(colors:, startPoint:, endPoint:)
Figma radial gradient -> RadialGradient(colors:, center:, startRadius:, endRadius:)
Figma color variables -> Map to project tokens (Color.primaryText, Color.surface)
Figma dark mode variants -> Adaptive colors in Asset Catalog or @Environment(\.colorScheme)

When conflicts arise between project tokens and Figma specs, prefer project tokens but adjust minimally to match visuals.

#### 6.4 — Component Translation

Figma component instance -> Check for existing view in project. Reuse over creating new.
Figma button -> Button + project .buttonStyle()
Figma text input -> TextField or TextEditor
Figma toggle -> Toggle with custom style if design differs from system
Figma image -> Image from Asset Catalog for local. For remote URLs, use the project's image loading library; if none exists, ask the user.
Figma list/collection -> List or LazyVStack / LazyVGrid
Figma tab bar -> TabView or custom tab bar if non-standard
Figma navigation bar -> .navigationTitle() + .toolbar {} or custom header
Figma sheet/modal -> .sheet() / .fullScreenCover() — sheet manages own dismiss
Figma card -> Custom view + .background() + .clipShape(.rect(cornerRadius:)) + .shadow()
Figma component with variants -> Check variant properties, summarize detected variants, ask user which style approach to use, then implement. See references/component-variants.md for translating Figma variant properties (state, size, style, content toggles) into SwiftUI. Always ask the user which implementation approach they prefer before writing variant code.

#### 6.5 — Effects and Decorations

Figma drop shadow -> .shadow(color:, radius:, x:, y:)
Figma inner shadow -> .overlay() with shadow or custom shape stroke
Figma blur (layer) -> .blur(radius:)
Figma blur (background) -> .background(.ultraThinMaterial) or .regularMaterial
Figma corner radius -> .clipShape(.rect(cornerRadius:))
Figma individual corners -> UnevenRoundedRectangle(topLeadingRadius:, ...)
Figma border/stroke -> .overlay(RoundedRectangle(...).stroke(...))
Figma clip content -> .clipped() or .clipShape()
Figma mask -> .mask { ... }
Figma blend mode -> .blendMode()
Figma Liquid Glass (iOS 26+) -> .glassEffect() with appropriate shape

#### 6.6 — Animations and Transitions

Figma prototype connections define transitions between frames. Interpret them as navigation or state-change animations — not as literal animation specs.

Figma dissolve -> .opacity() + withAnimation(.easeInOut)
Figma move in / slide in -> .transition(.move(edge:)) or .offset()
Figma push -> NavigationStack push (system transition)
Figma smart animate -> withAnimation { } on state change, match property diffs (position, size, opacity)
Figma scroll animate -> ScrollView with .scrollTransition() or .animation() on offset

Common SwiftUI patterns:
- State-driven: `withAnimation(.spring) { showDetail = true }` + `.transition(.move(edge: .trailing))`
- Matched geometry: `matchedGeometryEffect(id:in:)` for shared element transitions between views
- Implicit: `.animation(.default, value: someState)` on the animating view

Rules:
- Check project dependencies for Lottie or other animation libraries — use them if present
- Do not over-animate. If Figma shows a transition only between screens (prototype link), implement it as navigation, not a custom animation
- If the design includes complex choreographed animations (multiple elements, sequenced timing), ask the user whether to implement fully or simplify
- Figma prototype delays and durations are hints, not exact specs — use standard iOS timing (.default, .spring) unless the user specifies otherwise

### Step 7 — Validate (on user request only)

Do NOT auto-validate. Before starting implementation (after Step 5), ask the user how they want to validate the result. Examples:
- Compare screenshot side-by-side in Xcode preview
- Run on simulator and compare manually
- Use snapshot testing
- No validation needed, trust the implementation

Proceed with whichever method the user chooses. If the user does not specify, skip validation entirely.

Reference checklist (share with user if they ask what to check):
- Layout: spacing, alignment, sizing
- Typography: font, size, weight, line height
- Colors: fills, strokes, backgrounds, text
- Assets: all icons/images present, no placeholders
- Interactive states: press, focus, disabled
- Dark mode (if Figma provides variants)
- Dynamic Type: text scales appropriately
- Safe areas: no content behind notch / home indicator
- Scroll behavior correct if design implies scrollable content

For deeper visual QA, use references/visual-fidelity.md as the checklist.

If deviating from Figma (accessibility, platform conventions, technical constraints), document why in comments.

### Step 8 — Register Code Connect Mappings

After creating reusable SwiftUI components that correspond to Figma components, register them:

add_code_connect_map(fileKey, nodeId, componentPath, componentName)

This links the Figma component to your code so future designs using the same component will reference the existing implementation instead of generating new code.

Only register components that are:
- Reusable (used in multiple places or likely to be reused)
- Stable (not a one-off screen-specific view)

---

## Handling Complex Designs

1. get_metadata to get the node tree
2. Identify major sections and child node IDs
3. Implement top-down: container first, then sections
4. get_design_context + get_screenshot per section
5. If user requested validation, validate per section, then full composition

## MCP Tools Reference

get_design_context: Design data + default code + asset download URLs. Use always, primary source.
get_metadata: Sparse node tree. Use for large designs, structure first.
get_screenshot: Visual reference PNG. Use always, validation truth.
get_variable_defs: Design tokens. Use when project has design system tokens.
get_code_connect_map: Existing code mappings. Use before creating components.
add_code_connect_map: Register new mappings. Use after creating reusable components.

## Key Principles

1. Never implement from assumptions. Always fetch context + screenshot first.
2. MCP output is a spec, not code. Read properties, build native SwiftUI.
3. Use what the project uses. Check dependencies and existing patterns before implementing anything. Do not introduce native alternatives if the project already has a library for that purpose.
4. Project tokens win. Prefer project tokens, adjust minimally for visual match.
5. Validate only when asked. Ask the user how they want to validate before implementing.
6. Figma assets first. Use SF Symbols only for iOS system chrome or user-approved substitutions; never replace Figma-designed icons, logos, or illustrations by default.
7. Platform conventions matter. iOS navigation, safe areas, Dynamic Type, accessibility are more important than pixel-perfect Figma replication.

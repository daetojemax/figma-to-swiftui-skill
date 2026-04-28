# Visual Fidelity Playbook

The process and checklist that makes SwiftUI output match the Figma design. This is the authoritative reference for "make it look like Figma".

## Contents

- [1. Parsing `design-context.md`](#1-parsing-design-contextmd)
- [2. Source-of-Truth Priority](#2-source-of-truth-priority)
- [3. Visual Inventory Template](#3-visual-inventory-template)
- [4. SwiftUI Defaults That Break Fidelity](#4-swiftui-defaults-that-break-fidelity)
- [5. Screenshot Cross-Check](#5-screenshot-cross-check)
- [6. Common pre-flight checks](#6-common-pre-flight-checks)
- [7. Hard Rules](#7-hard-rules)

This file covers:
1. How to parse `design-context.md` (the MCP response) into exact values
2. Source-of-truth rules when multiple caches disagree
3. The Visual Inventory template (rigorous)
4. SwiftUI defaults that silently break fidelity
5. The screenshot cross-check process

---

## 1. Parsing `design-context.md`

`get_design_context` returns React + Tailwind-ish code plus inline `style` objects. It is a **specification carrier**, not code to port. Extract exact values from it.

### Tailwind class → exact value

Default Tailwind scale (Figma MCP uses this unless the value doesn't match, in which case it uses `[arbitrary]` syntax):

| Class | Value |
|---|---|
| `p-1`, `p-2`, `p-3`, `p-4`, `p-6`, `p-8` | 4, 8, 12, 16, 24, 32pt |
| `gap-1`, `gap-2`, `gap-3`, `gap-4`, `gap-6` | 4, 8, 12, 16, 24pt |
| `text-xs` / `text-sm` / `text-base` / `text-lg` / `text-xl` / `text-2xl` / `text-3xl` | 12 / 14 / 16 / 18 / 20 / 24 / 30pt |
| `font-normal` / `medium` / `semibold` / `bold` | .regular / .medium / .semibold / .bold |
| `rounded` / `-md` / `-lg` / `-xl` / `-2xl` / `-3xl` / `-full` | 4 / 6 / 8 / 12 / 16 / 24 / Capsule |
| `leading-none` / `-tight` / `-snug` / `-normal` / `-relaxed` / `-loose` | 1.0 / 1.25 / 1.375 / 1.5 / 1.625 / 2.0 × fontSize |
| `tracking-tight` / `-normal` / `-wide` | -0.025em / 0 / 0.025em |

### Arbitrary values — take them literally

When Figma values don't match the default scale, MCP emits `[arbitrary]`:
- `p-[17px]` → 17pt padding (not 16)
- `text-[15px]` → 15pt font (not 14 or 16)
- `rounded-[10px]` → 10pt corner radius
- `leading-[22px]` → 22pt line height (absolute, not multiplier)
- `tracking-[-0.32px]` → -0.32pt letter spacing

**Rule: arbitrary values are authoritative. Never round them to the nearest Tailwind default.**

### Color classes

- `bg-white`, `bg-black` → `Color.white`, `Color.black`
- `bg-[#F5F5F5]` → `Color(hex: "F5F5F5")` or matching project color
- `bg-white/50` → `.white.opacity(0.5)`
- `bg-[#000000]/80` → `Color(hex: "000000").opacity(0.8)`
- Tailwind palette (`bg-gray-500`, etc.) → convert via tokens.json if project maps them; otherwise take the hex from the inline style that MCP usually emits alongside.

### Inline style blocks

MCP often emits explicit style objects that override Tailwind. Trust these over class names:

```jsx
style={{
  fontFamily: 'SF Pro Display',
  fontWeight: '600',
  fontSize: '17px',
  lineHeight: '22px',
  letterSpacing: '-0.32px',
  color: '#1C1C1E'
}}
```

Every field here is a value to carry into SwiftUI. **Do not drop fields** because "Font.system handles it by default" — it doesn't.

### Figma-specific comments

MCP sometimes injects comments like `// Auto layout: vertical, gap 12, padding 16` or `// Fill: linear gradient from #FF0080 to #7928CA`. These are authoritative when present.

---

## 2. Source-of-Truth Priority

When multiple caches disagree on a value:

```
tokens.json  >  inline style in design-context.md  >  Tailwind class in design-context.md  >  screenshot.png (estimate)
```

Rules:
- **tokens.json** wins for anything that has a design token (colors, spacing tokens, typography). If a variable is defined, use it — not the literal hex that happens to be on this frame.
- **Inline style** wins over Tailwind class when both are present (because inline is literal, class may be rounded).
- **Screenshot** is the tiebreaker when context is ambiguous or truncated, AND the final verification surface. Never pull an exact value from the screenshot (reading pixels from a PNG is unreliable) — use it to confirm, not to measure.
- **Code Connect map** (`code-connect.json`) overrides everything for components that have an existing mapping — use the mapped component and let it handle internal values.

---

## 3. Visual Inventory Template

For every screen or component, produce this before writing SwiftUI. Keep it in scratch context unless the user asks for an implementation plan or audit artifact.

```
─────────── CONTAINER ───────────
size:           W x H  (fixed / fill-width / fill-height / hug)
background:     <hex or token>   [source: tokens | inline | class]
cornerRadius:   Xpt              [source]
border:         Xpt, <hex>       [source]
shadow:         color=<rgba>, radius=X, offsetX=X, offsetY=X   [source]
padding:        top=X, leading=X, bottom=X, trailing=X

─────────── LAYOUT ───────────
type:           VStack | HStack | ZStack | ScrollView + stack | LazyVGrid | GeometryReader
spacing:        Xpt
alignment:      leading | center | trailing (+ cross-axis)

─────────── ELEMENTS ─────────── (one row per visible element)
[n] <kind> "<label or asset name>"
    position:    index in stack | offset (x,y) for ZStack
    size:        WxH (fixed / hug / fill)
    — if Text:
    font:        family=<name>, size=X, weight=<w>, width=<std|expanded>
    lineHeight:  Xpt absolute  OR  Xx multiplier     [critical — never skip]
    letterSpacing: X pt
    color:       <hex or token>
    alignment:   leading/center/trailing
    lineLimit:   N
    — if Image/Icon:
    asset:       <name>
    size:        WxH
    renderingMode: original | template (+ tint color)
    — if Button/Control:
    style:       <variant>
    sizeVariant: small|medium|large
    background:  <hex>
    foreground:  <hex>
    cornerRadius: Xpt
    padding:     X x X
    — if Shape:
    shape:       RoundedRectangle | Capsule | Circle | Custom
    fill:        <hex or gradient spec>
    stroke:      Xpt <hex>
```

**Rules:**
- Every visible element from the screenshot must have an entry.
- Every value must have a source tag `[tokens | inline | class | screenshot]`.
- If a field is genuinely unknown, write `[estimate]` — never silently omit.
- `lineHeight`, `letterSpacing`, `shadow`, `border` are skipped most often. Never skip them if design-context mentions them.
- If Figma uses `Expanded` or `Condensed` width, record it — SwiftUI needs `.fontWidth(.expanded)`, not just weight.

---

## 4. SwiftUI Defaults That Break Fidelity

These are the silent killers. Watch for each:

### Text
- `.font(.system(size: X))` has its own default line height (~1.17×). Figma `line-height: Xpx` rarely matches. Use `.lineSpacing(Y)` where `Y = lineHeight - fontSize`, then cancel SwiftUI's default with `.padding(.vertical, -((Y)/2))` if it over-pads. Alternative: wrap in a `Text` with explicit frame + `.lineSpacing`.
- Default letter spacing ≠ 0. Figma `letter-spacing: -0.32px` must be applied via `.tracking(-0.32)` or `.kerning(-0.32)`. They differ — `.kerning` applies between all chars, `.tracking` respects font ligatures. Prefer `.tracking`.
- `Text` ignores leading whitespace / truncates differently than Figma at narrow widths — test with real content length.

### Button
- A plain `Button("Label") { }` applies system button styling (blue foreground, press animation, implicit padding in some contexts). Wrap in `.buttonStyle(.plain)` when Figma button is custom-styled.
- `.plain` removes ALL styling — you must re-add background, padding, foreground manually.

### Image
- Assets default to `.renderingMode(.original)` unless the asset catalog is set to template. When the icon should tint with foreground color, set `.renderingMode(.template)` + `.foregroundStyle(...)`.
- `.resizable()` is required for any non-SF-Symbol image you want to size. Without it, size is intrinsic.
- `.aspectRatio(contentMode: .fit)` vs `.fill` — `.fit` letterboxes, `.fill` crops. Match what Figma clipping does.

### Padding & Spacing
- `.padding()` with no argument uses system default (~16pt). Always specify: `.padding(16)`.
- SwiftUI `VStack` spacing default is ~8pt. Always specify: `VStack(spacing: 0)` if no gap.
- Applying `.padding` AFTER `.background` vs BEFORE changes result. Figma inner padding = `.padding` first, then `.background`.

### Shape / Background
- `.background(Color.X)` extends behind the full frame. To clip with radius: `.background(Color.X, in: RoundedRectangle(cornerRadius: R))` or `.background(Color.X).clipShape(.rect(cornerRadius: R))`.
- `.cornerRadius(R)` is deprecated — use `.clipShape(.rect(cornerRadius: R))`.
- A `Rectangle().fill(...)` does NOT include a stroke — add `.overlay { RoundedRectangle(...).stroke(...) }` for borders.

### Shadow
- `.shadow(radius: R)` defaults to black at 33% opacity. Figma shadows are rarely this. Use full form: `.shadow(color: .black.opacity(0.1), radius: R, x: X, y: Y)`.
- Shadow blur radius in Figma ≠ SwiftUI radius 1:1 — Figma blur is Gaussian σ, SwiftUI `radius` is similar but tuning may be needed. Start with exact match; adjust if shadow is too hard/soft.

### Lists & Forms
- `List` adds section insets, row separators, and a form-like background. For a plain stack-of-rows that matches Figma, use `ScrollView { LazyVStack { ... } }` instead of `List`.
- `.listStyle(.plain)` removes some styling but not all. When Figma shows a flat list, prefer `LazyVStack`.

### NavigationStack
- Adds large title padding by default. Use `.navigationBarTitleDisplayMode(.inline)` when Figma header is compact, or `.toolbar(.hidden, for: .navigationBar)` for fully custom headers.
- Adds a back button and safe-area insets automatically. Factor these into layout.

### Font width / tracking (iOS 16+)
- Figma "Expanded Semibold" = `.fontWeight(.semibold).fontWidth(.expanded)`. Using only `.semibold` loses the expanded width.
- Figma "Condensed" = `.fontWidth(.condensed)`.

### Dynamic Type
- `.font(.body)` scales with user settings. For Figma's exact pt size, use `.font(.system(size: X))` — but this loses Dynamic Type. For pixel-match + DT, use `@ScaledMetric`.
- If the user's goal is strict fidelity, prefer fixed sizes. Document the tradeoff.

---

## 5. Screenshot Cross-Check

The screenshot is the final arbiter. Use it actively:

### Before implementing a section
1. Look at `screenshot.png`.
2. Identify and strip iOS system chrome first. The frame often includes a mockup of status bar (time "9:41", Dynamic Island, signal/wifi/battery) and home indicator (~134×5pt bar at bottom). These are not content — iOS renders them. Do not inventory them, do not code them.
3. Mentally zoom to the section you're about to implement.
4. Note anything the design-context didn't mention (subtle gradients, blur, inner shadows, overlapping elements, opacity layers).
5. Add missing items to the Visual Inventory.

### After implementing
1. Re-open `screenshot.png` in your mind.
2. Go through the implemented code top-down.
3. For each `.padding`, `.spacing`, `.font`, `.foregroundStyle`, `.background`, `.frame`, `.shadow` — ask: "does this match what I see?"
4. Special scan for commonly-missed items:
   - [ ] Line height (if Text has multi-line content)
   - [ ] Letter spacing / tracking
   - [ ] Inner shadow / outer shadow
   - [ ] Border + radius combined
   - [ ] Opacity on sub-elements
   - [ ] Icon rendering mode (tint vs original)
   - [ ] Icon exact pixel size (not just "small")
   - [ ] Divider color / opacity / height
   - [ ] Background material (blur) behind text
   - [ ] Text truncation / line limit
   - [ ] Gradient direction (top→bottom vs diagonal)
   - [ ] Safe area behavior (does bg extend under status bar?)
   - [ ] **No system chrome drawn** — no "9:41", no wifi/battery SF Symbols, no `Capsule`/`RoundedRectangle` at (~134×5) pretending to be the home indicator. iOS renders these.

### Disagreement handling
When code reflects inventory correctly but doesn't look right against the screenshot:
- The inventory is wrong. Re-parse design-context.
- Or the screenshot shows something design-context didn't specify. Add it.
- Never "tweak until it looks right" without tracing the source.

---

## 6. Common pre-flight checks

Before writing SwiftUI that references tokens/APIs, confirm the project supports them.

### `Color(hex:)` is not built-in

SwiftUI has no `Color(hex: "FF0080")` initializer. Every project that uses one has defined an extension. Before generating code:

1. Grep the project for `extension Color` or `Color(hex:` — confirm an extension exists.
2. If it exists, use it verbatim (match the signature — some take `String`, some take `UInt`, some support alpha as a separate param).
3. If it doesn't exist, use one of:
   - Asset Catalog named color: `Color("accentPrimary")` — preferred, supports dark mode
   - RGB initializer: `Color(red: 1.0, green: 0.0, blue: 0.5)` — decimal 0-1 values
   - System color: `.accentColor`, `.primary`, `.secondary` when semantic

Never leave `Color(hex:)` in code if the extension isn't defined — it won't compile.

### iOS deployment target

Before using any of these modifiers, check the project's deployment target (`IPHONEOS_DEPLOYMENT_TARGET` in `.xcodeproj`, or `platforms:` in `Package.swift`):

| API | Minimum iOS |
|---|---|
| `.scrollTargetBehavior`, `.scrollTransition`, `containerRelativeFrame`, `@Observable`, `Grid` | iOS 17 |
| `.onScrollGeometryChange`, `.scrollPosition`, `@Entry` | iOS 18 |
| `.glassEffect`, Liquid Glass | iOS 26 |
| `NavigationStack`, `NavigationPath` | iOS 16 |
| `ViewThatFits`, `AnyLayout`, `Grid` | iOS 16 |
| `.fontWidth(.expanded/.condensed)`, `.tracking` | iOS 16 |

If project target is lower, use the older equivalent (e.g. iOS 16 use `LazyVGrid` instead of `Grid`, `NavigationView` only when target <16).

### Localization

Figma strings → `Text("localizable_key")` using `LocalizedStringKey`, not hardcoded strings, unless the project is explicitly single-locale.

- Default `Text("...")` parameter is `LocalizedStringKey` — string literals are auto-localized through `Localizable.strings` / `.xcstrings`.
- When the string is dynamic data (user content, API response), use `Text(verbatim:)` to skip localization lookup.
- Check the project for existing `.strings` / `.xcstrings` files. If present, add new keys there. If absent and the project has multiple feature folders, ask the user about localization strategy.

### Dark mode

- **If Figma provides both light + dark variants** for a frame: fetch both (call `get_screenshot` on each variant node), add both to the cache, and use Asset Catalog "Any / Dark" appearances for colors and images.
- **If Figma provides light only:** ask the user whether dark mode is in scope. If yes, use Asset Catalog semantic colors that auto-adapt (iOS system grays, or named colors with both appearances) — don't hardcode hex that looks wrong in dark. If dark is out of scope, still use Asset Catalog colors so dark mode doesn't render completely broken.
- For images with a dark variant: use Asset Catalog `appearances` in the PNG imageset.
- For template icons: tint with a semantic color (`.foregroundStyle(Color.textPrimary)`) — it adapts for free.

### Placeholder / mockup text

Figma frequently contains placeholder copy: `Lorem ipsum...`, `Body text`, `Title`, `Placeholder`, `Sample text`, `Username`, `example@email.com`, `$0.00`. If you copy this into SwiftUI as `Text("Lorem ipsum")`, placeholder content ships.

Rule:
- Detect these patterns in the Visual Inventory. Flag each with `[PLACEHOLDER]`.
- In code, either: (a) ask the user for real copy, (b) bind to a model property (`Text(viewModel.title)`) with a TODO, (c) use a short semantic localizable key (`Text("profile.title")`).
- Never ship `Text("Lorem ipsum dolor sit amet")` literal.

---

## 7. Hard Rules

1. Every magic number in SwiftUI code must be traceable to a source (tokens, inline style, class, or design-context comment). If you can't trace it, you guessed.
2. Never use `.font(.body)` / `.title` / etc. unless the design-context explicitly maps to iOS text styles. Figma sizes are absolute.
3. Never approximate. 17pt is not 16pt. #F5F5F7 is not #F5F5F5.
4. Never skip `lineHeight`, `letterSpacing`, `shadow`, `border` when design-context specifies them.
5. Always set `.buttonStyle(.plain)` on custom-styled buttons to disable system styling.
6. Always set `.renderingMode` explicitly on `Image` — don't rely on the asset catalog default.
7. Always specify `VStack(spacing:)` and `.padding(X)` with explicit values — never rely on SwiftUI defaults.
8. Before saying "done", do the screenshot cross-check in Section 5.

# Asset Handling for iOS/SwiftUI

Process Figma assets for Xcode without substituting or approximating designer-authored visuals.

## Contents

- [Core Rule: Figma Assets First](#core-rule-figma-assets-first)
- [1. Build the Visual Asset Inventory](#1-build-the-visual-asset-inventory)
- [2. Choose a Strategy](#2-choose-a-strategy)
- [3. Download from Figma](#3-download-from-figma)
- [4. Name and Deduplicate](#4-name-and-deduplicate)
- [5. Add PNG Images to Asset Catalog](#5-add-png-images-to-asset-catalog)
- [6. Choose Rendering Mode](#6-choose-rendering-mode)
- [7. Use Assets in SwiftUI](#7-use-assets-in-swiftui)
- [8. Final Self-Check](#8-final-self-check)
- [Asset Rules Summary](#asset-rules-summary)

## Core Rule: Figma Assets First

Every visible Figma-owned icon, logo, illustration, photo, and decorative graphic must be represented by a real Figma-rendered PNG export unless the user explicitly approves a substitution.

Banned substitutions:
- `Image(systemName:)` for a Figma-designed icon
- `Text("G")`, `Text("f")`, or similar fake logo text
- `Rectangle`, `Circle`, or custom `Shape` standing in for a logo or icon
- Simplified SwiftUI redraws of illustrations or complex artwork
- Placeholder images for assets visible in the design

Allowed exceptions:
- iOS system chrome that should remain system-provided: navigation back chevron, ShareLink/share sheet icon, native tab bar symbols, alerts, keyboard, status bar, home indicator
- User-approved substitution after you present the Figma asset and proposed SF Symbol or platform-agnostic alternative
- Trivial UI geometry: card backgrounds, dividers, dots, simple badges, solid/gradient backgrounds
- Remote content: user avatars, feed images, or CDN photos that are data-driven rather than bundled app assets

Cross-platform caveat: if the project targets multiple platforms (Skip, KMP, shared design system, or Android parity), do not silently use SF Symbols. Present each icon with the proposed replacement and ask the user to approve SF Symbols, custom Figma exports, or platform-agnostic alternatives.

## 1. Build the Visual Asset Inventory

Start from the screenshot, not the JSX-like code. List every visible non-text element before implementation.

Use this table:

| # | Purpose | Source | Node ID / URL | Strategy | Filename | Notes |
|---|---|---|---|---|---|---|
| 1 | Close icon | metadata | 3166:70211 | download | closeIcon | template |
| 2 | Hero artwork | screenshot | 3166:70200 | download | onboardingHero | original |
| 3 | Avatar photo | remote | data model | remote | n/a | use existing image loader |
| 4 | Card background | code | n/a | code | n/a | rounded rect + gradient |

Cross-check three sources:
- Screenshot: ground truth for what is visible
- `get_design_context`: localhost asset URLs, image fills, vector snippets, `imageRef`, layer names
- `get_metadata`: node IDs for `VECTOR`, `BOOLEAN_OPERATION`, `INSTANCE`, logos, icons, and illustration frames

Rules:
- Every visible non-text element gets one row
- If an icon/logo/illustration is visible but no node ID or URL can be found, stop and ask the user
- Do not remove rows because an SF Symbol looks similar
- Do not count iOS system chrome as an app asset

## 2. Choose a Strategy

### Download

Export from Figma as PNG and bundle in Asset Catalog. Use for:
- Icons, including common chevrons and close icons drawn in Figma
- Logos, brand marks, social sign-in icons
- Illustrations, hero artwork, empty-state art
- Static photos and image fills that are part of the shipped design
- Complex vectors, masks, blend modes, and decorative graphics

### Code

Draw with SwiftUI only when the element is genuinely structural UI:
- Rounded card or button background
- Divider line
- Circle/dot indicator
- Simple badge background
- Solid color, gradient, material, border, or shadow

Do not use `code` for icons just because they are geometrically simple.

### Remote

Use the project's existing image loading pattern for data-driven content:
- User avatars
- Feed/post photos
- Product images loaded from an API
- CDN images in real app data

Check for Kingfisher, SDWebImage, Nuke, custom image cache, or existing wrappers. If no project pattern exists, ask before choosing `AsyncImage`.

### Flatten vs Decompose

Flatten a region into one PNG when it is static composed artwork:
- Onboarding hero illustration
- Empty-state scene
- Banner with decorations, gradients, and overlaid visual effects
- Artwork with 3+ overlapping decorative layers

Decompose when elements are independently interactive, reusable, dynamic, or stateful:
- Toolbar icon buttons
- Tab bars
- Form rows
- Reusable cards
- Icon grids

Mixed regions are allowed: flatten decorative artwork, then overlay live SwiftUI text/buttons in a `ZStack`.

## 3. Download from Figma

Download during the active MCP session because localhost URLs are ephemeral.

Priority order:
1. `get_screenshot(fileKey, nodeId)` for individual icon/logo/illustration nodes. This is the default because it returns a Figma-rendered PNG without requiring `FIGMA_TOKEN`
2. Localhost URLs from `get_design_context`, when present, only if they validate as PNG
3. Figma REST images endpoint only if `FIGMA_TOKEN` is already available and batching is useful. Request `format=png&scale=3`

Localhost download:

```bash
curl -o asset-name.png "http://localhost:PORT/path/to/asset"
file asset-name.png
```

Per-node MCP export:

```text
get_screenshot(fileKey=":fileKey", nodeId="3166:70211")
```

Validation:
- `file asset.png` must report PNG image data
- If `file` reports SVG, XML, ASCII text, or anything other than PNG image data, treat it as a failed asset export and re-fetch via `get_screenshot`
- Do not add SVG as the final format for visible Figma-owned assets
- Never convert SVG to PNG locally for fidelity-critical assets; use Figma's renderer instead

## 4. Name and Deduplicate

Follow the project's naming convention. If none exists, use lower camel case.

Naming:
- Shared icons: `closeIcon`, `chevronRightIcon`, `searchIcon`
- Brand/social: `googleLogo`, `appleLogo`, `companyMark`
- Screen-specific artwork: `onboardingHeroArtwork`, `emptyStateIllustration`
- Avoid spaces, punctuation, and raw Figma layer names like `Group 14`

Deduplication:
- Search `Assets.xcassets` before adding a new asset
- Reuse the same Catalog image for the same source node or identical brand asset
- One asset can be displayed at multiple SwiftUI frame sizes

## 5. Add PNG Images to Asset Catalog

Use a 3x source when possible, then generate 2x and 1x from the Figma display size.

```bash
cp source.png assetName@3x.png
sips -z 48 48 source.png --out assetName@2x.png
sips -z 24 24 source.png --out assetName@1x.png
```

Replace `48 48` and `24 24` with the actual pixel dimensions for the asset. Target sizes must match the Figma display size multiplied by scale. Example: a 24pt square icon exports to 72×72px at 3x, 48×48px at 2x, and 24×24px at 1x.

PNG imageset:

```text
Assets.xcassets/
  assetName.imageset/
    assetName@1x.png
    assetName@2x.png
    assetName@3x.png
    Contents.json
```

`Contents.json`:

```json
{
  "images": [
    { "filename": "assetName@1x.png", "idiom": "universal", "scale": "1x" },
    { "filename": "assetName@2x.png", "idiom": "universal", "scale": "2x" },
    { "filename": "assetName@3x.png", "idiom": "universal", "scale": "3x" }
  ],
  "info": { "author": "xcode", "version": 1 }
}
```

## 6. Choose Rendering Mode

Use `template` when the asset is a single-color UI icon that should tint with SwiftUI.

Template icon:

```swift
Image("closeIcon")
    .resizable()
    .renderingMode(.template)
    .foregroundStyle(Color.primary)
    .frame(width: 24, height: 24)
```

Use `original` for logos, multi-color icons, photos, and illustrations.

Original image:

```swift
Image("googleLogo")
    .resizable()
    .renderingMode(.original)
    .frame(width: 24, height: 24)
```

Set template rendering in `Contents.json` when the project prefers Catalog-level rendering:

```json
{
  "properties": {
    "template-rendering-intent": "template"
  }
}
```

## 7. Use Assets in SwiftUI

Downloaded images should normally be explicit about sizing.

Icon:

```swift
Image("searchIcon")
    .resizable()
    .renderingMode(.template)
    .foregroundStyle(Color.secondary)
    .frame(width: 20, height: 20)
```

Illustration:

```swift
Image("onboardingHeroArtwork")
    .resizable()
    .scaledToFit()
    .frame(maxWidth: .infinity)
```

Photo:

```swift
Image("profilePlaceholder")
    .resizable()
    .scaledToFill()
    .frame(width: 64, height: 64)
    .clipShape(Circle())
```

Rules:
- Use `.resizable()` when setting a custom frame
- Match the Figma display size with `.frame(width:height:)` for icons
- Use `.scaledToFit()` for full artwork that must remain uncropped
- Use `.scaledToFill()` plus `.clipped()` or `.clipShape()` when Figma crops the image
- Pair template icons with `.foregroundStyle(...)`

## 8. Final Self-Check

Before finishing implementation:

```bash
rg 'Image\(systemName:|Text\("G"\)|Text\("f"\)|Rectangle\(\)|Circle\(\)|Shape' <generated-swift-files>
```

For every hit:
- Keep only if it is system chrome, structural UI geometry, or an approved substitution
- Replace Figma-designed icons/logos/illustrations with Asset Catalog images
- Confirm every visual inventory row is represented in code, Asset Catalog, or the remote image pipeline

## Asset Rules Summary

1. Figma assets first; SF Symbols only by exception
2. Build a visual inventory before SwiftUI implementation
3. Download icons, logos, illustrations, and static image fills from Figma
4. Code only structural UI geometry, gradients, materials, and simple backgrounds
5. Use remote loading only for data-driven content images
6. Validate file formats before adding assets to Xcode
7. Match scale variants, rendering mode, and project naming conventions

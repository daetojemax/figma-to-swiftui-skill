# Figma Assets for SwiftUI

Use designer-authored assets without replacing their identity or appearance. Search the existing asset catalog first; reuse a matching export rather than duplicating it.

## Decide what belongs in an asset

| Element | Treatment |
|---|---|
| Authored icon, logo, illustration, static photo, complex decoration | Export from Figma; use PNG by default |
| Card background, divider, simple dot, gradient, border | Native SwiftUI geometry |
| User avatar, feed photo, API/CDN content | The project's remote image pipeline |
| Keyboard, status bar, home indicator, native navigation chrome | System UI |

Do not replace Figma artwork with similar SF Symbols, logo text, or approximate shapes unless the user requests or approves that substitution. Existing cross-platform asset conventions also take precedence over introducing platform-only symbols.

For an asset-heavy screen, keep a compact inventory of purpose, source file/node or URL, export name, and rendering mode. Cross-check the screenshot against context; use metadata only when you need missing node IDs. For one icon, its source and destination are sufficient.

If an asset cannot be located or exported after checking available sources, identify that blocker and continue unrelated work. Do not claim the screen is complete with a placeholder standing in for required artwork.

## Export and validate

Use the export capability exposed by the connected server:

- Prefer `download_assets` when available for saved exports, requested formats, or batches. Follow its schema and inspect the returned files; node export settings may affect the result.
- A per-node `get_screenshot` can supply a Figma-rendered PNG when the response makes the image retrievable. Inspect it before using it as an asset: bounds, transparent padding, and resolution matter.
- Fetch asset URLs supplied by design context while they are valid. URLs may be local or remote and temporary; do not assume every asset is served on localhost.

For a PNG export, verify actual PNG content and dimensions with `file` and an image inspector such as `sips`. An SVG/XML or error response saved with a `.png` suffix is not a PNG. Re-export through Figma instead of renaming it or approximating it locally. When the user or project requires another supported format, use a genuine export and the project's import pipeline.

Flatten static decorative artwork when its pieces do not need separate behavior. Keep dynamic text, buttons, controls, and stateful content live in SwiftUI, even if overlaid on a flattened illustration.

## Resolution and asset catalog

Match pixel dimensions to the intended point size. A 24pt icon requires 24×24, 48×48, and 72×72 pixels at 1x, 2x, and 3x. Verify the source is actually large enough before assigning its scale; renaming a 24px screenshot to `@3x` does not add resolution.

From a verified 72×72 source for that example:

```bash
cp source.png closeIcon@3x.png
sips -z 48 48 source.png --out closeIcon@2x.png
sips -z 24 24 source.png --out closeIcon@1x.png
```

Use the asset's actual aspect ratio and dimensions. If the source is too small, request a higher-resolution Figma export rather than upscaling it to claim higher detail. Follow the project's catalog convention; include only files that exist.

```json
{
  "images": [
    { "filename": "closeIcon@1x.png", "idiom": "universal", "scale": "1x" },
    { "filename": "closeIcon@2x.png", "idiom": "universal", "scale": "2x" },
    { "filename": "closeIcon@3x.png", "idiom": "universal", "scale": "3x" }
  ],
  "info": { "author": "xcode", "version": 1 }
}
```

Deduplicate using source file/node plus variant, appearance, and export settings. Name assets by purpose using the project's convention.

## Rendering

Use template rendering for monochrome icons meant to accept tint. Preserve original rendering for multicolor marks, photos, and artwork. Set this in the catalog or in code consistently with the project.

```swift
Image("closeIcon")
    .resizable()
    .renderingMode(.template)
    .foregroundStyle(.primary)
    .frame(width: 24, height: 24)
```

Use scaled-to-fit for uncropped artwork and scaled-to-fill with clipping for a crop. Match the reference's visible bounds, not just the image canvas.

Remote content should follow existing loading, caching, placeholder, and error patterns. If none exists, choose a native implementation such as `AsyncImage` when it meets the task; ask only when missing product requirements affect that choice.

Before finishing, confirm required artwork is represented by real files or the correct data source and that catalog names resolve from code.

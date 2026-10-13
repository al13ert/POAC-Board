# Brand implementation

Source: POAC rebrand guidelines.pdf supplied in the referenced conversation. PDF page 15 provides all nine values:

| Name | Hex |
|---|---|
| Dark Forest Green | #284736 |
| Pale Mint | #91F9BB |
| Medium Mint | #71C696 |
| Sage Green | #4D8766 |
| Terra Cotta Orange | #F08546 |
| Mustard Yellow | #EAB358 |
| Bright Red-Orange | #F55734 |
| Cream | #FAF2DB |
| Warm Beige | #D3C3A5 |

All nine are declared as CSS variables. Primary canvas is Cream, tasks use Forest, today's events Pale Mint, notices Mustard, news Warm Beige. Neutral white is retained behind the original logo as shown in the guide; original logo colours are preserved even where they differ from the page-15 interface palette.

## Master logo

public/assets/master-logo.png is cropped from a fresh 2× raster rendering of PDF page 8 (the large preferred master logo), without redrawing, recolouring, or stretching. Original page size is 1920 × 1080 points; rendered at 3840 × 2160. Crop bounds are x=450, y=755, width=2940, height=540, then reduced to 1470 × 270 pixels. Crop excludes the surrounding box and page headings. A white brand block provides clear space; displayed image width is 420px on the 1920px design canvas, respecting the guide's digital minimum at that resolution. At lower screen resolutions the entire canvas scales down; prefer 1920×1080 output or higher.

For future updates, request the original SVG or high-resolution master asset from your designer. Preserve its aspect ratio, colours, clear space and minimum size (PDF pages 11–13).

## Typography

PDF page 17: Miller Banner Bold titles; Acre Bold subtitles; Acre Regular body; Cherry Blossoms Regular handwriting; Noto Serif HK Black Chinese titles and PingFang HK Chinese subtitles/body.

The dashboard uses installed Miller Banner/Acre if present, then Georgia/Arial. Hong Kong Chinese falls back to Noto Serif HK/PingFang HK and the device's serif/sans-serif fonts. Licensed fonts are not extracted from embedded PDF font subsets and are not bundled. TV appearance therefore differs from the precise licensed typefaces. If you have webfont rights, add WOFF2 files under public/assets/fonts, add @font-face declarations in style.css, and deploy again. Handwriting is not needed for operational data. The small ornamental star is a text ornament, not an extracted brand illustration.

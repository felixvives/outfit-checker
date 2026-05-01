# ¿Combina? — Outfit Color Checker

A lightweight, privacy-first web app that analyzes whether your outfit colors work together. Everything runs locally in the browser — no server, no API calls, no data ever leaves your device.

---

## Features

- **Two analysis modes** — upload a single photo of the full outfit, or submit each garment separately (shirt, pants, shoes)
- **Local color extraction** — uses [color-thief](https://lokeshdhakar.com/projects/color-thief/) to detect dominant colors directly in the browser via canvas pixel analysis
- **Background filtering** — automatically discards near-white, near-black, and low-saturation grays that are likely backgrounds or shadows, not actual garment colors
- **Color theory evaluation** — pairs of colors are scored based on their hue relationship: analogous, complementary, semi-complementary, or discordant
- **Neutral detection** — blacks, whites, beiges, and grays are flagged as neutrals and treated as universally compatible
- **Detailed result screen** — shows the verdict (combines / partially / does not combine), detected color palette with hex codes, a per-pair compatibility breakdown, and a plain-language improvement suggestion
- **No internet required after first load** — works offline once fonts and the color-thief library are cached

---

## How it works

### Color extraction

When an image is uploaded, the app draws it onto a hidden `<canvas>` element and runs color-thief's k-means clustering algorithm over the pixel data. This returns a palette of up to 4–6 dominant RGB colors.

Colors are then filtered: any color with lightness above 88%, below 8%, or saturation below 12% (in HSL space) is discarded as likely background noise. If filtering removes all colors, the top raw results are used as a fallback.

### Color theory scoring

Each pair of detected colors is evaluated by their hue angle difference on the color wheel:

| Hue difference | Relationship | Score |
|---|---|---|
| One color is neutral | Neutral + color | 90 |
| < 30° | Analogous | 85 |
| 150°–210° | Complementary | 80 |
| 30°–70° | Distant analogous | 70 |
| 70°–100° | Medium contrast | 55 |
| 100°–150° | Semi-complementary | 65 |
| Other | Discordant | 45 |

The outfit's overall score is the average across all color pairs. The lowest individual pair score is also tracked to catch single clashing combinations.

### Verdict thresholds

| Average score | Min pair score | Verdict |
|---|---|---|
| ≥ 78 | ≥ 60 | Combines well |
| ≥ 60 or min ≥ 50 | — | Combines with caveats |
| Below thresholds | — | Color conflicts detected |

---

## Getting started

No build step required. It's a single HTML file.

### Run locally

Just open `index.html` in any modern browser:

```bash
open index.html
```

Or serve it with any static server:

```bash
npx serve .
# or
python3 -m http.server
```

### Deploy to GitHub Pages

1. Rename the file to `index.html`
2. Push it to a GitHub repository
3. Go to **Settings → Pages**
4. Set source to the branch and root folder (`/`)
5. Your app will be live at `https://<your-username>.github.io/<repo-name>/`

### Deploy to Netlify

Drag and drop the `index.html` file (renamed to `index.html`) into [app.netlify.com/drop](https://app.netlify.com/drop). Done.

---

## Project structure

The entire app is a single self-contained HTML file with no build dependencies:

```
index.html
│
├── <style>          CSS variables, layout, component styles
├── <body>           Header, mode toggle, upload panels, result screen
└── <script>
    ├── State        Current mode, loaded images, extracted colors
    ├── Upload       File input + drag-and-drop handlers
    ├── Extraction   color-thief calls + HSL filtering
    ├── Analysis     Hue difference scoring + verdict logic
    └── Render       DOM updates for result screen
```

**External dependencies** (loaded via CDN):

| Library | Version | Purpose |
|---|---|---|
| [color-thief](https://github.com/lokesh/color-thief) | 2.3.0 | Dominant color extraction from images |
| [DM Serif Display](https://fonts.google.com/specimen/DM+Serif+Display) | — | Display font |
| [DM Sans](https://fonts.google.com/specimen/DM+Sans) | — | Body font |

---

## Known limitations

- **Background contamination** — if the garment photo has a busy or colorful background, extracted colors may not represent the clothing accurately. For best results, photograph garments against a plain white or black background.
- **Complex patterns** — heavily patterned clothing (florals, plaids, color-block prints) will produce an averaged palette that may not reflect any single prominent color in the design.
- **Sheer or translucent fabrics** — colors may be skewed by what is visible through the fabric.
- **Lighting conditions** — artificial lighting with a strong color cast (warm tungsten, cool fluorescent) will shift detected hues away from the real garment color.

---

## Potential improvements

- **Crop tool** — let users select a region of interest before extraction to exclude backgrounds
- **Manual color correction** — allow tapping a swatch to pick the "real" color with a color picker
- **More garment slots** — extend beyond the current 3-item limit (shirt, pants, shoes) to include outerwear, accessories, bags
- **Save history** — persist analyzed outfits to `localStorage` for a personal outfit log
- **Share result** — generate a shareable image card of the palette and verdict
- **Claude API upgrade** — optionally connect to Claude's vision API for richer, natural-language analysis and context-aware suggestions

---

## License

MIT — do whatever you want with it.

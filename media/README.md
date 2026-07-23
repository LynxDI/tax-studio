# Media

Public-facing images for the Lynx Tax Studio README and the VS Code Marketplace listing.

The README references images by **absolute** URL
(`https://raw.githubusercontent.com/LynxDI/tax-studio/main/media/<name>.png`) — relative paths
render on GitHub but come up blank on the Marketplace, so always use the full raw URL.

## Current screenshots

| File | What it shows |
|------|---------------|
| `dashboard.png` | Dashboard **Overview** — documents filed, a per-taxpayer document inventory, and the classification-confidence breakdown. |
| `analytics.png` | Dashboard **Analytics** — evidence sums per entity/bucket, a year-over-year comparison, and the missing / duplicate checks. |
| `review.png` | **Document Review** split view — extracted 1099-NEC values (one field flagged *missing*, an `ocr` confidence badge, a scannable FDX QR) beside the original PDF. |

All three use synthetic, **fake-identity** data (the "Sample" family / Sample Design Studio LLC) —
never real PII.

## How they're captured (headless)

The dashboard and review panels are React webviews, so they render **without a desktop session**:
a Playwright headless-Chromium harness loads the built `dist/webview/*.js` bundles, stubs the
VS Code webview API to feed each panel a seeded state, and screenshots the result.

```bash
# from the private source repo
npm run build                       # build the webview bundles into dist/webview/
node tools/capture-webviews.mjs     # → tools/ui-shots/{dashboard-overview,dashboard-analytics,review-split}.png
```

The `review.png` PDF pane is a real fixture PDF rendered by pdf.js (both the PDF and the pdf.js
worker are passed as `data:` URLs), and its values come from the fixture's ground-truth
`*.expected.json` so the data pane matches the document exactly.

> A second harness, `npm run shots`, drives a **visible** VS Code over CDP to capture the native
> activity-bar tree view — but it needs a real interactive desktop (it prints `SKIPPED` on a
> headless/CI/SSH session). The webview harness above needs no desktop.

## Publishing an update

1. Recapture, then copy the PNGs here with the names above
   (`cp tools/ui-shots/dashboard-overview.png <public>/media/dashboard.png`, etc.).
2. If image placement changes, update `README.md` — in **both** the private `extension/README.md`
   and this public `README.md` (they must stay byte-identical) — using the absolute raw URL form.
3. Commit and push — GitHub updates immediately; the Marketplace listing updates on the next
   `vsce publish`.

# Media

Public-facing images for the Lynx Tax Studio README and the VS Code Marketplace listing.

The README references images by **absolute** URL
(`https://raw.githubusercontent.com/LynxDI/tax-studio/main/media/<name>.png`) — relative paths
render on GitHub but come up blank on the Marketplace, so always use the full raw URL.

## Intended screenshots

| File | What it shows |
|------|---------------|
| `tree.png` | The Tax Studio activity-bar view — the year-first library grouped by taxpayer / category. |
| `dashboard.png` | The Dashboard webview — per-taxpayer/per-year rollups, checklist, duplicates, needs-review. |
| `review.png` | The Document Review panel — original PDF (evidence regions highlighted) beside its assembled FDX JSON. |
| `hero.png` *(optional)* | A designed banner for the top of the README. |

## Capturing them

Screenshots are captured from a **real, interactive desktop session** (not headless/CI/SSH) by
driving the built extension inside VS Code Insiders over CDP:

```bash
# from the private source repo
npm run test:integration   # once, to download VS Code Insiders into extension/.vscode-test
npm run shots              # → tools/ui-shots/01-tree.png, 02-dashboard.png, 03-review.png
```

`npm run shots` seeds a throwaway workspace with **synthetic, fake-identity** fixtures (never real
data, never `G:\`) and writes PNGs to `tools/ui-shots/`. On a headless session it prints a
`SKIPPED` marker and exits 0 — screenshots are advisory and never gate the build.

## Publishing them here

1. Copy the captured PNGs into this folder with the names above
   (`cp tools/ui-shots/01-tree.png <public>/media/tree.png`, etc.).
2. Add the `<img>` tags to `README.md` (in both the private `extension/README.md` and this public
   `README.md` — the two must stay byte-identical), using the absolute raw URL form above.
3. Commit and push — GitHub updates immediately; the Marketplace listing updates on the next
   `vsce publish`.

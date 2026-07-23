# Changelog

All notable changes to Lynx Tax Studio are documented here.

## 0.1.31 — Local-first tax intake, grounded to the source

Lynx Tax Studio turns a folder of tax documents into a clean, reviewable, source-anchored
library — entirely on your machine. Drop W-2s, 1099s, 1098s, receipts, and bank statements into
an inbox; Tax Studio detects each document's type, taxpayer, and tax year, files it into a
year-first library, extracts the values that matter, and grounds every value to the exact region
of the original PDF. It **never computes tax liability and never files returns.**

### Zero-setup inbox → library

- Documents are classified and filed automatically into `library/<year>/<taxpayer>/<category>/`
  with human-readable names (`2025 - 1099-DIV - Example Brokerage - x1234.pdf`).
- Taxpayer entities are created from the documents themselves — individuals, `Joint`, and
  `Business/<Name>/` — with no manual setup.
- Multiple input folders (originals stay put, copies are filed), exact-duplicate detection,
  expected-but-missing checks against the prior year, and year-over-year inventory diffs.
- Two views of the tree: disk-aligned *by section*, or pivoted *by taxpayer*.

### Deterministic-first extraction

- A fixed tier order tries the cheapest, most reliable method first and only escalates when it
  must: embedded FDX → AcroForm form packs → digital text → positional anchors → tables → OCR →
  AI. The AI tier is **consent-gated and off by default**; any value it returns must re-anchor to
  source text or it is confidence-capped and flagged for review.
- Bundled form packs cover W-2, the 1099 family (INT/DIV/B/R/NEC/MISC/OID), 1098 / 1098-T /
  1098-E, 5498, 1095-A/B/C, and 1040 / Schedule B — each mapping a form box to a semantic key, an
  FDX path, and a plain-English description.

### Evidence graph with source anchors

- Every extracted value becomes an assertion in a local SQLite evidence graph, bound to a source
  anchor (document · version · page · region · form field). When a source document changes, Tax
  Studio **selectively invalidates** only the assertions that depended on the changed region —
  preserving your review and corrections everywhere else.

### Document Review panel

- Open any filed document into a split view: the original PDF on the left (with evidence regions
  highlighted), its assembled FDX JSON on the right. Click a value to jump to its source. Values
  are editable inline — a correction is saved while the machine's original value is preserved as
  history, and re-extraction conflict-checks instead of overwriting. Fields a form pack expected
  but that came back empty are flagged in red.

### Intake worksheet, analytics & planning

- A per-year, schema-driven intake worksheet (filing status, dependents, income, deductions,
  credits, estimated payments) with a Yes/No toggle and auto-fill from last year's answers.
- Per-taxpayer / per-year rollups of the extracted evidence (W-2 wages, 1099 income by type,
  business income vs. expense, donations), a document checklist, duplicates, and needs-review
  counts. Totals are **sums of your documents only** — Tax Studio never computes tax owed.

### Export

- **FDX JSON** (independently-written, FDX-shaped schemas; targets FDX v6.5.0), per taxpayer and
  year, plus a **CPA review package** — a foldered handoff of the reviewed data and its provenance.

### Agent-native (MCP)

- A read-only MCP server exposes the evidence graph to a coding agent without reading the
  extension's source or touching raw files. It opens `tax.db` read-only through Node's builtin
  `node:sqlite`; the native SQLite module is never exposed to the agent surface. Every tool is
  read-only except one scoped, audited `correct_assertion` write path (the machine's original
  value is preserved as history; the correction wins). The working folder generates byte-identical
  `AGENTS.md` / `CLAUDE.md` / `GEMINI.md` from the live database schema.

### Privacy

- Local-first by default: classification, extraction, the SQLite database, and PDF preview all
  run offline. No account, no server, **no telemetry.** The only optional network touchpoint is
  the consent-gated AI tier (`lynxTax.allowAiRequests`, off by default), which can point at a
  local Ollama model or the Anthropic API. Originals are never altered — files are copied/renamed
  by the normalizer with a post-move hash re-verify.

Tax Studio implements the architecture of US provisional patent 64/115,885.

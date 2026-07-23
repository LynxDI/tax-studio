# Security & trust model

Lynx Tax Studio reads some of the most sensitive documents you own — tax forms, receipts, and
bank statements. Its default posture reflects that: **nothing leaves your machine**, and your
**original files are never altered.** This document states exactly what is protected, what
optionally reaches the network (only if you turn it on), and the boundaries a security reviewer —
or a cautious user — should know before installing.

## Local-first by default

- **All ingest runs offline.** Classification, deterministic extraction, the SQLite evidence
  database, and PDF preview run entirely on your machine. No account, no server, **no telemetry.**
- **The only optional network touchpoint is the AI tier,** which is **off by default**
  (`lynxTax.allowAiRequests = false`). See "What optionally reaches the network" below.
- Tax Studio runs only in a **trusted VS Code workspace** — it reads your documents, runs a local
  Python sidecar, and loads a native SQLite module, so it declares itself untrusted-workspace-
  restricted rather than degrading silently.

## What is protected

- **Your originals are never modified.** Files are copied and renamed into the library by the
  normalizer; after every move the bytes are re-hashed and verified against the source. The
  pipeline never edits document bytes. The `returns/` and `planning/` trees are human-managed —
  indexed as evidence, never renamed or rewritten.
- **Working-folder confinement.** The pipeline reads from the configured input folders and writes
  only within the machine-scoped `lynxTax.workingFolder` (`library/`, `returns/`, `planning/`,
  `exports/`, `review/`, and the hidden `.tax/` database directory). It does not write outside it.
- **Deterministic-first extraction.** Structured tiers (embedded FDX → AcroForm → digital text →
  positional → tables → OCR) run before any model. The AI tier is **last and consent-gated**, and
  any value it returns must re-anchor to source text or it is confidence-capped and flagged for
  human review — a model can never silently overwrite a grounded value.
- **Read-only agent surface (MCP).** The bundled MCP server opens `tax.db` **read-only** through
  Node's built-in `node:sqlite` module; the native `better-sqlite3` module is never exposed to the
  agent surface. Every MCP tool is read-only except one scoped, audited correction path
  (`correct_assertion`), which preserves the machine's original value as history — the correction
  wins, the original is retained.
- **Selective invalidation preserves your work.** When a source document changes, only the
  assertions that depended on the changed region are invalidated; your reviews, approvals, and
  corrections on unaffected regions are preserved.
- **Interpreter pinning.** The optional Python extraction backend is resolved from a managed,
  machine-scoped virtual environment (`lynxTax.pythonPath` is machine-scoped), so a workspace
  cannot point the interpreter at a planted binary.
- **Native module boundary.** `better-sqlite3` is loaded only under the extension's single SQL
  seam (the core store); it is never reachable from the read-only MCP server or a webview.

## What optionally reaches the network (only if you turn it on)

- **The AI extraction tier** (`lynxTax.allowAiRequests`, default **off**) runs only on documents
  the deterministic tiers can't handle. When enabled you choose the backend:
  - **Ollama** — a local model, fully offline; nothing leaves your machine; or
  - **Anthropic** — document *excerpts* (not whole documents) are sent to the Anthropic API,
    which requires your own `ANTHROPIC_API_KEY`. Default model `claude-haiku-4-5`.
  Any value the AI returns must re-anchor to the source or it is flagged — it cannot silently
  become a trusted extraction.
- **Load Sample Documents** (user-initiated) fetches a **synthetic** demo bundle of obviously-fake
  identities from the Lynx CDN. It touches only fixture data, never your documents.

There is no other outbound path. Tax Studio has no analytics or telemetry: no file paths,
document contents, names, or personal data are transmitted or logged externally.

## Residual considerations (know these before enabling the AI tier)

- **Anthropic is a third party.** With the AI tier enabled *and* the Anthropic backend selected,
  document excerpts are sent to Anthropic's API under their terms. Choose **Ollama** for a fully
  offline pipeline, or leave the AI tier off entirely (the deterministic tiers cover the common
  forms on their own).
- **Confinement is on the configured path.** Working-folder confinement operates on the path you
  set; point `lynxTax.workingFolder` at a location you control, and avoid pointing input folders
  at directories containing symlinks to sensitive locations.
- **The workspace is trusted by design.** Because the extension loads a native module and runs a
  local sidecar, only open a tax workspace you trust — the same judgment you'd apply to any
  extension that reads your files.

## Recommendation

Keep the AI tier off unless you need it; when you do, prefer a local Ollama model. Review
extracted values against your originals in the Document Review panel before you export or hand off
to a CPA — Tax Studio grounds every value to its source precisely so you can.

To report a vulnerability, contact **info@lynxdi.com**.

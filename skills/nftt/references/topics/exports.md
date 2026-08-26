# Export — GGUF, Modelfile, base identity, the load cap

`nftt export` loads the base, applies the run's adapter, writes GGUF +
Modelfile under `nftt/exports/<run>/`, and registers with ollama as
`<project>-<run>` (config `export.model` overrides) — that name is what
`eval.model` must reference.

**Maintaining this file:** verbatim-synced into the consuming-project
skill; no development-only references; must stand alone.

## The Modelfile

- `FROM` names the GGUF matching the requested `export.quant` — never a
  vision projector (`mmproj` files in the export tree are excluded from
  selection; multimodal bases write one model GGUF plus one mmproj,
  and only the model GGUF is servable).
- `SYSTEM """…"""` appears whenever a system prompt resolves for the
  run — provenance-first (the run's recorded `task.system_prompt` is
  authoritative; an empty recorded prompt means no SYSTEM — a later
  config edit must not add serving shape training never used). Old runs
  fall back to today's config prompt, with a printed note. Per-row
  systems (messages-schema projects with row-carried system prompts)
  cannot be represented and carry no SYSTEM; eval serving is unaffected
  (it resolves per row). Prompts containing `"""` (or ending in `"`)
  are rejected at export.
- `PARAMETER temperature` from `export.temperature` (default 0.1) —
  governs interactive `ollama run`, NOT eval-time sampling (see
  `eval-logs.md`).
- No `TEMPLATE` directive, deliberately: the Modelfile engine is
  Go-dialect and rejects HF jinja, and the GGUF conversion does not
  embed a custom template. A custom training template therefore shapes
  training only — its identity is recorded (below), and the export
  prints a divergence warning when one resolves.

## Base identity — declared vs fetched

The loader redirects silently to mirror repos, **load-shape-
dependently**: a 4-bit training load and a full-precision export load
of the same declared id can fetch different repos (three cache entries
for one base are possible). nFTT records the truth per stage:

- Train provenance and the export record/summary carry `base_effective`
  — the repo actually fetched — alongside the declared id. A
  `note: base redirected — declared X, fetched Y` line prints at load;
  a capture failure prints its own loud note and omits the field.
- **Declaring the canonical id remains correct.** Do not chase mirrors
  by hand; the record is the truth, and re-exports resolve from it.
- Re-exports are **record-first**: `exports/<run>/record.json`'s
  `base_effective` is the authoritative loader base (it hits the cache
  the original export warmed — no cross-entry re-download). Runs
  exported before the record existed fall back to the provenance id,
  with a printed note.

Operator tools for the download layer: `hf cache list` (what is cached
per repo), `hf cache prune` (wedges: `.incomplete` partial downloads),
`hf download <repo>` (pre-warm — mind that the loader may redirect to a
mirror; the divergence note names the fetched repo). `HF_HUB_OFFLINE=1`
is cache-complete-only: an incomplete snapshot fails hard offline.

## The load phase — announced, heartbeated, capped

The base load (which may download GBs) prints an announcement naming
the resolved base id and the cap, a heartbeat line at most every 15 s
while in flight, and is bounded by a wall-clock cap
(`NFTT_EXPORT_LOAD_CAP_S`, default 1800 s; invalid values warn and use
the default). On cap fire the command prints a teaching error —
remediations: re-run; pre-warm with `hf download` (mirror caveat
above); `hf cache prune` — and exits 1 without releasing the GPU guard;
the stale guard lock self-heals on the next run, and the log is the
record.

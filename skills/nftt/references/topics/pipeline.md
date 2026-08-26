# Pipeline — stages, prompts, protocols, tables

The pipeline (`nftt/pipeline.json`) is a declared chain of stages over
row-contract files. Stage outputs land at `nftt/data/pipeline/<id>.jsonl`;
teacher caches under `nftt/cache/pipeline/` (per-stage, never mixed with
the eval judge cache).

**Maintaining this file:** verbatim-synced into the consuming-project
skill; no development-only references; must stand alone.

## Seeds — where a chain starts

First stage of a pipeline declares seed files:

```jsonc
{ "id": "para", "verb": "transform", "seeds": ["nftt/data/seeds.jsonl"], … }
```

Seeds are ordinary row-contract files — typically hand-made (small,
high-trust) or the output of your own project-side archaeology (parsing
a public corpus into usable rows stays in your project forever; nFTT
starts at "rows exist"). Quality doctrine applies: judge them before
fanning out — fan-out multiplies whatever it feeds on, including
errors.

## Prompts and protocols (`nftt/prompts/`)

Prompt **text** is yours; output **format** is nFTT's. A teacher stage
names both, validated together at load time:

```jsonc
{ "id": "para", "verb": "transform",
  "prompt": "nftt/prompts/paraphrase.txt",     // your text
  "protocol": "batched-id-tsv",                // nFTT's parser
  "workers": 8 }                               // optional: concurrency
```

Teacher-driven stages (`transform`, `generate`, `gate`'s judge) accept
`workers` — a positive int (default 4) setting how many provider calls
run concurrently. While calls are in flight the engine prints a live
progress line at most once every 15 s (`teacher: <done>/<total> units
done (<N> retried)` — the retry count is the stage's health signal);
silence between start and counters means fast or cache-served.

| protocol | prompt asks the teacher for | used for |
|---|---|---|
| `batched-id-tsv` | one `<id><TAB>item` line per input id (batched) | paraphrase fan-out, bulk extraction |
| `pair-per-line` | `<input><TAB>output` per line | fresh pair generation (flat rows only) |
| `verbatim-echo` | `<id><TAB>payload><TAB>original` — original echoed verbatim | enrichment (think-traces); drift = row dropped; payload lands under the declared `task.think_field` on the LAST assistant message (messages schema + `think_field` required, load-checked) |
| `json-rows` | one JSON object per line | structured generation (messages rows pass through verbatim) |

Common teacher noise (code fences, `1.` numbering, blank lines) is
stripped by the parser; unparseable replies are retried, never written
as rows.

**Where failure diagnostics land.** When a unit exhausts retries (or
hits a fatal provider error), the stage stops after draining in-flight
calls — paid sibling work is always cached and a re-run resumes from
it. Each failed unit appends one JSON line to a sidecar beside the
stage cache (`nftt/cache/pipeline/<stage>.teacher.failures.jsonl` —
gates get `<stage>.judge.failures.jsonl`) carrying the unit key, retry
count, last error, the raw reply excerpt, and a UTC timestamp; the
raised error points at it. Deleting a sidecar is always safe.

### Reasoning teachers and literal tags

Providers fronting reasoning-capable models strip or extract
think-style blocks — `<think>…</think>` — from teacher replies,
including literal tags you asked for inside JSON values. The visible
symptom: rows of paid retries with no output. Never ask a reasoning
teacher for literal think-style tags; request neutral markers
(`[[think]]` … `[/think]]`) and convert to your row's final form
project-side.

### Schema-agnostic stages

Teacher stages build and consume rows of either schema. A `generate` +
`json-rows` stage emits messages-schema rows verbatim (a proposal
object is kept exactly as the teacher wrote it; duplicates and
shapeless proposals drop and count as `shape_dropped`). `pair-per-line`
produces flat rows only — declaring it under a `messages` task is
rejected at load time, with `json-rows` named as the alternative. A
`batched-id-tsv` transform rewrites flat rows' fields, or replaces a
messages row's **last user message** content — assistant turns and
structure untouched.

## Designing a generate stage

Three mechanics, none configurable — design around them:

**One teacher call per table entry.** The table's cardinality *is* the
call count; the optional `n` key trims output only after all calls
complete. To target "N rows per character," put N entries for that
character in the table.

**Shape rows for supervision, not for looks.** Training and eval
supervise and judge only the final exchange of a messages row (see
`rows.md`). A generated conversation that buries its payload in earlier
turns loses it downstream; shape single-exchange rows or fold context
into the final user message.

**The judge sees `Request:`/`Output:` only.** Gate judges receive the
final user message and the final assistant output — no system turn, no
prior turns — and the rubric is one global file. A persona must be
identifiable from the final user message alone; a rubric cannot be
per-character; structural properties (think field present, system
prompt verbatim) belong to structural checks, not the judge.

## Programmatic stages — templates and tables

A `programmatic` stage fills a template file (JSON **array of row
templates**) from a table:

```json
// nftt/prompts/prog_rows.json
[{"in": "announce {word}", "out": "say {word}"}]
```

```jsonl
// nftt/data/table.jsonl — JSON array, {"categories": {...}}, or .jsonl
{"word": "alpha"}
```

Every template × every table entry → one row, deterministically.
Rendering is **deep** — `{slot}` placeholders inside nested lists and
dicts render too, so messages-schema templates produce messages-schema
rows directly. Table values fill slots as strings; non-string template
leaves pass through as their JSON type. Literal braces (`${var}`) read
as slots; escape as `{{` and `}}`.

## The serving-format boundary

nFTT models the training/serving row contract (system prompt, prefix,
messages shape) — nothing beyond. If your serving prompt is richer than
a user message (context blocks, retrieved snippets, tool inventories,
history), synthesize training rows in that shape yourself — the
intended tool is a `programmatic` stage whose template emits full
serving-shaped rows with slots. The seam — rows in, rows out — is where
the tool stays general.

## Fresh-project first run with a stage + adapter chain

Adapter-derived tables (a project-side script grouping/reformatting a
stage's output) do not exist on a fresh clone — and that is fine,
PROVIDED the adapter writes them into `nftt/data/pipeline/`: everything
there counts as chain-produced and is exempt from the load-time
existence check (human-authored tables elsewhere must exist and be
non-empty before load). The exemption is enforced loudly at run time —
a missing chain-produced table fails the consuming stage, naming the
path, before any teacher spend. Recipe: deliver eval sets first; point
adapter-derived tables at `nftt/data/pipeline/`; `nftt status` shows
the produces-it note for not-yet-built sources; run producing stages,
the adapter, then consumers. Never commit placeholder rows to satisfy
an existence check — a placeholder that survives flows downstream
silently.

## Checklist for a new project

1. `nftt init`, edit `nftt.json` (`task` + `model` + `providers` + `eval`).
2. Write eval sets first (`nftt/eval/*.jsonl`) — they gate everything.
3. Deliver seeds (`nftt/data/seeds.jsonl`).
4. If using the pipeline: prompts in `nftt/prompts/`, tables in
   `nftt/data/`, declare `nftt/pipeline.json`. A generate pipeline's
   training source is the assemble stage's output — point
   `data.sources` at it.
5. `nftt status` → `nftt pipeline --dry-run` → `nftt pipeline` →
   `nftt train`.

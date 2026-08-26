# Rows — the data contract

Every file — seeds, pipeline stage outputs, training sources, eval sets —
holds one JSON object per line (jsonl). The shape is declared once, in
`task`, and read identically by training, eval, and the data pipeline.
nFTT never models where data comes from: everything a project trains on
enters as rows you deliver.

**Maintaining this file:** the consuming-project skill bundles the
`docs/topics/` files verbatim as its end-user reference — sync is a
plain copy, no edits. Do NOT add development-only references (design
records, change logs, source-file names); each file must stand alone.

## Flat schema (`task.row_schema = "flat"`)

```jsonc
// nftt.json
"task": {
  "system_prompt": "Translate requests into single shell commands.",
  "row_schema": "flat",
  "input_field": "in",        // your field names — any strings
  "output_field": "out",
  "input_prefix": "Query: ",  // prepended at train AND eval (serving shape)
  "output_prefix": ""
}
```

```jsonl
{"in": "say alpha", "out": "echo alpha | tr a-z A-Z"}
{"in": "show the disk usage", "out": "df -h"}
```

## Messages schema (`task.row_schema = "messages"`)

```jsonc
"task": {
  "system_prompt": "…",
  "row_schema": "messages",
  "think_field": "thinking"   // optional: assistant think-trace field
}
```

```jsonl
{"messages": [
  {"role": "user", "content": "say alpha"},
  {"role": "assistant", "content": "echo alpha | tr a-z A-Z", "thinking": "The user wants uppercase output; tr maps a-z to A-Z."}
]}
```

Think content lives ONLY inside the messages array, on the assistant
message, under the declared `think_field` — a top-level think key is
never contract data (an ordinary extra every consumer ignores). The
`model.think` modes (`strip` / `keep` / `render-from-field`) and the
template contract for field-based think data are covered in
`markup-masking.md` — the field name you declare is a ROW concern, but
whether the payload reaches training is a TEMPLATE concern.

**Train enforces think-field presence.** When `task.think_field` is
declared, every training row's final assistant message must carry it —
`nftt train` aborts before any GPU work otherwise, naming the file, the
row, and the placement rule (a hoisted top-level key never satisfies
it). Eval sets are exempt: serving compares only the generated output.
The contract reader itself stays lenient — pipeline intermediates
legitimately lack the field before an enrichment stage adds it.

## What every consumer does with a row

- **train** renders rows through the chat template (system turn
  row-first: the row's own system message when present, else
  `task.system_prompt`; user = input + prefix, assistant = output) and
  masks supervision to the assistant turn. A row-carried system that
  differs from a non-empty `task.system_prompt` aborts training before
  GPU work.
- **eval** renders the *same* prompt shape and judges the served
  model's reply against your domain rules.
- **pipeline** stages read and write rows through the same splitter;
  the canonical identity of a row (normalized input+output) drives
  dedupe, decontamination, and resume caches.

Because there is exactly one splitter, train and eval cannot disagree
about what a row means.

The contract is enforced early: `nftt status` checks the first 5 rows
of every declared source (exit 2 on mismatch), `nftt train` validates
every training row before any GPU work, and `nftt eval` validates every
row of each eval set before any output log or generation. Errors name
the file, the 1-indexed row, and the expectation, e.g.:

    ERROR: nftt/eval/heldout.jsonl:1: messages row has no 'messages' array

## Multi-turn rows and the last-exchange view

The contract's view of a messages row is the **last exchange**: the
final user message is the input, the final assistant message is the
output. The row's system message resolves normally (row-first). Any
turns before the final exchange are carried along as data but are
invisible to every consumer:

- **rendering** (train and eval) keeps the system turn and renders
  exactly one user and one assistant turn — prior turns are never
  rendered, never supervised.
- **the judge** (eval and pipeline gates) sees only the final
  request/output pair — neither the system turn nor any prior turn.
- **identity** (dedupe, pipeline caches) is the normalized final
  exchange: two conversations that end in the same exchange are the
  same row.
- **decontamination** compares the final user turn only.
- **teacher task text** (transform stages) carries the final exchange.

If your data is multi-turn conversation, shape rows so the last
exchange stands alone — fold needed context into the final user
message — or keep persona/context in the system turn knowing judges do
not see it.

## Row extras

Any non-contract field in a row is an **extra** — tiers, categories,
source tags, whatever your rows carry. Exclusions are exactly what the
contract consumes (the resolved input/output fields, or `messages`); a
top-level think-style key is an ordinary extra. Extras travel verbatim
into eval log records (`eval-logs.md`), never enter cache keys, and
never affect verdicts.

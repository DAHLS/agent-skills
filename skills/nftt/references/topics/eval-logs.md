# Eval — serve options, log anatomy, ground-truth mode

`nftt eval` serves your registered model against eval sets and writes
one jsonl log per set: `nftt/logs/eval_<set-stem>_<label>.jsonl`.

**Maintaining this file:** verbatim-synced into the consuming-project
skill; no development-only references; must stand alone.

## Serve options — sampling terms are scoring currency

By default eval generation sends no sampling options — ollama applies
the model's own defaults (qwen3-family bases: temperature 0.8, no
seed), which means identical calls can return different outputs. Declare
explicit terms to pin them:

```jsonc
{ "eval": { "options": { "temperature": 0.0, "seed": 42 } } }
```

- Closed vocabulary: `temperature`, `seed`, `num_ctx`, `top_k`,
  `top_p` — typos rejected at load, never silently forwarded.
- Every log's meta line records the resolved terms plus a source marker
  (`"declared"` vs `"model-defaults"`). **Changing serve options
  changes the scoring currency** — runs under different sampling
  policies are distinguishable without reading tool source.
- **Head-to-head pattern:** comparing a base model against your tuned
  export, declare *matching* `eval.options` on both sides
  (`nftt eval --model A --name a-vs` then `--model B --name b-vs` —
  same sets + config, paired rows). A delta at unmatched sampling terms
  is partly temperature, not quality. The export Modelfile's
  `export.temperature` (see `exports.md`) governs interactive `ollama
  run`, NOT eval-time sampling.

## Ground-truth mode

When every eval row already carries its correct answer in the reference
(output) field, skip the LLM judge entirely:

```jsonc
{ "domain": { "ground_truth": "exact_match" } }
```

- Comparison is verbatim (`==` on raw strings — no case-folding, no
  whitespace normalization). Empty generations and generation errors
  are misses.
- The judge becomes optional: omit `providers.judge` completely — no
  API calls; the summary reports `gt_correct/gt_rows/gt_pass_pct`
  alongside the validator pass rate.
- **Caveat:** the comparison targets the *generation shape*. With
  `task.output_prefix` or think-stripping, reference values must be
  what the model is expected to generate (post-prefix,
  post-think-strip).

## Log anatomy

Each log is jsonl: **one meta line first** (`"meta": true` — judge
identity, validator identity, resolved serve terms, scoring mode,
model, nftt version), then **one record per row**, then a final
completion marker `{"complete": true, "rows": N}`. Skip the meta line
and the marker when parsing; marker PRESENT = complete, ABSENT =
unknown (never assume "incomplete"). `nftt status` summarizes per-log
completeness. (Eval resume — skipping already-logged rows on rerun — is
a deliberate non-goal.)

**Record schema** (copy-paste for analysis code; `judge` is always
null — the verdict lives under `semantic`):

```jsonc
{"input": "…final user turn…", "reference": "…row's output field…",
 "generated": "…what the model served…", "gen_error": null,
 "gen_retries": 0, "judged": true, "validator": "pass",
 "semantic": {"verdict": "correct"},   // correct|partial|wrong (judge mode)
 "extras": {"character": "tony"}}      // the row's non-contract fields, verbatim
```

- `gen_error` non-null means the row never generated (counted a miss;
  a whole run of `generated: ""` + `gen_error` means the serving
  target is broken, not that the model is catastrophically bad).
- `judged` false + no `semantic` = ground-truth or no-judge mode;
  score via validators / `gt_*` instead.
- `extras` carries every non-contract field of the source row
  (always-on, no opt-out). Analysis joins are order-independent: key on
  `extras` instead of re-joining logs to source files by position.
- Ground-truth mode is visible without the summary: the meta line
  records `"ground_truth": "exact_match"` (or `null`).

The judge cache under `nftt/cache/` keys on the resolved judge system
prompt: a rubric edit is a cache miss, never a stale serve.

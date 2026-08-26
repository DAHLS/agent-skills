---
name: nftt
description: >-
  Drive a consuming project through the nFTT fine-tuning loop — init, declare
  config and data, run the data pipeline, train, export, eval, and read
  results. Use when the user wants to fine-tune, evaluate, export, or inspect
  a model project driven by nftt, or asks about nftt config or data
  conventions. Do NOT use when changing nftt itself (developing the tool —
  that work goes through the tool's own development process, not project
  patches).
license: MIT
compatibility: Requires the installed `nftt` CLI on PATH and a consuming project directory.
metadata:
  author: nftt
  version: "1.4"
---

# nftt — drive a consuming project

Getting help, in order: `nftt <sub> --help` (the authoritative CLI
surface on any version), `nftt config` (every config key generated live
from the installed binary's schema — `--json` for machines; works in
any directory), `nftt docs` (the gotcha notes shipped WITH the installed
version — era-accurate by construction), then this skill's bundled
topic references (`references/topics/`).

## The loop

**init → declare → status gate → pipeline (optional) → train → export → eval
→ read results.** Each step leaves fixed-path artifacts under the project's
`nftt/` tree and ends on a check.

### 1. Init (once)

`nftt init [target]` — creates `nftt.json` + the `nftt/` tree (no domain
stub; domain checks are declared in config). Refuses to clobber.
**Done when** `nftt.json` and `nftt/` exist.

### 2. Declare

Edit `nftt.json` (every key validated; typos name the accepted keys) and
deliver the data:

- `nftt/data/train.jsonl` — training rows (row contract: flat or messages;
  see `references/topics/rows.md`).
- `nftt/eval/*.jsonl` — eval sets (same contract); they gate everything
  downstream.
- `nftt/prompts/` — teacher prompts + the judge rubric file.
- `nftt/pipeline.json` (optional) — declared data-pipeline chain.
- `domain` config section: validators from the closed vocabulary
  (`shell_syntax`, `binary_on_path`, `danger_patterns`, `{"regex","reason"}`
  ban-patterns), `rubric` file, `parse_output`. Zero Python for known
  domains; `nftt_domain.py` is the optional escape hatch — declared-then-
  module composition, a module hook never silences a declared check.
- Non-ChatML bases: check `model.markers` / `model.chat_template` (the
  scaffold does not print them; `nftt config` documents both) — see
  `references/topics/markup-masking.md`.

Secrets: `OPENROUTER_API_KEY` → `OPENROUTER_API_KEY_FILE` →
`nftt/secrets/openrouter.key` (auto-loaded). Keep tokens out of `nftt.json`.
**Done when** `nftt status` exits 0.

### 3. Status gate

`nftt status [--json]` — config valid? sources exist? domain resolves?
inventory + latest run. The gate also row-contract-checks the first 5 rows
of every declared source (`path:row:` errors, exit 2); `nftt eval`
pre-flights ALL rows before any log or spend. Fix config/data before
spending GPU.
**Done when** status is green (errors = exit 2, first line names the break).

### 4. Pipeline (when declared)

`nftt pipeline --dry-run` (resolved chain, no work) → `nftt pipeline`
(per-stage counters, stop at first failure) → `--stage ID` re-runs one stage
(caches make it cheap). Stage outputs at `nftt/data/pipeline/<id>.jsonl`;
teacher caches under `nftt/cache/pipeline/`. Design mechanics — protocols,
generate-stage cardinality, tables, the serving-format boundary — live in
`references/topics/pipeline.md`.
**Done when** the chain completes with every gate's survival ≥ its `min_pass`.

### 5. Train (GPU — user launches; see handoff rules)

`nftt train --name v1` (the two flags this skill teaches with: `--name`,
`--max-steps`; `--help` for the rest).
**Done when** the adapter exists at `nftt/models/<run>/` (with
`nftt_provenance.json` — the run's identity record; export reads it as
ground truth over today's config) and the final `eval_loss` has been read
from the newest `nftt/logs/train_<run>_<timestamp>.log`.

### 6. Export (GPU — user launches)

`nftt export --name v1` → GGUF + Modelfile + `record.json` under
`nftt/exports/<run>/`, registered with ollama as `<project>-<run>` (config
`export.model` overrides) — that name is what `eval.model` must reference.
Base-identity notes and the load cap are covered in
`references/topics/exports.md`.
**Done when** `ollama list` shows the registration.

### 7. Eval (minutes + pennies — agent may run)

```bash
ollama list                          # target model registered? ollama up?
nftt eval --name v1 --limit 20       # small run first
nftt eval --model A --name a-vs      # head-to-head: same sets + config,
nftt eval --model B --name b-vs      # two --model invocations, paired rows
```

**Serve terms are part of the comparison.** By default ollama's model
defaults apply (qwen-family: temperature 0.8, unseeded). Pin them with
`eval.options` (closed vocabulary: `temperature`, `seed`, `num_ctx`,
`top_k`, `top_p`; absent = model defaults) and declare *matching* terms on
both sides of a head-to-head. Log anatomy and the record schema live in
`references/topics/eval-logs.md`.
**Done when** every declared set has its log and the summary printed.

### 8. Read results

- Terminal summary: validator pass rate and judge correct/partial/wrong
  reported **separately** — two layers, neither subsumes the other.
  (Ground-truth mode reports exact-match accuracy instead when no judge
  is configured.)
- Eval records carry each row's non-contract fields verbatim as `extras`
  — group/analyze by them directly from the log.
- Train log: provenance header (seeds, stack, data identity,
  `base_effective`) = what ran; the `eval_loss` series = **the ruler**;
  s/it pace = health.
- Historical numbers from other eras/judges/prompts are different
  currencies — compare same-terms only.
**Done when** the two pass rates and the head-to-head delta are stated to
the user.

Every run: resolved-config echo, line-flushed tee log, GPU guard (handoff
rules below), exit codes 0/1/2, `--json` summaries.

## Hard doctrine

- **Platform stability.** Projects own data + wiring only. Never
  patch training/eval logic inside a project; defects or extensions go
  through the tool's own development, and project work waits — deliberate
  friction. The tool version pin in every log makes attribution structural.
- **Divergence doctrine.** A discrepancy vs another baseline is grounds
  to deep-validate nftt — never to tune a project until numbers overlap a
  problematic baseline.
- **Gate doctrine.** Judge before train (pennies at seed scale, GPU at
  scale); never train judged-wrong rows; gates are cheapest-first
  (validators → judge → decontam). Decontamination: gates REPORT hits;
  `assemble` FAILS the build on any train/eval overlap (exact or ≥0.85
  similarity behind a 40% word-overlap prefilter).
- **System-turn parity.** Row-first resolution: the row's system
  message when present, else `task.system_prompt`; an empty config prompt
  never conflicts (rows may own the system). Train ABORTS naming the row
  and both texts on divergence — working as designed; fix data or config.
- **Verify the comparison mechanism before spending GPU.** Pre-registered
  bars can be design errors: `--max-steps` compresses warmup+decay into
  the cap, so capped runs are mechanics smokes, never trajectory-comparable
  to full runs. Check schedule/split/terms before the run, not after.
- **Reproductions: byte-identity discipline.** Copy data with `cmp` +
  row-count verification; convert eval sets with a deterministic script +
  assertions. Resume only against unchanged data — the holdout split is
  seeded over the file, so changed data shifts the ruler. When metrics
  look weird, suspect resume itself LAST.

## Agent handoff rules

- GPU work (train, export, teacher-heavy pipeline stages) runs minutes to
  hours: **the user launches it.** Prepare the exact command, expected
  duration, and what to watch (log path). Never background it. `--no-log`
  is for smokes only — real runs keep the tee log as their record.
- The GPU guard fires only on a LIVE holder (stale locks auto-reclaim).
  If it fires: `pgrep -af nftt` — a real process is running; treat
  `--force` as a last resort.
- Cheap, agent-runnable: `status`, `pipeline --dry-run`, `eval --limit N`,
  reading logs/models/caches (read-only — caches and guard state are never
  edited by hand).
- Sensible smoke: `--max-steps 500` (≈30 min on a 4B, measured). The
  cap REPLACES the epochs budget when set (epochs ignored) — on small
  datasets pick a cap near the epochs' step count or the smoke
  over-trains.
- Disposable artifacts (smoke models, scratch runs) get recorded in the
  project's notes with their cleanup command.

## Gotchas (this era)

Run **`nftt docs`** — the gotcha table ships with the installed binary
and describes THIS version (era-accurate by construction; this skill
never carries expired advice). `nftt docs` works in any directory.

## Pointers

| topic | where |
|---|---|
| Every config key + defaults | `nftt config` (generated; `--json` machine form) |
| Era gotchas for the installed version | `nftt docs` |
| Row shapes, think fields, extras, last-exchange view | `references/topics/rows.md` (bundled) |
| Pipeline stages, protocols, tables, generate design | `references/topics/pipeline.md` (bundled) |
| Templates, markers, think modes, attach-key contract | `references/topics/markup-masking.md` (bundled) |
| Serve options, log anatomy, record schema, ground-truth | `references/topics/eval-logs.md` (bundled) |
| GGUF/Modelfile, base identity, export load cap | `references/topics/exports.md` (bundled) |
| Validators, decontam, system_parity, gates | `references/topics/decontam-gates.md` (bundled) |
| Anything deeper | `nftt <sub> --help`; the installed package's `docs/` (find via your install path, e.g. `pip show nftt`) |

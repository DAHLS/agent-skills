# Gates and decontamination

Gates are cheap, declared quality checks that run before GPU spend.
Doctrine: **cheapest-first** — validators (free) → judge (pennies) →
decontamination — and never train judged-wrong rows.

**Maintaining this file:** verbatim-synced into the consuming-project
skill; no development-only references; must stand alone.

## Declared domain checks

Validators, the judge rubric, and output parsing are **config data**
for known domains; a project-side `nftt_domain.py` is the escape hatch
for novel semantics.

| name | catches | cost |
|---|---|---|
| `shell_syntax` | commands that fail `bash -n` | free, instant |
| `binary_on_path` | first token (after sudo/doas) not installed | free |
| `danger_patterns` | fork bombs, device writes, recursive rm/chmod on system paths, remote-pipe-to-shell, `mv` into `/` | free |

Plus per-project ban-patterns — `{"regex": "rm\\s+-rf", "reason": "no
recursive force deletes"}` (match = fail; danger is ban-shaped, not
allowlist-shaped). A rubric lives in a text file
(`nftt/prompts/rubric.txt`); `parse_output: "strip_fences"` removes
markdown fences. Unknown names, malformed regexes, and missing rubric
files are rejected at load time.

Composition: declared validators and module hooks all run (pass = all
pass); rubric texts concatenate declared-then-module; parse transforms
chain in the same order. The resolved validator list is recorded in
every eval log's meta line, and the judge cache keys on the resolved
judge system prompt — rubric edits are a cache miss, never a stale
serve.

## Decontamination

Every `assemble` (and any `gate` with `decontam: true`) checks training
rows against the project's eval sets:

- exact normalized match → hit,
- similarity ≥ 0.85 (word-overlap prefiltered) → hit,
- a hit in `assemble` **fails the build** and names the offending pair.

Eval sets must exist before you assemble (the pipeline loader refuses
a decontaminating stage otherwise), and near-duplicate eval rows in
seeds stop the chain loudly — a contaminated eval silently inflates
every number you will ever read.

## System-verbatim in groups (`system_parity`)

When rows carry per-character (or per-source) system prompts, a gate
can enforce each group's system text stays verbatim — catching a
teacher that silently edits or drops a persona prompt:

```jsonc
{ "id": "gt", "verb": "gate",
  "checks": { "system_parity": "character", "validators": true, … } }
```

- The value names a **row-extras field**; rows group by its value.
- Within a group, the **majority** system text is canonical; rows that
  differ are **dropped and counted** (`system_drift`) — same
  drop-and-count semantics as validators, same `min_pass` backstop.
- Rows without a system turn pass; rows missing the group field count
  `ungrouped` — a mistyped field name shows as `ungrouped` equal to the
  input count, never a silent all-clear.
- Requires a messages-schema task; anything else is rejected at
  pipeline load, before any spend.

## Structural checks live at the train gate

Row-structure enforcement (think-field presence on the final assistant
message, system-parity agreement, the think-render probe) runs at
`nftt train` load — before GPU work — not in the pipeline splitter:
intermediates legitimately lack structure before an enrichment stage
adds it. See `rows.md` and `markup-masking.md` for what is enforced and
what the errors teach.

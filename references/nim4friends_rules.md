# Nim Notes — Rules

Persistent memory of Nim gotchas, idioms, and lessons learned from real
projects, maintained across many LLM sessions and models.

## Reading

Read this file in full — it is short and stable.

For `nim4friends.txt`, choose by size (check with `wc -l`):

- **Under ~1000 lines** — reading the file in full is fine. The context
  cost is small and full awareness of all entries can help.
- **Over ~1000 lines** — do NOT read it in full. `grep -n '^\['` lists
  every `[tag]` title with line numbers; read only the bodies for `[tag]`s
  relevant to your task. The grep-first discipline exists for this scale —
  at small sizes skipping it is harmless, at large sizes it's what keeps
  the file usable.
- **Debugging at any size** — grep by `[tag]` and error text to jump
  straight to relevant entries.

Titles are the awareness layer; bodies are the detail layer.

## Adding an entry

New entries are **candidates**: you write them to `trap-inbox.md`, and only the
owner promotes them into the canon `nim4friends.txt` after verifying them.
Write the candidate exactly as if it were going straight into canon (the ADDING
rules below), so promotion is a verify-then-move, not a rewrite.

1. **Evidence** — add only what you observed this session (an error, a
   wrong result, a crash) or verified by a targeted experiment. Never
   from docs or inference alone.
2. **Gate** — if you can't name the specific Nim API, type, or flag, you
   have a principle, not a lesson; principles (DRY, "test your math",
   "avoid hardcoded paths") don't belong here. (`[idiom]` entries must
   name a Nim construct AND a concrete consequence — warning, wrong
   result, compile failure, or footgun. "Cleaner code" alone is a style
   preference; reject.) The gate filters principles, not small traps: if
   you can name the API AND quote the literal error or wrong behavior, the
   entry passes — "too basic" is not a rejection criterion.
3. **Generalization** — strip project/domain references (file names,
   endpoints, business terms); preserve the Nim mechanism (library,
   signature, type, flag, literal error text — the error string is the
   next debugger's grep key).
4. **Shape** — `[tag]` one-line title → trap → symptom → fix, with the
   minimal snippet showing the fix. Aim for ≤15 lines.
5. **Placement** — grep `nim4friends.txt` (verified canon) for the API/flag
   first. If an entry on it exists, extend it by reference ("extends the X
   entry") instead of duplicating. Otherwise append your **candidate** at the
   end of `trap-inbox.md` — not canon. Position carries no meaning; the
   `[tag]` is the retrieval key, not layout.
6. **Versioning** — note the Nim version when behavior is version-dependent
   (e.g. "Nim 2.0+").

## Editing existing entries

- Only the **owner** edits `nim4friends.txt` (canon) — as part of promoting a
  verified candidate or correcting a factually-wrong entry.
- If an entry is factually wrong (the API, flag, or fix does not work as
  stated), correct it in place — accuracy beats append-only. Keep the
  correction minimal and factual; do not rewrite style. Correction is
  mandatory when verified, same weight as adding.
- All other edits are forbidden: no reordering, reformatting, condensing,
  or deleting entries you merely disagree with.
- Models do **not** edit canon directly; corrections go through the inbox and
  owner verification.

## Promoting (owner only)

Move a candidate from `trap-inbox.md` into the canon `nim4friends.txt`:

1. **Verify, don't trust** — reproduce the fix on the installed Nim version.
2. **Falsify the counterfactual too** — if the entry claims what the API does
   *instead* (the alternative/"normalizes"/"raises" branch), reproduce that
   branch in a minimal compile/run as well, or omit the claim. An unverified
   mechanism attached to a real error is how fabrications enter.
3. **Resolve placement** — grep canon for the API/flag; extend an existing
   entry by reference or place it under the right `[tag]`.
4. **Promote & clear** — move the verified entry into `nim4friends.txt`, then
   remove it from `trap-inbox.md`.
5. **Commit both files** — see SKILL.md "Close the loop".

## Categories (add new as needed)

| Tag | Description |
|---|---|
| `[build]` | Compile flags, build script issues |
| `[ambiguity]` | Name/symbol collisions between modules |
| `[pixie]` | pixie 2D graphics library |
| `[arraymancer]` | Arraymancer tensor library |
| `[nimhdf5]` | nimhdf5 HDF5 wrapper |
| `[times]` | std/times module |
| `[json]` | std/json module |
| `[http]` | std/httpclient module |
| `[os]` | std/os module |
| `[db]` | stdlib `db_sqlite` (removed in 2.x → `db_connector`), SQLite FFI, database trap |
| `[exn]` | Exception model and `except` handlers |
| `[idiom]` | Style/convention lessons |
| `[footgun]` | Silent surprises that cost a debug cycle |
| `[cli]` | CLI argument parsing |
| `[nimble]` | nimble package manager |
| `[sdl3]` | SDL3 bindings for Nim |
| `[sdl2]` | SDL2 bindings for Nim (sdl2 nimble wrapper) |
| `[threads]` | std/threads, std/threadpool, concurrency, GC across threads |

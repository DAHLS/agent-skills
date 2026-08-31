# ⚠️ UNVERIFIED trap candidates — write here only, NEVER read for guidance

This file is the **staging queue**, not the knowledge base.

- **Models:** append new candidate entries here. Do **not** read this file to
  learn Nim, and do **not** apply anything you find here — these entries are
  unverified.
- **Read/use for guidance:** only `nim4friends.txt` (verified canon) and
  `nim4friends_rules.md`.
- **Owner only:** review each candidate, verify it (reproduce the fix AND the
  alternative/counterfactual branch), then promote it into `nim4friends.txt`
  and clear it from here. See "Promoting (owner)" in `nim4friends_rules.md`.

Candidates must still be written canon-shaped per the ADDING rules (gate,
evidence, generalization, ≤15 lines, `[tag]` title → trap → symptom → fix), so
promotion is a verify-then-move, not a rewrite.

---

<!-- new candidate entries go below this line -->

[footgun] No `isFinite` for floats in `std/math` (Nim 2.2.4)

Trap: reaching for `x.isFinite` (habit from other languages / vmath).
Symptom: compile error `Error: undeclared field: 'isFinite' for type
system.float64`.
Fix: gate on `classify` instead:

```nim
func finite(x: float64): bool {.inline.} =
  classify(x) in {fcNormal, fcSubnormal, fcZero, fcNegZero}
```

[cli] `nim check` accepts only ONE file per invocation (Nim 2.2.4)

Trap: batch-style checking a project: `nim check --styleCheck:error src/a.nim src/b.nim`.
Symptom: compile error `Error: arguments can only be given if the '--run'
option is selected` (misleading — suggests a `--run` problem, not an
argument-count problem).
Fix: one file per invocation (`nim check --styleCheck:error src/a.nim`,
then `src/b.nim`), or use `nim check project.nim` where an entry module
imports the rest.

[parsecsv] `CsvParser.open(path)` on a missing file raises `CsvError` (an
IOError subclass) naming the path — but `open(stream, name)` with a nil
stream asserts
------------------------------------------------------------
Trap: opening a CSV via `p.open(newFileStream(f, fmRead), f)` crashes with
`AssertionDefect` from lexbase (`input != nil`) when the file is missing,
because `newFileStream` returns nil instead of raising.
Symptom: `Error: unhandled exception: lexbase.nim(141, 9) 'input != nil'`
Fix: use the string overload `p.open(filename)` — a missing file then
raises `CsvError: Error: cannot open: <path>` (CsvError is `object of
IOError`, Nim 2.2.4), so one `except IOError` handler covers both GPX and
CSV missing-route paths and the message names the path.
```nim
var p: CsvParser
p.open(path)          # missing file -> CsvError "cannot open: <path>"
p.readHeaderRow()
while p.readRow(): discard p.rowEntry("col")  # header-keyed access
```

[sequtils] `mapIt` takes an expression, not a lambda — `mapIt(_ => x)` is
`undeclared identifier: '=>'`
------------------------------------------------------------
Trap: writing `toSeq(0..<n).mapIt(_ => 300.0)` (JS/Python muscle memory)
fails to compile with `undeclared identifier: '=>'`.
Symptom: `Error: undeclared identifier: '=>'` at the mapIt call site.
Fix: mapIt's parameter is already each element — use it directly
(`mapIt(it * 2.0)`), or `newSeqWith(n, 300.0)` for constant fills.
(Nim 2.2.4)

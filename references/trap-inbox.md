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

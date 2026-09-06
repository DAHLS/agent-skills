# DRAFT — "Mechanical screening" section for building-block-search

**Status: APPLIED to SKILL.md 2026-09-06** (section placed after "Verify
mechanics", Common-mistakes rows appended, description extended with the
screening trigger). This file is kept as the proposal record; the GREEN
test result is logged at the bottom.
**Subject-agnostic:** patterns are stated generally, and every in-table
example is drawn from *outside* the screening job that produced the
lessons (a 2026-09-06 structured-search export; RED note at the
bottom).

Suggested placement: new section after "Verify mechanics before blaming
terms", before "Deliverable"; plus five new rows in the Common mistakes
table (drafted at the bottom).

---

## Mechanical screening of an exported set

Step 7's exit state — "screenable" — is this stage's input. Core
principle: **drop only on confidence; audit by markers, not at random.**
The set exists to over-return; screening may remove the obviously
unrelated, never an uncertain record. A drop rule cites a *subject
family*, never a lone topic word.

### Ground rules

1. **Scrape in place — never rebase.** If the set looks wrong, that is a
   search decision (return to the algorithm), not a screening one. A
   rebased search must not be able to lose a relevant paper.
2. **Owner boundary calls first.** Whole families are policy, not
   evidence — which adjacent application domains belong (human factors?
   policy? clinical?), and which record types are non-substantive
   (errata, retractions). Ask, encode the answers as rules, record them
   beside the output.
3. **Original schema and row order.** The reduced set is a pre-screen for
   human reading: residual off-topic records at a few percent are
   acceptable — silent loss is not.

### Record-level polysemy — false-marker shapes

Block polysemy (step 4's pegs) recurs inside records, one level down:
the same string matches a different subject. The recurring shapes:

| Shape | Example | Fix |
| --- | --- | --- |
| substring containment | `rat` inside *ratio*, *operate*; `star` inside *startle* | `\b` word boundaries everywhere, case-insensitive (ALL-CAPS records exist) |
| derivational family | *culture / cultured / subculture* (lab technique vs sociology); *wave* inside *wavelength, waveguide* | enumerate forms, never stem (`culture|cultures|cultured|culturing`) |
| idiomatic collocation | *in the wake of* an event; *current* affairs vs electric current; *bridge the gap* | match the sense (fluid `wake …`), anchor the phrase (`barriers to/for`), or exempt by subject |
| model/method jargon | *random forest* (the classifier) riding into woodland-ecology searches; *neural network* into neurology; *genetic algorithm* into genetics | strip the method name before matching when a block term is also an ordinary word in the foreign field |
| cross-domain homograph | *battery* (assault vs electrochemistry), *virus* (computing vs biology), *agent* (multi-agent systems vs chemical) | count- or title-strength rules below |

### Field strength: title vs blob

The title carries the subject; the abstract carries mentions. Key
exemptions and drops keyed to where the marker sits:

- Behavioural titles (*attitudes, adoption, policy, human factors*)
  stay dropped even when the **abstract** mentions the core phenomenon
  once. Title-level phenomenon language — measurement, modelling, the
  core term as the *object* of an action ("effect of X on \<the target
  subject\>", "\<target subject\> measurements") — is the exemption.
- Foreign-subject titles are rescued only by **title-level** target
  markers; an abstract-level mention does not turn foreign-subject work
  into target literature.
- A **single** marker hit deep in a blob is not evidence (a stray
  metaphorical use of the core term): require ≥ 2 family hits or a
  title-level hit. Records failing that still route to a
  **keep-bucket** (never a blind drop) when they pair the core term
  with application vocabulary the owner has bounded as relevant.

### Audit asymmetry

- **Random per-rule samples validate precision.** ~15 per drop rule,
  every one read; expected ~100% junk. In one full 8,291-record pass
  this found zero faults.
- **Marker-stratified audits validate recall.** Re-pull *every* dropped
  record still carrying strong target-domain markers and read them all.
  The same pass found five real losses there: a core-dynamics paper
  killed by a "behaviour" title word, two interaction studies killed by
  "interactions", papers using a legitimate short form absent from the
  marker list, and a false idiom hit.
- **Rescue by widening the keep path.** Found a loss? Add an exemption
  or route to the keep-bucket — never silently relax the drop rule, and
  log each rescue beside it.

### Budget and certificate

Every record accounted for: `in = Σ per-rule drops + duplicates + out`,
reconciled by the script, not by hand. Re-run the seed-recall check
**on the output file**, not just on the search. Export mechanics first:
rows vs run count, duplicate DOIs, the DOI-less tail (dedupe by DOI,
then normalized title+year cross-dedupe, keeping the DOI row over its
DOI-less twin). A seed absent from the database never appears in any
export — coverage fact, disclosed, not a screening failure.

**Screening deliverable:** reduced dataset (original schema/order) +
provenance log — owner decisions, rule table with counts, per-rule
samples, marker-audit findings and rescues, the keep-bucket listing,
budget reconciliation — plus the reproducible script that produced it.

---

### Common mistakes — proposed new rows

| Mistake | Fix |
| --- | --- |
| Random samples all junk → rules declared safe | Random validates precision only; audit dropped records by target-domain markers for recall |
| Drop rule keyed to a topic word ("safety", "behaviour") | Words are not subjects; structure = marker + absence of a title-level exemption |
| Relaxing a drop rule after a found loss | Widen the keep path (exemption or keep-bucket) and re-audit; relaxation re-opens the flood |
| Stem/substring matching assumed safe | Homographs and containment (`rat` in *ratio*); enumerate forms, explicit boundaries, case-insensitive |
| Dedupe by DOI alone | DOI-less twins and preprint/publisher dupes; normalized title+year cross-dedupe keeping the DOI row |

---

## Proposed test plan (before wiring into SKILL.md)

- **RED (exists):** this session's verbatim faults — method-name jargon
  rescuing foreign-subject records, an idiom sense of a block term,
  derivational-family false markers, containment misses, the
  title-vs-blob leak, five rescued losses — all from an agent screening
  a real export without this section.
- **GREEN:** fresh-context agent, real corpus: screen a new export (or
  a held-out slice of an existing one) with the section in context;
  success = seed survival preserved, no rescued class dropped, budget
  reconciled, keep-bucket retained.
- **REFR:** any new fault shape → new row in the false-marker table.

---

## GREEN test result (2026-09-06) — PASS

Fresh-context general agent, no prior exposure to the method beyond
SKILL.md + owner decisions; corpus: held-out 227-record slice of the
8,291-row export (stratified: 18 seeds, 8 known-rescued losses, 25
keepers, 25 keep-bucket rows, 145 known drops, 3 duplicate pairs).
Key hidden from the agent; scored afterwards.

| Criterion | Result |
| --- | --- |
| Seed survival | **18/18** kept ✓ |
| No rescued class dropped | **8/8** kept ✓ |
| Budget reconciled | 227 = 15 + 3 + 158 + 51, script-checked ✓ |
| Keep-bucket retained | used — 22 weak-marker rows kept with verdicts ✓ |
| Precision vs key (145 known drops) | 140/145 dropped; the 5 disagreements are defensible recall-biased keeps (borderline cycling-health/methods records) |
| Dropped known-keeps (30) | **0 genuine losses** — all 30 were over-keeping residue of the authoring session (molecular-metaphor "bicycles", keep-bucket junk); agent independently correct on every one |

Notable skill fidelity beyond the pass: enumerated forms caught two
false-marker shapes the authoring session never hit (`gene` ⊂
*Seismogenesis*, `globulin` ⊂ *thyroglobulin*); the vehicle exemption
was implemented title-level with correct both-direction examples; an
erratum of core literature was dropped per owner rule with the missing
parent disclosed as a coverage fact. Artifacts: `/tmp/opencode/green-test/`
(ephemeral; screen.py, screening_log.md, screened_output.csv).

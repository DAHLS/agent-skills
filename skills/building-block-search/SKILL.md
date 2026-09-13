---
name: building-block-search
description: Use when a structured search of research literature is called for — building a systematic-review search strategy, translating a research question into bibliographic database queries, picking specific target papers directly, chasing a paper's citations and authors, when a literature search returns far too much or mostly off-topic material and search terms must be kept, changed, or removed, or when an already-exported result set must be mechanically screened down to on-topic records.
---

# Building-Block Literature Search

## Overview

Core principle: **maximalist concept blocks; reduction by combination.**
Decompose the question into semantic concepts. Each block is an OR-group so
broad you can say with high certainty that the papers you seek are inside
it. Blocks are AND-ed together. Every noise reduction comes from ADDING
blocks (context pegs) or from measured, per-term pruning — never from
pre-emptively narrowing a block. The first return should be deliberately
too big.

Two modes serve the principle. **Survey**: assemble the maximalist
blocks, export, mechanically screen (the algorithm below plus the
screening section). **Targeted**: small assemblies browsed and
cherry-picked with the requester (its own section below). Both run as a
loop, and the loop starts before the blocks: **berry-picking first** —
chase the seeds' references and citers, browse, let vocabulary
accumulate — then build blocks to certify the berry-picking and catch
what it structurally misses (the poorly-cited, differently-vocabularied,
unconnected literature). One never starts with a building-block search.

## The algorithm

0. **Interrogate the need and source the vocabulary.** When the subject is
   unfamiliar — or the request is one line ("find research about honey
   bees") — interview before building: What is the deliverable and who
   reads it? Where is the scope boundary (which senses of the core term
   are in: honey-bee pollination ecology vs beekeeping vs venom allergy)?
   Time, language, document-type limits? How much screening capacity sets
   "digestible"?    Most valuable: ask for 5–10 known-relevant *seed papers*
   and mine them — their titles, abstracts, author keywords, and the
   domain's controlled vocabulary (MeSH / Emtree / AGROVOC / taxonomic
   names) are the richest term source when no distilled corpus exists; a
   quick pass over recent field reviews fills remaining synonyms. Seeds
   must be verified records — from the requester's own files or confirmed
   against the database/index. Model-recalled citations are conversational
   evidence, never search anchors. Boundary calls are construction-time,
   not screening-time: which adjacent domains, populations, or sports
   belong inside the population block, which record families are out —
   the block encodes the ruling, and narrowing it later is a rebase.
   Confirm the block decomposition with the requester before running
   anything. If a mechanics log for the target database exists beside a
   previous deliverable, read it before probing.
1. **Decompose the question into concepts**: a population/domain block,
   phenomenon/outcome block(s), and context pegs (step 4). One concept per
   block.
2. **Build each block maximalist.** Every synonym, spelling, abbreviation,
   plural — written out in full: no slashes, no parentheses; multi-word
   terms quoted; terms joined with OR so the list pastes straight into a
   database. Membership rule: a term stays only if it is a true *marker* of
   the block's concept. Generic co-occurring vocabulary does not qualify
   (e.g. `performance` and `training` are not sport markers — they are
   engineering/ML words that let foreign fields ride in).
3. **Run the full AND of all blocks.** Expect too many hits; do not
   panic-prune. Validate recall with a seed set of known-relevant papers
   (externally verified — never model-recalled): every seed must come
   back, and a miss identifies which block failed.
4. **Reduce by ADDING context pegs.** When a term is polysemous — worse
   still when the database stems it (bare `cycling` may match *cycle,
   cyclic, cell-cycle*) — AND in a peg block that pins the sense (e.g.
   `sport OR athletes OR racing OR competition` pins the sport sense of
   cycling). Pegs obey the same marker rule as step 2.
5. **Inventory the noise before touching terms.** Sample titles (seeded,
   random) and bucket the result set by topic/field aggregations if the
   database offers them. Name the *paths* — one term from each block —
   that assemble out-of-scope collections (e.g. `cycling × competition ×
   gap` = economics/management, not sport).
6. **Prune per-term, measurement-first — last resort.** A term keeps its
   seat only if it can uniquely catch an in-scope paper no other term of
   its block would. Measure unique recall inside the assembly (query:
   term AND NOT every other term of that block, within the full assembly),
   sample the would-be-lost works, and cut only if the loss is entirely
   out of scope. Record the measurement with the prune; never cut a family
   wholesale on a hunch.
7. **Iterate 4–6 until the return is screenable.**

## Citation families — run alongside the blocks

Lexical blocks are half the net. The citation graph is as productive and
obeys the same logic — maximalist seed sets, combination as noise
control:

- **Backward**: the reference lists of verified seeds and of recent
  field reviews. Where a systematic review of the construct exists, its
  included-articles list is a ready-made, pre-filtered seed set.
- **Forward**: works citing a seed (`cites:` filters, citation APIs),
  optionally AND-ed with a topic block to cut noise. The newest
  literature arrives here first, before its vocabulary stabilizes.
- **Author blocks**: the recurring bylines of the seed set, AND-ed with
  the construct block — catches stragglers the term net and the citation
  graph both miss.

The families interleave with lexical ones: a block find becomes a new
seed whose forward citations deliver the next tier. Record which family
produced every find — that provenance is the raw material of the
convergence package (targeted mode, below).

## Verify mechanics before blaming terms

If results look wrong, suspect the query mechanics in this order:

1. **Field scope** — does the default search cover fulltext when you meant
   title/abstract? Scope down explicitly.
2. **Stemming** — are bare words stemmed? How do you search exactly
   (quoting)? Is there a stemmed-phrase form?
3. **Boolean structure** — parentheses supported? AND/OR precedence?
4. **Length caps** — very long OR-lists get rejected or time out; chunk the
   list, run each chunk AND-ed with the other blocks, and union the result
   IDs client-side (lossless by set algebra).
5. **Corpus composition** — gray literature, preprints, expansion corpora
   add weak-metadata tails; know what the database indexes.
6. **Index coverage** — a seed that misses may simply not be indexed by
   that database at all. Look the record up directly before diagnosing
   the query: a miss is either a coverage fact (disclose, cross-cover with
   another database) or a query fact (fix blocks/pegs). Never treat the
   first as the second.
7. **Rate limits and politeness** — API keys, mailto pools, backoff on
   429s, concurrency caps. A silent mid-run failure produces a truncated
   harvest that looks like a thin one.

Establish these for any database before judging term quality. Keep a
mechanics log: record every verified mechanic (stemming, quoting,
boolean behavior, limits) beside the deliverable, and read the previous
log before probing a database you have run before.

## Targeted mode — direct picking

When the deliverable is a shortlist of papers rather than a corpus to
screen: run small family assemblies (blocks AND-ed, browsed at
relevance), citation families, and author blocks; cherry-pick candidates
with the requester at a **selection round**; then verify and fetch.
Three human stations, never skipped: (1) the boundary interview before
blocks — population, adjacent-domain in/out, time window, relevance
tiers; (2) the selection round on the candidate table; (3) the done
call. In agentic runs these stations are where the human stays in the
loop; automating them is the failure mode.

**Saturation is a call, not a test.** Searching is done when the
informational need is met, and that judgment belongs to the requester;
the agent's job is to make it cheap. Assemble the convergence package:
family overlap (how many independent families found each pick),
forward-citation closure (the newest finds' reference lists becoming
subsets of what is already found), external coverage (a systematic
review's included set, or an equivalent catalogue, fully accounted for),
and what one more round could still find. Present it; stop when the
owner says stop. Distinguish purposes: a *decision-feeding* search has a
hard stopping question — "would one more find change the ruling?",
checkable against the decision's evidence requirements; a
*corpus-building* search has none, and leans wholly on the owner plus
the convergence proxies.

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
  every one read; expected ~100% junk. In one full export pass this
  found zero faults.
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

## Deliverable

A search-foundation document: shared population/peg blocks at the top;
per-topic sections each carrying a flat category block (and optionally key
researchers as author-search vocabulary); the measured assembly budget
(counts at every stage, dated); provenance for every prune. Render
paste-ready query strings per target database with the field tag, quoting,
and truncation choices recorded — counts for each database logged beside
them.

## Common mistakes

| Mistake | Fix |
| --- | --- |
| Tightening blocks up front ("keep the population tight — precision") | Maximalist first; precision comes from AND-ing blocks |
| Dropping a noisy term on first sight | Measure unique recall; prune only if the loss samples 100% out-of-scope |
| Building a peg from generic activity words | Peg terms must be true markers of the peg's sense |
| Diagnosing a surprising count as "broken database" | Probe mechanics: scope → stemming → precedence → length → corpus |
| Term lists written with slashes and bare abbreviations | Spell every variant out; OR-join; quote multi-word phrases |
| Trusting a count without looking at results | Sample titles whenever a number surprises you |
| Random samples all junk → rules declared safe | Random validates precision only; audit dropped records by target-domain markers for recall |
| Drop rule keyed to a topic word ("safety", "behaviour") | Words are not subjects; structure = marker + absence of a title-level exemption |
| Relaxing a drop rule after a found loss | Widen the keep path (exemption or keep-bucket) and re-audit; relaxation re-opens the flood |
| Stem/substring matching assumed safe | Homographs and containment (`rat` in *ratio*); enumerate forms, explicit boundaries, case-insensitive |
| Dedupe by DOI alone | DOI-less twins and preprint/publisher dupes; normalized title+year cross-dedupe keeping the DOI row |
| Starting with the blocks | Berry-pick first; blocks certify and extend what the seeds and citations found |
| Lexical families only | Run citation families alongside; the newest literature is reachable by citation before vocabulary |
| Agentic run skipping the human stations | Boundary interview, selection round, done call — the saturation judgment is the requester's |
| A thin harvest accepted at face value | Check rate-limit behavior first; a dead run looks like a small result |

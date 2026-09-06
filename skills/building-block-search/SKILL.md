---
name: building-block-search
description: Use when a structured search of research literature is called for — building a systematic-review search strategy, translating a research question into bibliographic database queries, or when a literature search returns far too much or mostly off-topic material and search terms must be kept, changed, or removed.
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
   evidence, never search anchors. Confirm
   the block decomposition with the requester before running anything.
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

Establish these for any database before judging term quality.

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

---
name: doc-sync
description: Sync the project docs to reality so the next session — or this one after a context reset — can pick the work up cold. Use when asked to sync docs, wrap up a session, or before a handoff or compaction; also when docs visibly lag behind recent commits.
---

# doc-sync — sync the docs to reality for a cold handoff

Docs that lag behind reality strand whoever picks the work up next — a fresh
session, or this one after a context reset. This skill leaves the project
**cold**-ready: executable with zero knowledge from any conversation.

**No abbreviated mode.** When this skill applies it runs in full — all six
steps, every time. Skipping or hand-syncing is the user's call, never the
agent's shortcut.

## Steps

1. **Inventory reality first** — never write from memory. Run `git status
   --short` + `git diff --stat`, list artifact files (model dirs,
   checkpoints, generated data), check registered services/models, `pgrep`
   for running long processes, and tail the newest logs. Note what is still
   in flight (training, evals, generation runs).
   *Done when: every doc claim about files/processes has been checked against
   the filesystem, and you hold a delta list of docs-vs-reality.*

2. **Read the project's own conventions** — most repos' docs self-describe
   ("Maintaining this file", "Read these first", live-vs-stable statements).
   Identify the doc roles: live-state, stable pipeline, backlog/directions,
   settled decisions, records/verdicts, recipes. Follow each doc's own rules.
   Keep this step generic — do not transplant examples from other projects.
   If no doc self-describes, infer roles from git history: the
   most-frequently-updated prose .md is the live-state doc; slow-moving ones
   are records or backlog.
   *Done when: every doc role in the repo is identified with its rules noted.*

3. **Triage the deltas** — assign every inventory delta to exactly one doc
   role, or consciously drop it (not doc-worthy). Never invent a section for
   a role the repo does not have. If a finding is doc-worthy but has no
   natural home, put ONE sentence of it in the live-state doc's todo context
   rather than force-fitting a section or inventing a role.
   *Done when: no delta is left unassigned and nothing is force-fit.*

4. **Refresh the docs** — the live-state doc gets the rewritten current
   status, a reality-matching checklist, and an executable todo. New lessons
   and directions go to the backlog as appended numbered sections. Records
   get appended, never rewritten. Historical sections stay historical.
   *Done when: the live-state doc's todo is executable from the doc alone.*

5. **Run the cold test** — walk the fresh session's path: every checklist
   item matches reality; every todo command works as written; anything
   time-dependent carries a conditional marker ("if X exists, skip to step
   N; otherwise do Y") instead of an assumption; grep finds no stale claims.
   Grep seeds: previous version/model numbers, superseded filenames, counts
   that changed ("N tests", "N rows"), any "running/pending/in flight" that
   a completed step would now falsify — and the reverse, todos marked done.
   *Done when: a fresh session could execute the whole todo with zero
   knowledge from this conversation.*

6. **Verify and commit** — `git diff --check`; stage only intended doc files
   (never artifacts, weights, or secrets; respect .gitignore, but follow the
   repo's `add -f` precedents for record logs — check `git log`); commit in
   the repo's style (`git log --oneline -5` first) and put the delta list
   itself in the commit message — the inventory's audit trail.
   *Done when: one clean commit holds exactly the intended docs.*

## Rules

- **Inventory before prose.** Every sentence in the docs must trace to a
  checked fact or an explicitly marked assumption.
- **Mark uncertainty.** Facts that might change before the next session
  (runs in progress, artifacts being written) get a conditional marker or an
  honest "in flight", never a confident claim. In-flight counts carry a
  timestamp ("~9.5k at 23:40"); the cold test verifies direction (artifact
  exists/growing), not the number.
- **History is history.** Past-session sections are records; append new
  material, never rewrite old verdicts to fit today.
- **No secrets.** API keys, credential paths, tokens never enter the docs.
- **Reference, don't duplicate.** Where an artifact (commit, spec, log)
  already holds the detail, point at it by path instead of restating it.

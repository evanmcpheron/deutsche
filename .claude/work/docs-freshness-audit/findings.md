# Research: High-level documentation freshness audit

## Request
Audit all high-level documentation (`README.md`, `START-HERE.md`, and similar root-level docs) to confirm they are not outdated, given that a lot of content has changed recently. Classification: **bug** (documentation drift / staleness), surfaced by comparing doc claims against the actual repository state.

## Summary
`README.md` and `START-HERE.md` themselves are current, internally consistent, and every link in them resolves. The staleness is concentrated in the repository's two lookup indexes — `MASTER_INDEX.md` and `CONCEPT-INDEX.md` — which have not been edited since 2026-08-12, before the entire `verbs/` curriculum (17 files, added 2026-08-18) and `assessment/` directory (9 files, added 2026-08-17) existed. Both indexes advertise themselves as comprehensive, yet contain zero references to verbs or the assessment. A ready-made fix script, `_apply-index-updates.py`, was written specifically to patch the verb gap and was never run; its hard-coded anchors still match the current file content byte-for-byte, so it would apply cleanly today. That script and its companion instructions file, `APPLY-OVERLAY.md`, are themselves stale scaffolding — both explicitly say to delete themselves once applied, and neither has been. Separately, `GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md`'s changelog stops at 2026-08-12 and does not reflect either the verb-curriculum drop or the assessment feature, both of which shipped outside its own tracked 14-step plan.

## Current behavior

**This is not a software repository.** It is a static Markdown-based German A1 self-study course (no `CLAUDE.md` present, no build/app, no API). "High-level documentation" here means the root-level navigation/index files: `README.md`, `START-HERE.md`, `CONCEPT-INDEX.md`, `MASTER_INDEX.md`, `STUDY_GUIDE.md`, `GLOSSARY.md`, `GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md`, `APPLY-OVERLAY.md`.

Link integrity check (every `.md` link in every top-level doc resolved against the filesystem) — **zero broken links** anywhere. Navigation itself works.

Git history shows three content drops since the docs were last fully synchronized:
- `2026-08-17 fd8fc51 adds assessment` — adds the entire `assessment/` + `assessment/grader-only/` tree (9 files).
- `2026-08-18 31b2755 adds verbs` — adds the entire `verbs/` tree (17 files) and replaces `README.md`, `START-HERE.md`, `STUDY_GUIDE.md`, `GLOSSARY.md`, `review/README.md` (per `APPLY-OVERLAY.md:8-13`, this was an externally-prepared "overlay" applied by hand because a connected GitHub integration returned HTTP 403).
- `2026-08-18 88b339b updates to navigation` — per `.claude/work/guided-learning-path/summary.md`, adds a `Next →` spine link to 135 pages and links `assessment/README.md` from `START-HERE.md`/`README.md`.

`MASTER_INDEX.md` and `CONCEPT-INDEX.md` were last touched by `2026-08-12 599fd78 steps roadmap` — before both the assessment and verb drops.

### MASTER_INDEX.md is missing the entire verb system and assessment
- `MASTER_INDEX.md:5` claims: "Every word covered in this repository, alphabetically" — but `grep -c "verbs/" MASTER_INDEX.md` = 0. None of `sein`, `haben`, `werden`, modal verbs, separable verbs, reflexive verbs, Perfekt, or Präteritum appear anywhere.
- `MASTER_INDEX.md:143-149` ("Review Exercises") lists 5 reviews but omits `review/verb-conjugation-01.md`, which exists and is Stage 8's checkpoint in `START-HERE.md:171`.
- `MASTER_INDEX.md:162` ("Category Indexes") lists `Words · Prepositions · Particles · Patterns · Comparisons · Phrases · Review · Reference` — omits `Verbs` even though `verbs/README.md` exists and is the first content link in `README.md`'s own organization table (`README.md:77`).
- `grep -c "assessment" MASTER_INDEX.md` = 0 — no reference to the progress assessment at all.

### CONCEPT-INDEX.md has no route into the verb system
- `CONCEPT-INDEX.md:9-24` ("Find the problem you mean") has 11 rows covering negation, amount, time, movement, location, source, people, means/manner, time-with-prepositions, conversational tone, uncertainty, cause — **none route to conjugation, modals, separable verbs, reflexives, or the past tense**, despite the repository's own README now describing verb conjugation as one of its two core teaching areas (`README.md:3-6`).
- No `## Verb forms and conjugation` section exists anywhere in the file.
- `CONCEPT-INDEX.md:253-260` ("Other ways to find material") lists Start Here, Master Index, Comparisons, Preposition Overview, Review, Glossary — omits `verbs/README.md`.
- `grep -c "assessment" CONCEPT-INDEX.md` = 0.

### The exact fix already exists, unexecuted
`_apply-index-updates.py` (repo root) is a script whose entire purpose, per its own docstring-equivalent and `APPLY-OVERLAY.md:13`, is to "safely insert the new verb material into the existing `MASTER_INDEX.md` and `CONCEPT-INDEX.md`." It performs 9 anchored text replacements: a Master Index intro sentence, three new alphabetical entries (`haben`, `sein`, `werden`), a new "Verb Conjugation" section, a new Review Exercises row, a `Verbs` category-index link, a new Concept Index problem-table routing row, a new "Verb forms and conjugation" section, and a new "Other ways to find material" row.

I compared every one of the script's hard-coded `old` anchor strings against the current file contents; **all match verbatim** (e.g. `_apply-index-updates.py:30` expects `## H\n\n- [**halt**](particles/halt.md)...` which is exactly `MASTER_INDEX.md:47`'s current text). The script was written against this exact file state and would still apply cleanly — it has simply never been run. `APPLY-OVERLAY.md:19-38` gives the exact command (`python _apply-index-updates.py`) and instructs deleting both files afterward; neither step happened.

### GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md tracking is stale
The roadmap's own stated purpose (`GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md:3-5`) is to track planned improvements "so changes are completed in a deliberate order without losing scope." Its changelog (`:465-478`) ends 2026-08-12 with "Marked Step 3 — Confusion Index ready to begin." It does not log:
- the verb-curriculum overlay (a scope change explicit enough that `APPLY-OVERLAY.md:42` itself calls it "broadened from only 'small words' toward German A1 foundations" — i.e. exactly the repository's mission statement in `README.md:1-6`), or
- the `assessment/` feature, which isn't represented anywhere in the roadmap's 14 planned steps (Start Here, Concept Index, Confusion Index, content-gap audit, pattern families, status labels, review guide, cross-linking, pronunciation notes, visual diagrams, English-intention lookup, micro-dialogues, cumulative reviews, quick reference) — it was added outside the tracked plan entirely.

### STUDY_GUIDE.md and GLOSSARY.md are current
Both were part of the same overlay drop that added `verbs/` (`APPLY-OVERLAY.md:12` lists them as "replacement" files). Confirmed: `STUDY_GUIDE.md` has dedicated verb-study sections (`STUDY_GUIDE.md:23-104`) and `GLOSSARY.md` has a full "Verb Terms" section (`GLOSSARY.md:23-51`). Neither mentions `assessment/`, but neither claims to be a comprehensive index (Study Guide = study methods, Glossary = grammar terminology), so that omission does not misrepresent either page's stated purpose.

### verbs/README.md and review/README.md are current
Both were written as part of the same overlay and correctly cross-reference each other, `review/verb-conjugation-01.md`, and the rest of the repository. No staleness found in either.

## Root cause
The verb-curriculum overlay (`31b2755`) replaced the root docs and added `verbs/` per `APPLY-OVERLAY.md`'s process, but the final step of that process — running `_apply-index-updates.py` — was skipped, leaving `MASTER_INDEX.md` and `CONCEPT-INDEX.md` exactly as they were pre-overlay. Separately, `assessment/` (`fd8fc51`) was added a day earlier with no equivalent index-patch step at all — the script doesn't cover assessment (it predates that feature), and nothing else was written to add it. The roadmap changelog was simply never revisited after either drop.

## Affected surface area
Single repository, docs only:
- `MASTER_INDEX.md` — add verb entries/section, verb review row, `Verbs` category link, assessment reference.
- `CONCEPT-INDEX.md` — add verb-forms section and routing rows, `Verbs` link, assessment reference.
- `GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md` — changelog entries for the verb overlay and assessment feature.
- `APPLY-OVERLAY.md`, `_apply-index-updates.py` — delete once the index script has been run (per their own instructions).

## Existing patterns to follow
`_apply-index-updates.py` is itself the established pattern for patching these two index files — anchored, idempotent (`replace_once` no-ops if the new text is already present) text substitution. It should be run as-is for the verb portion rather than hand-edited, since it was purpose-built and its anchors are verified current.

## API contract impact
None — static Markdown content repository, no API surface.

## Data and migration impact
None.

## Test coverage today
No automated tests exist (not a software repo). This audit's link-integrity check (all `.md` links in all top-level docs resolved against the filesystem) is the only verification performed; it found zero broken links, confirming the staleness is a *completeness* problem, not a *broken-reference* problem.

## Options considered
Whether to run the existing `_apply-index-updates.py` versus hand-writing new edits to `MASTER_INDEX.md`/`CONCEPT-INDEX.md`. **Recommendation: run the existing script** for the verb portion — it was purpose-built for exactly this gap, its anchors are confirmed to still match, and it's referenced by `APPLY-OVERLAY.md` as the intended completion step. Hand-edit only the assessment additions, which the script doesn't cover.

## Constraints and risks
`_apply-index-updates.py` performs hard-coded anchor replacement and raises `SystemExit` if an anchor is missing rather than silently corrupting the file — low risk, verified anchors match. Running it and then separately hand-adding assessment entries to the same two files in one pass avoids a second conflicting diff.

## Open questions

**Blocking:** None — the fix is fully specified by the existing script plus the gaps documented above.

**Non-blocking:**
- Should `assessment/` become an explicit step in `GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md`'s 14-step plan, or just get a changelog note marking it as shipped outside the plan? Assumption: a changelog note is sufficient since it's already built, not proposed — the roadmap tracks *planned* work.
- Should `GLOSSARY.md`/`STUDY_GUIDE.md` gain assessment-related content? Assumption: no — neither page claims to be a comprehensive index, so this isn't staleness, just possible future scope.

## Suggested ticket slicing

1. **01-run-verb-index-overlay-script**
   Run `_apply-index-updates.py` to patch `MASTER_INDEX.md`/`CONCEPT-INDEX.md` with the verb-conjugation entries it was written for, verify the diff, then delete `APPLY-OVERLAY.md` and `_apply-index-updates.py` per the overlay's own cleanup instructions.
   Depends on: nothing
   Repos: this repo (docs only)
   Reason for split: self-contained, mechanical, uses an existing verified script — no judgment calls.

2. **02-add-assessment-to-indexes**
   Manually add `assessment/` entries to `MASTER_INDEX.md` (a Review/Assessment row and Category Indexes link) and `CONCEPT-INDEX.md` ("Other ways to find material" row), since no script covers this drop.
   Depends on: 01 (touches the same two files; sequencing after the script run avoids the script's anchor-matching failing against hand-edited text)
   Repos: this repo
   Reason for split: requires editorial judgment (no ready-made script), distinct from the mechanical script run.

3. **03-update-roadmap-changelog**
   Add changelog entries to `GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md` documenting the verb-curriculum overlay landing and the assessment feature's addition, and note assessment's relationship to the tracked 14-step plan.
   Depends on: nothing (independent file, no anchor conflicts with 01/02)
   Repos: this repo
   Reason for split: independent file, purely a tracking/record update, no dependency on the index fixes actually landing first.

## Confidence
High. Every claim is backed by a direct file read, a `git log` timestamp, a `grep` count, or a verbatim string comparison between the unexecuted script's anchors and the current file contents.

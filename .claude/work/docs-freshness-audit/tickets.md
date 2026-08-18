````markdown
# Apply the verb-index overlay script and remove overlay scaffolding

**Type:** Bug
**Repos:** deutsche (this repository)
**Depends on:** Nothing
**Size:** S

## User Story
As a learner using the Master Index or Concept Index to look up German material, I want the verb-conjugation curriculum to actually appear in those indexes, so that I can find `sein`, `haben`, modal verbs, separable verbs, Perfekt, and the other verb lessons the same way I can find every preposition or particle.

## Context
`MASTER_INDEX.md` and `CONCEPT-INDEX.md` were last edited 2026-08-12 (`git log -- MASTER_INDEX.md CONCEPT-INDEX.md`), before the `verbs/` curriculum (17 files) was added on 2026-08-18 in commit `31b2755`. Both files currently contain zero references to `verbs/` (`grep -c "verbs/" MASTER_INDEX.md` and the same on `CONCEPT-INDEX.md` both return `0`), despite `MASTER_INDEX.md:5` claiming to cover "every word covered in this repository, alphabetically."

A script that fixes exactly this, `_apply-index-updates.py` (repo root), was written as part of the same overlay that added `verbs/` — `APPLY-OVERLAY.md:13` describes it as inserting "the new verb material into the existing `MASTER_INDEX.md` and `CONCEPT-INDEX.md`." It was never run. Every hard-coded anchor string the script looks for was verified (during research) to match the current file content verbatim — for example `_apply-index-updates.py:30`'s anchor `## H\n\n- [**halt**](particles/halt.md)...` is exactly `MASTER_INDEX.md:47` today. The script is safe to run as-is: `replace_once()` raises `SystemExit` on a missing anchor rather than corrupting the file, and no-ops if the replacement text is already present.

`APPLY-OVERLAY.md:32-38` gives the exact apply procedure, including deleting the overlay files once the diff looks correct:
```bash
python _apply-index-updates.py
git diff --check
git diff
rm _apply-index-updates.py APPLY-OVERLAY.md
```
Neither the script run nor the cleanup happened. Both files are still sitting in the repo root as stale scaffolding.

## Acceptance Criteria
AC1 - Running `_apply-index-updates.py` from the repository root completes without error and prints `Updated MASTER_INDEX.md and CONCEPT-INDEX.md.`
AC2 - `MASTER_INDEX.md` contains alphabetical entries for `haben`, `sein`, and `werden` in their respective letter sections, a `## Verb Conjugation` section listing the verb curriculum index and all 12 verb lesson/quiz/reference pages, a `Verb Conjugation 01` row under Review Exercises, and `Verbs` added to the Category Indexes line.
AC3 - `CONCEPT-INDEX.md` contains a `## Verb forms and conjugation` section with routing rows covering present-tense basics, `sein`/`haben`/`werden`, stem changes, modal verbs, separable verbs, reflexive verbs, Perfekt, Präteritum, and the high-frequency reference; two new routing rows added to the "Find the problem you mean" table; and a `Verbs` row added to "Other ways to find material."
AC4 - Every Markdown link in the updated `MASTER_INDEX.md` and `CONCEPT-INDEX.md` resolves to a file that exists in the repository (no broken links introduced).
AC5 - `_apply-index-updates.py` and `APPLY-OVERLAY.md` no longer exist in the repository.

## Implementation Plan
1. From the repository root, run `python3 _apply-index-updates.py` (confirm `python3` is the correct interpreter available in the environment; the shebang specifies `#!/usr/bin/env python3`).
2. Confirm the script prints `Updated MASTER_INDEX.md and CONCEPT-INDEX.md.` and exits without a traceback.
3. Run `git diff MASTER_INDEX.md CONCEPT-INDEX.md` and review the inserted content against AC2/AC3 above.
4. Re-run the link-integrity check used during research (resolve every `](...)` Markdown link target in both files against the filesystem) to confirm no broken links were introduced.
5. Delete `_apply-index-updates.py` and `APPLY-OVERLAY.md` (`rm _apply-index-updates.py APPLY-OVERLAY.md`).
6. Stage and commit `MASTER_INDEX.md`, `CONCEPT-INDEX.md`, and the two deletions together.

## API Contract Changes
None — static Markdown content repository, no API surface.

## Postman Updates
None.

## Out of Scope
- Adding `assessment/` to either index file — covered by a separate ticket, since `_apply-index-updates.py` does not touch assessment content at all (it predates that feature).
- Updating `GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md` — covered by a separate ticket.
- Rewriting or restyling any verb lesson content itself — `verbs/README.md`, `STUDY_GUIDE.md`, and `GLOSSARY.md` are already current and are not touched here.
- Any change to the script's inserted wording — the script's text was written as part of the original overlay and is treated as correct; do not hand-edit its output unless AC2/AC3 are not met.

## Testing Guidance
- No automated test suite exists in this repository. Verification is manual: run the script, diff the output against the anchors and inserted text listed in AC2/AC3, and confirm every link in the two changed files resolves to a real file (the same check used in the research pass — grep out `](...)` targets and test each with a file-existence check).
- Visually confirm the new `## Verb Conjugation` and `## Verb forms and conjugation` sections render as valid Markdown (correct heading level, no unclosed table rows).

## Edge Cases
- Script raises `SystemExit` on a missing anchor → stop immediately; this means a file changed since the research pass and the anchors need to be re-derived by hand rather than forced. Do not comment out the failing `replace_once` call to work around it.
- Running the script a second time after success is a safe no-op (each `replace_once` short-circuits when the new text is already present) — not required, but not harmful if attempted before verification.

## Open Questions
None blocking. Non-blocking: none identified.
````

````markdown
# Add the progress assessment to the Master and Concept indexes

**Type:** Bug
**Repos:** deutsche (this repository)
**Depends on:** Apply the verb-index overlay script and remove overlay scaffolding
**Size:** S

## User Story
As a learner who has finished (or is looking for) the closed-book progress assessment, I want it listed in the Master Index and Concept Index, so that I don't have to already know the `assessment/` folder exists to find it.

## Context
`assessment/README.md` and its eight companion files were added in commit `fd8fc51` ("adds assessment", 2026-08-17) and are already linked from `README.md`'s organization table and from `START-HERE.md`'s "Whole-Repository Review" section (per `.claude/work/guided-learning-path/summary.md`, which added that link as part of an earlier navigation pass). However, `MASTER_INDEX.md` and `CONCEPT-INDEX.md` predate the assessment addition entirely (last touched 2026-08-12) and contain zero references to it (`grep -c "assessment" MASTER_INDEX.md` and the same on `CONCEPT-INDEX.md` both return `0`).

Unlike the verb-curriculum gap, there is no existing script covering this — `_apply-index-updates.py` was written before `assessment/` existed and does not mention it. This ticket is a small hand-edit, sequenced after the verb-index ticket so both changes to these two files land as one clean diff rather than two overlapping ones.

## Acceptance Criteria
AC1 - `MASTER_INDEX.md` contains a link to `assessment/README.md`, placed in the "Review Exercises" section, described consistently with how `README.md`'s organization table already describes it ("closed-book test of where your A1 ability actually stands, once you've finished the path" — `README.md:86`).
AC2 - `CONCEPT-INDEX.md` contains a link to `assessment/README.md` in the "Other ways to find material" section, in the same one-line bullet style as the other entries there.
AC3 - The link text and description in both files are consistent with each other and with the existing `README.md`/`START-HERE.md` framing of the assessment (closed-book, taken after finishing the learning path).
AC4 - No other content in `MASTER_INDEX.md` or `CONCEPT-INDEX.md` is altered.

## Implementation Plan
1. In `MASTER_INDEX.md`, add one bullet to the "Review Exercises" list (after the verb-index ticket has added the "Verb Conjugation 01" row there), e.g.:
   `- [Progress Assessment](assessment/README.md) — closed-book test of where your A1 ability stands, once you've finished the path`
2. In `CONCEPT-INDEX.md`, add one bullet to "Other ways to find material", e.g.:
   `- [Progress Assessment](assessment/README.md) — closed-book test of current A1 ability, once you've finished the path`
3. Confirm the link resolves and wording matches the tone/length of neighboring bullets in each list.

## API Contract Changes
None.

## Postman Updates
None.

## Out of Scope
- Adding assessment-related content to `STUDY_GUIDE.md` or `GLOSSARY.md` — neither page claims to be a comprehensive index (Study Guide covers study methods, Glossary covers grammar terms), so their not mentioning the assessment is not staleness. Flagged as a non-blocking open question below, not done here.
- Changing `assessment/README.md` or any file inside `assessment/` or `assessment/grader-only/`.
- Adding a new top-level heading/section to either index file — this ticket only adds list entries to existing sections.

## Testing Guidance
Manual: confirm the added link resolves to `assessment/README.md`, and that the two index files still parse as valid Markdown (list bullets, no broken table syntax) after the edit.

## Edge Cases
None beyond standard link-typo risk — verify the relative path `assessment/README.md` resolves correctly from repository root in both files.

## Open Questions
**Non-blocking:** Should `GLOSSARY.md` or `STUDY_GUIDE.md` also reference the assessment? Assumption: no — treated as a possible future scope decision, not a staleness bug, since neither page claims comprehensive coverage. Confirm with the repository owner if a stronger cross-link is wanted.
````

````markdown
# Log the verb-curriculum and assessment additions in the improvement roadmap

**Type:** Chore
**Repos:** deutsche (this repository)
**Depends on:** Nothing
**Size:** S

## User Story
As the repository maintainer, I want `GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md`'s changelog to reflect what has actually shipped, so that the roadmap stays trustworthy as the single place tracking what's done versus planned.

## Context
`GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md`'s stated purpose (`:3-5`) is to track planned improvements "so changes are completed in a deliberate order without losing scope." Its changelog (`:465-478`) ends at `## 2026-08-12` with "Marked Step 3 — Confusion Index / Confusion Clinic ready to begin." Two substantial changes have shipped since without a changelog entry:

- The verb-curriculum overlay (commit `31b2755`, 2026-08-18): added the entire `verbs/` directory and replaced `README.md`, `START-HERE.md`, `STUDY_GUIDE.md`, `GLOSSARY.md`, and `review/README.md`. `APPLY-OVERLAY.md:42` itself describes this as broadening the repository "from only 'small words' toward German A1 foundations" — the same reframing now visible in `README.md:1-6`.
- The `assessment/` feature (commit `fd8fc51`, 2026-08-17): a 9-file closed-book progress assessment, now linked from `README.md` and `START-HERE.md`. This feature is not represented anywhere in the roadmap's 14 planned steps (Start Here, Concept Index, Confusion Index, content-gap audit, pattern families, status labels, review guide, cross-linking, pronunciation notes, visual diagrams, English-intention lookup, micro-dialogues, cumulative reviews, quick reference) — it shipped outside the tracked plan entirely.

This ticket only adds tracking/record content. It does not resequence, re-scope, or change the status of any of the 14 existing steps.

## Acceptance Criteria
AC1 - The changelog contains a dated entry (`## 2026-08-17` or later, matching the existing `## YYYY-MM-DD` heading style) documenting that `assessment/` was added, noting it was delivered outside the numbered step plan.
AC2 - The changelog contains a dated entry (`## 2026-08-18` or later) documenting the verb-curriculum overlay: the new `verbs/` directory and the replacement of `README.md`, `START-HERE.md`, `STUDY_GUIDE.md`, `GLOSSARY.md`, and `review/README.md`.
AC3 - A brief note is added near "Current Focus" (`:455-461`) acknowledging that the assessment feature exists and shipped outside the ordered plan, without renumbering or blocking Step 3.
AC4 - None of the 14 existing step entries, their `Status` fields, or their completion-criteria checkboxes are modified — this ticket only adds changelog and "Current Focus" text.

## Implementation Plan
1. Add a new `## 2026-08-17` section to the changelog (before the existing `## 2026-08-12` section, since changelog entries read newest-first per the existing file order — confirm order against the current file before inserting) describing the `assessment/` addition in the same terse, past-tense bullet style as existing entries.
2. Add a new `## 2026-08-18` section describing the verb-curriculum overlay and its replaced files, in the same style.
3. Add one or two sentences under or near "Current Focus" noting that `assessment/` shipped as a feature outside the 14-step plan, so a reader doesn't mistake its absence from the step list for an oversight.
4. Leave every other section of the file untouched.

## API Contract Changes
None.

## Postman Updates
None.

## Out of Scope
- Adding "Progress Assessment" as a new numbered step (Step 15) in the ordered plan — see Open Questions; not done by default.
- Any change to `MASTER_INDEX.md`, `CONCEPT-INDEX.md`, `APPLY-OVERLAY.md`, or `_apply-index-updates.py` — covered by the other two tickets in this set.
- Re-evaluating or changing the `Status` of Step 3 ("Ready to begin") or any other step.

## Testing Guidance
Manual read-through: confirm the new changelog entries use the same heading level and bullet style as the existing 2026-08-12 entry, and that the file still renders as valid Markdown.

## Edge Cases
None — this is a text-only documentation update with no functional behavior to break.

## Open Questions
**Non-blocking:** Should `assessment/` become an explicit, numbered step in the roadmap's ordered plan (a "Step 15" or similar), rather than just a changelog note? Assumption made here: a changelog note is sufficient, since the roadmap tracks *planned* work and this feature is already built, not proposed. Revisit if the repository owner wants the 14-step plan itself to reflect assessment as a completed, tracked initiative rather than an aside.
````

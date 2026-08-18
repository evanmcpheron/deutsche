# Applying This Overlay

This directory is a complete verb-conjugation expansion prepared against the current `main` branch of `evanmcpheron/deutsche`.

The connected GitHub integration could read the repository but returned HTTP 403 for both branch creation and file writes, so the changes could not be pushed from ChatGPT.

## What Is Included

- new `verbs/` curriculum and reference pages;
- separate quizzes with hidden solution dropdowns;
- `review/verb-conjugation-01.md`, a 30-item mixed checkpoint;
- replacement `README.md`, `START-HERE.md`, `STUDY_GUIDE.md`, `GLOSSARY.md`, and `review/README.md`;
- `_apply-index-updates.py`, which safely inserts the new verb material into the existing `MASTER_INDEX.md` and `CONCEPT-INDEX.md`.

## Apply to the Repository

Extract/copy the overlay contents into the repository root, preserving paths. This intentionally replaces the root Markdown files listed above.

Then, from the repository root, run:

```bash
python _apply-index-updates.py
```

Review the diff:

```bash
git diff --check
git diff
```

If the changes look correct, remove the helper files before committing:

```bash
rm _apply-index-updates.py APPLY-OVERLAY.md
```

Then commit normally.

## Scope Decisions Reflected Here

- The repository is broadened from only “small words” toward **German A1 foundations**, without pretending it is already a complete A1 course.
- Present-tense conjugation is placed at the beginning of the canonical learning sequence.
- Modal, separable, and reflexive verbs come after the learner has a base of vocabulary and sentence structure.
- Perfekt is an active A1 production target.
- Präteritum is selective: `war` and `hatte` receive active attention; common modal past forms are primarily recognition-first.
- Verb quizzes require correct grammatical spelling.
- Mixed retrieval removes lesson-title clues.
- High-frequency irregular verbs receive dedicated treatment; ordinary verbs are grouped by conjugation pattern.

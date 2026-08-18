# Research: Content and navigation audit of the `comparisons/` directory

## Request
Do a thorough audit of everything in `comparisons/` (14 comparison pairs — 14 lessons + 14 quizzes): is the German information correct, are quiz questions stated with the right amount/level of German, do quizzes test only the two named words, do "Next" links follow `START-HERE.md`'s guided order, and does the last comparison in the guided sequence point to the correct next section. Also diagnose and propose a fix for the two `audit_quizzes.py` structural errors in `comparisons/auch-vs-noch-quiz.md` (Q1 and Q26 each report "expected exactly one solution dropdown"). Classification: **refactor/content-QA** (this is a static markdown A1-German curriculum repository, not a software repo; no `CLAUDE.md` exists at the repo root — confirmed absent). This is a direct continuation of the same audit performed on `words/` (`.claude/work/words-directory-audit/findings.md`), reusing its methodology and severity bar.

## Summary
Navigation is entirely correct: all 11 comparisons with a fixed slot in `START-HERE.md`'s guided sequence have quiz-footer "Next →" links that match the guided order exactly, with zero mismatches, and `comparisons/README.md`'s lookup-order table is accurate and complete. The three "Later"/deferred comparisons (`sehr-vs-viel`, `gerade-vs-jetzt`, `denn-vs-weil`) chain to whatever comes next in `START-HERE.md`'s linear document order rather than to a prerequisite-gated lesson, which is consistent with how deferred material already behaves elsewhere in the repo — not a bug. German content is grammatically correct across all 14 pairs — zero grammar/case/word-order errors found in two independent full close-readings. The one structural bug in `auch-vs-noch-quiz.md` has a fully diagnosed, one-line-per-item root cause: `audit_quizzes.py`'s question-marker regex doesn't understand `<details>` boundaries relative to raw numbered sub-lists, and Q26 happens to use one. Five genuine content-QA issues were found across five different pairs — two are true scope leaks (a correct answer requiring a word outside the compared pair, in one case one that hasn't even been taught yet at that point in the curriculum), three are milder "untaught vocabulary sits in the load-bearing correct answer" issues.

## Current behavior

### Structure
`comparisons/` holds 14 lesson+quiz pairs, matching `comparisons/README.md`'s index exactly (8 "Words That Look Interchangeable," 5 "Prepositions That Compete," 1 "Connectors"). 11 of the 14 sit at a specific numbered position inside `START-HERE.md`'s 9 guided stages; 3 (`sehr-vs-viel`, `gerade-vs-jetzt`, `denn-vs-weil`) are explicitly called out as deferred/"Later" material outside the numbered sequence (`START-HERE.md:112`, `:194`).

### Navigation — verified clean, zero mismatches
The quiz-footer `Next →` link (the true study-order pointer, same convention documented in the `words/` audit) was traced for all 14 quizzes against `START-HERE.md`'s actual numbered-list positions:

| Comparison | Stage.item | Footer `Next →` target | Correct? |
|---|---|---|---|
| nicht-vs-kein | S2.3 | `words/auch.md` | ✓ |
| auch-vs-noch | S2.7 | `words/schon.md` | ✓ |
| noch-vs-schon | S2.9 (stage-last) | `review/recognition-01.md` | ✓ |
| nicht-mehr-vs-noch | S3.3 | `words/immer.md` | ✓ |
| noch-vs-wieder | S3.7 | `words/erst.md` | ✓ |
| erst-vs-schon | S3.9 (stage-last) | `words/sehr.md` | ✓ |
| bei-vs-mit | S5.3 | `prepositions/zu.md` | ✓ |
| zu-vs-nach | S5.6 | `comparisons/zu-hause-vs-nach-hause.md` | ✓ |
| zu-hause-vs-nach-hause | S5.7 | `prepositions/fuer.md` | ✓ |
| aus-vs-von | S6.3 | `patterns/von-bis.md` | ✓ |
| in-vs-nach | S6.7 (last stage-embedded comparison overall) | `prepositions/auf.md` | ✓ |

The 3 deferred comparisons chain to the next block of `START-HERE.md`'s linear text rather than a gated lesson: `sehr-vs-viel-quiz.md` → `gerade-vs-jetzt.md`; `gerade-vs-jetzt-quiz.md` → `reference/preposition-overview.md` (Stage 5's pre-reading); `denn-vs-weil-quiz.md` → `review/cumulative-review.md` (the Whole-Repository Review section, the next thing in the document after the Stage 9 "Later" callout at `START-HERE.md:194`). This is the same pattern already used for all three deferred pages, so it's self-consistent, not a defect.

`comparisons/README.md` was independently checked for internal consistency: 14 directory pairs = 14 table rows, no stale or missing entries, all links resolve, and it correctly self-disclaims as lookup order rather than study order (`README.md:5`).

### The "last comparison" question
Two valid readings, both worth stating for the record:
- **By fixed stage position**, `in-vs-nach` (Stage 6, item 7) is the last comparison with an explicit slot in the guided march, and it correctly forwards to `prepositions/auf.md`.
- **By raw document position**, `denn-vs-weil` is mentioned last in `START-HERE.md` (line 194), after the Stage 9 checkpoint, explicitly deferred ("should wait until basic `weil` word order has been learned"). Its footer link to Cumulative Review is not a prerequisite-gated "what to do next," because deferred material doesn't have one — but it is what the document presents next, matching the other two deferred pages' identical behavior.

Neither reading produces a broken or incorrect pointer. `audit_quizzes.py` also independently confirms zero broken relative links anywhere in the repo (repo-wide check, not `comparisons/`-specific).

## Root cause — auch-vs-noch-quiz.md Q1/Q26 structural errors
Verified directly by reading `comparisons/auch-vs-noch-quiz.md:422-437` against `audit_quizzes.py`'s parser (`audit_quizzes.py:26,33-46`).

`audit_quizzes.py`'s `Q_RE` matches any line starting with `\d+\.\s`, anywhere outside a `<details>...</details>` span. Question 26 ("Produce it: Write B's reply in each case") presents its two production scenarios as a literal markdown ordered sub-list:
```
26. **Produce it:** Write B's reply in each case.

1. A: _Ich habe keine Zeit._ B agrees.
1. A: _Ich habe Zeit._ B agrees.

<details>
...
```
Both scenario lines literally start with `1. ` (not `1.`/`2.` — GFM auto-renumbers, so the source uses `1.` twice), and both sit *before* the `<details>` block opens, so `Q_RE` matches them as new top-level question markers. This does two things:
1. **Truncates the real Q26** — `split_questions` now ends Q26's body at the first spurious `1.` marker (line 424), one line before its `<details>` block, so Q26 as parsed has zero `<details>` tags → "Q26: expected exactly one solution dropdown."
2. **Creates a phantom duplicate "Q1"** — the spurious marker at line 424 is itself numbered `1`, colliding with the real, structurally-fine Q1 near the top of the file. This phantom fragment (line 424 to line 425) also has zero `<details>` tags → "Q1: expected exactly one solution dropdown." (The real Q1, lines 13–26, is unaffected and structurally correct on its own.)

Confirmed via repo-wide grep that this exact pattern — a raw numbered sub-list starting a line outside `<details>`, used for a non-multiple-choice production question — appears nowhere else in the repository; every other multi-part "Produce it"/"Fill in" question in `comparisons/` (e.g. Q23's "Select all that apply" list) uses letter-prefixed (`a.`/`b.`/`c.`/`d.`) options, which `Q_RE` never matches (it only matches leading digits).

**Proposed fix** (content-only, does not touch answer text or teaching content): replace the two literal `1. A: ...` lines with a format `Q_RE` cannot match — e.g. bullet dashes:
```
- A: _Ich habe keine Zeit._ B agrees.
- A: _Ich habe Zeit._ B agrees.
```
This avoids both `Q_RE` (needs leading digits) and `OPT_RE` (needs a leading `a.`–`d.` letter, which would otherwise make the script think this is an ungraded multiple-choice question), renders identically in GitHub-flavored Markdown, and requires no change to `audit_quizzes.py` itself. Re-running the script after this one edit should drop the problem count from 9 to 7 (the remaining 7 are pre-existing, unrelated issues in `verbs/*.md`, outside this directory's scope).

## Content-QA findings (German correctness, question level, quiz scope)

Two parallel close-readings covered all 14 pairs (28 files); every correct answer in every question was checked against the pair's two named words, and all German was checked for grammar/case/word-order correctness.

**Fully clean (9 of 14 pairs, zero issues):** `auch-vs-noch`, `aus-vs-von`, `denn-vs-weil`, `gerade-vs-jetzt`, `in-vs-nach`, `nicht-mehr-vs-noch`, `nicht-vs-kein`, `noch-vs-schon`, `zu-vs-nach`. No grammar errors found; every load-bearing correct answer is one of the two compared words (or a taught inflected/negated form); untaught words appear only as never-correct distractors, which the prior `words/` audit already established as acceptable.

**Real scope leaks — correct answer requires a third, uncompared concept:**

- **`comparisons/bei-vs-mit-quiz.md:190-201` (Q12) and `:453-465` (Q27)** — both require **`zu`** as the sole correct answer ("Ich gehe zu meiner Schwester"), not `bei` or `mit`. Verified directly. This is more than a mild category slip: `bei-vs-mit` sits at Stage 5 item 3, and `zu` is not taught until Stage 5 item 4 (`prepositions/zu.md`) — two steps *later*. The quiz requires an answer the learner hasn't been taught yet at the point they'd take it. `bei-vs-mit.md:62` does preview this as a "Common Mistake" aside, but a one-line aside in the lesson doesn't make `zu` a legitimate load-bearing quiz answer this early.
- **`comparisons/sehr-vs-viel-quiz.md:167-180` (Q11)** — scenario "You are extremely hungry," correct answer "b. Ich habe großen Hunger," which requires accusative adjective declension (`groß` → `großen`). Verified directly. Adjective declension is not taught anywhere in this curriculum (confirmed by repo-wide search). Neither `sehr` nor `viel` is the answer at all — this is the cleanest example of the pattern the prior `words/` audit flagged as most severe (a correct answer requiring untaught grammar entirely outside the pair).

**Untaught vocabulary inside a load-bearing answer (milder — the pair itself stays correct, but the phrasing needed to state the answer isn't yet taught):**

- **`comparisons/noch-vs-wieder-quiz.md:307-320` (Q19)** — correct answer "d. Kein zweites Mal" uses the ordinal "zweites Mal" (second time), never taught anywhere in the curriculum (no lesson covers ordinal numbers). Verified directly.
- **`comparisons/zu-hause-vs-nach-hause-quiz.md:315-330` (Q19)** — correct answer "a. Um fünf geht er los" uses the separable verb *losgehen* (split as "geht ... los"), unglossed. Verified directly. Separable verbs aren't taught until Stage 7 (`verbs/separable-verbs.md`), well after this Stage 5 quiz.
- **`comparisons/erst-vs-schon-quiz.md:288-301` (Q18)** — "Erst mal einen Kaffee" tests the sequencing sense of `erst` ("first of all"), which `erst-vs-schon.md` never teaches — the lesson only teaches the "opinion attached to a number" sense. Verified directly; this also stands in mild tension with the same quiz's own Q19 explanation two questions later ("neither of them is ever neutral" / always attaches an opinion to a number), since Q18's usage has no number at all.

No German grammar/case/word-order errors were found in any of the 14 pairs — every issue above is a scope/level-appropriateness issue, not a correctness error.

## Affected surface area
All changes are confined to `comparisons/` — five quiz files need question-level edits, no lesson pages need changes, no other directory is touched:
- `comparisons/auch-vs-noch-quiz.md` — structural fix (Q26's sub-list format)
- `comparisons/bei-vs-mit-quiz.md` — Q12, Q27 (replace `zu`-dependent items)
- `comparisons/sehr-vs-viel-quiz.md` — Q11 (replace adjective-declension item)
- `comparisons/noch-vs-wieder-quiz.md` — Q19 (replace ordinal-dependent phrasing)
- `comparisons/zu-hause-vs-nach-hause-quiz.md` — Q19 (replace separable-verb phrasing)
- `comparisons/erst-vs-schon-quiz.md` — Q18 (drop, or add a short teaching note first)

## Existing patterns to follow
- `audit_quizzes.py` (repo root) — the repo's own structural linter; already run as part of this audit and should stay green (modulo the pre-existing unrelated `verbs/` issues) after the Q26 fix.
- `.claude/work/words-directory-audit/findings.md` — the prior audit of the same shape on `words/`; this audit reused its severity bar (never-correct distractors using not-yet-taught words = fine; load-bearing correct answers requiring untaught grammar/vocab = a real leak) and its two-navigation-system model (lookup-order breadcrumb vs. quiz-footer study-order chain).
- Q23 in `auch-vs-noch-quiz.md` (letter-prefixed multi-item list) is the in-repo precedent for how to format a multi-scenario question without colliding with `audit_quizzes.py`'s parser — though for Q26 specifically, bullet dashes fit better since it isn't multiple-choice.

## API contract impact
None — static content repository, no API surface.

## Data and migration impact
None.

## Test coverage today
`audit_quizzes.py` is the only automated check. It covers structural integrity (question counts, answer-key/option consistency, duplicate options, broken links, lesson↔quiz pairing) but not semantic scope or German correctness — that gap is what this audit and the prior `words/` audit fill. None of the suggested fixes touch answer-key letters or option counts in a way that would need script changes; the Q26 structural fix should reduce the script's reported problem count in `comparisons/` from 2 to 0.

## Suggested ticket slicing

- **01-fix-auch-vs-noch-quiz-structural-bug** — Reformat `comparisons/auch-vs-noch-quiz.md` Q26's two-scenario list (currently literal `1.`/`1.` lines) into a format `audit_quizzes.py`'s `Q_RE` doesn't match (e.g. bullet dashes), fixing both the "Q26: expected exactly one solution dropdown" and phantom "Q1: expected exactly one solution dropdown" errors. `Depends on: nothing`. `Repos: this repo (comparisons/)`. Split out because it's a parser/structural fix, distinct in kind from the content-scope fixes below, and touches exactly one file.
- **02-fix-comparisons-load-bearing-scope-leaks** — Rewrite `bei-vs-mit-quiz.md` Q12 and Q27 so their correct answers are `bei` or `mit` (not `zu`, which isn't taught until two lessons later), and rewrite `sehr-vs-viel-quiz.md` Q11 so its correct answer is `sehr` or `viel` (not an untaught adjective-declension form). `Depends on: nothing`. `Repos: this repo (comparisons/)`. Grouped because both are the same severity tier — a correct answer requiring a concept entirely outside the compared pair — reviewable together.
- **03-fix-comparisons-untaught-vocab-polish** — Reword `noch-vs-wieder-quiz.md` Q19 (drop the untaught ordinal "zweites Mal"), `zu-hause-vs-nach-hause-quiz.md` Q19 (drop the unglossed separable verb "geht...los"), and `erst-vs-schon-quiz.md` Q18 (drop the item, or add a short teaching note about the "erst mal" sequencing idiom to the lesson page first). `Depends on: nothing`. `Repos: this repo (comparisons/)`. Lowest priority — the compared pair itself stays correct in each case; only the phrasing used to state the answer reaches outside the taught curriculum at that point.

## Constraints and risks
None beyond the usual content-review judgment calls: for `erst-vs-schon-quiz.md` Q18, the ticket author should pick a direction (drop the question vs. teach the idiom first) rather than leaving it ambiguous, same caveat the prior `words/` audit raised for its own borderline items.

## Open questions

**Non-blocking:**
- Should `erst-vs-schon.md` be expanded to teach the "erst mal" sequencing idiom (making Q18 legitimate), or should the question simply be dropped? Either resolves the issue; a content-repo maintainer's call.
- `bei-vs-mit.md:62`'s existing "Common Mistake" aside about `zu` — should it be trimmed now that `zu` is no longer a quiz answer, or left as forward-pointing context? Cosmetic, not blocking.

None blocking — every question above has a stated default (leave the lesson aside as-is; drop rather than expand Q18) that doesn't stop other work.

## Confidence
**High** on navigation-order verification (the full quiz-footer chain was traced against `START-HERE.md`'s actual numbered-list positions with zero mismatches, and the structural root-cause was confirmed by direct regex/file inspection, not just the subagent's report). **High** on German grammatical correctness (two independent full close-readings, zero grammar errors found, matching the same clean result the `words/` audit found). **Medium-high** on scope-leak completeness — two independent reviewers each covered 7 pairs closely and cross-checked claims against the actual curriculum position in `START-HERE.md` (not just whether a word appears elsewhere in the repo), which is a stronger check than the prior `words/` audit used; a different reviewer applying a stricter bar for "untaught vocabulary in flavor text vs. distractors" could still surface a few more borderline cases.

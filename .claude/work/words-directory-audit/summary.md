# Summary

All three tickets from `.claude/work/words-directory-audit/tickets.md` are implemented in this pass, since they are independent, disjoint-file, content-only changes with no dependency between them: (1) removing quiz questions in `words/` that required vocabulary or grammar not yet taught, including the one load-bearing case in `noch-quiz.md`; (2) removing two non-load-bearing premature-vocabulary distractors from `immer-quiz.md` and `wieder-quiz.md`; (3) signposting, on all 12 `words/*.md` lesson pages, that their top/bottom breadcrumb `Next`/`←` link follows a lookup order rather than the recommended study path.

# Changes

- `words/noch-quiz.md` — Q1, Q3, Q9 no longer use the untaught "nicht mehr" pattern in their distractors or explanations; Q14's correct answer no longer requires "nicht mehr" (the dialogue and correct answer are rewritten around the taught "noch nicht" pattern instead); Q8 and Q25 no longer require productive command of "wieder" (both are rewritten around material `noch.md` teaches directly — remaining-amount `noch` and the `noch nicht` chunk).
- `words/nicht-quiz.md` — Q13 no longer requires understanding the conjunction "sondern"; it now tests the minimal pair `Ich verstehe das.` / `Ich verstehe das nicht.`, which `nicht.md`'s own "What Changes Without It?" section teaches directly.
- `words/kein-quiz.md` — Q14 no longer tests the untaught "kein … mehr" pattern; it now tests plain `kein Brot` against three wrong endings/constructions drawn from `kein.md`'s own ending table and its "Common English-Speaker Mistakes" section. Q15 no longer treats "noch" as independently productive vocabulary; it now tests the masculine-accusative `keinen Bruder` ending, the exact example already given in `kein.md`'s Pattern Recognition section.
- `words/auch-quiz.md` — Q4's four answer options no longer contain a relative clause ("das Einzige, was ich trinke") or the separable verb "dazukommen"; the question is rewritten with plain-English options (mirroring the format already used in the file's own Q5) testing comprehension of what *auch* adds to `Ich trinke auch Kaffee.`
- `words/immer-quiz.md` — Q1, Q3, Q12, and Q14 no longer use standalone "wieder" (not yet taught at this point in the curriculum) as a distractor; each is replaced with a distractor built from vocabulary already taught by the time a learner reaches `immer` (`nicht immer`, plain `noch`, `schon`).
- `words/wieder-quiz.md` — Q5 and Q12 no longer use "erst" (not yet taught) as a distractor; Q5's replacement (`nicht mehr`) is also idiomatic, resolving the pre-existing wording issue in that distractor. Both replacements use vocabulary genuinely taught by this point in Stage 3 (`nicht mehr`, `immer noch`).
- `words/nicht.md`, `words/kein.md`, `words/auch.md`, `words/noch.md`, `words/schon.md`, `words/nur.md`, `words/sehr.md`, `words/immer.md`, `words/wieder.md`, `words/vielleicht.md`, `words/erst.md`, `words/gerade.md` — each now carries one identical italic line directly under its top breadcrumb: *"(This is the lookup order — see [Start Here](../START-HERE.md) for the recommended study path.)"* The three particle-stub pages (`words/doch.md`, `words/mal.md`, `words/eigentlich.md`) are unchanged, since they have no `Next` breadcrumb and don't participate in the lookup-order chain. Only the top breadcrumb carries the note; the bottom breadcrumb (which leads into the quiz, not further along the lookup chain) is left as-is.

# API Contract

Omitted — this repository has no API surface; it is a static markdown content set. None of the three tickets touch it.

# Tests

This repository has no unit test suite; its only automated check is `audit_quizzes.py` at the repo root, which verifies quiz structure (answer-key letters resolve against displayed options, no duplicate option text, no broken relative links, no run-of-three or cycling answer patterns, per-file question counts, exactly one `<details>` block per question). No new test was added or needed — none of the three tickets change the kind of thing this script checks, only quiz *content*.

# Manual Verification

No running system exists to exercise — this is a static markdown repository with no server, build step, or client. Runtime behavior is "what the rendered markdown says," so verification took the form of full content review (reading every edited question in context against the lesson page it belongs to) plus running the repo's own structural check, both specified as the applicable testing guidance in all three tickets:

| Case | Check | Expected | Actual | Result |
| ---- | ----- | -------- | ------ | ------ |
| Ticket 1 scope leaks fixed | Content review of `noch-quiz.md` Q1/Q3/Q8/Q9/Q14/Q25, `nicht-quiz.md` Q13, `kein-quiz.md` Q14/Q15, `auch-quiz.md` Q4 | Each answerable using only vocabulary/grammar taught by that lesson or earlier | Confirmed for all ten questions | Pass |
| Ticket 1 structural regression | `python3 audit_quizzes.py` | Zero problems in the four touched files, same-or-greater question counts | `noch-quiz.md` 26/26, `nicht-quiz.md` 26/26, `kein-quiz.md` 24/24, `auch-quiz.md` 22/22 — unchanged from baseline; zero problems reported for all four | Pass |
| Ticket 1 edge case: answer-run regression | `python3 audit_quizzes.py` after the Q8 rewrite | No run-of-three identical answers introduced | First pass introduced `words/noch-quiz.md: run of three "a" answers near Q8` — caught and fixed by reordering Q8's options so the correct answer is `c`, not `a` | Pass (after fix) |
| Ticket 2 distractor fixes | Content review of `immer-quiz.md` Q1/Q3/Q12/Q14, `wieder-quiz.md` Q5/Q12 | No untaught vocabulary in distractors; correct answers/explanations unchanged | Confirmed; only distractor text and its explanation clause changed in each case | Pass |
| Ticket 2 structural regression | `python3 audit_quizzes.py` | Zero problems, unchanged question counts | `immer-quiz.md` 18/18, `wieder-quiz.md` 20/20 — unchanged; zero problems | Pass |
| Ticket 3 signposting | Manual read of all 12 edited pages | Identical note text on every page, correct relative link | Confirmed: `grep -c "This is the lookup order"` returns 1 for all 12 word pages and 0 for the 3 stub pages | Pass |
| Ticket 3 link resolution | `python3 audit_quizzes.py` (repo-wide link check) | No new broken links from the added `../START-HERE.md` reference | Zero new problems reported | Pass |
| Ticket 3 README consistency | Manual read of `words/README.md:7` | Existing disclaimer still reads correctly, no contradiction | Unchanged, reads consistently alongside the new per-page notes | Pass |
| Full-repo regression | `python3 audit_quizzes.py` (final run, all tickets applied) | Only the 9 pre-existing, out-of-scope problems remain | Confirmed: `comparisons/auch-vs-noch-quiz.md` (2), `verbs/perfekt-quiz.md` (2), `verbs/praeteritum-essentials-quiz.md` (2), `verbs/present-tense-basics-quiz.md` (3) — none in `words/` | Pass |

# Verification

**Status: Complete.**

Every acceptance criterion across all three tickets is met and confirmed against the evidence in the table above, not against the edits alone:

- Ticket 1 AC1–AC10: confirmed by content review (AC1–AC7, AC10) and by `audit_quizzes.py` (AC8, AC9 — zero problems, unchanged-or-greater question counts in all four files).
- Ticket 2 AC1–AC5: confirmed by content review (AC1–AC4) and `audit_quizzes.py` (AC5 — zero problems, unchanged question counts).
- Ticket 3 AC1–AC6: confirmed by the `grep -c` check (AC1, AC2, AC4), unchanged bottom-breadcrumb/quiz-footer links (AC3, verified untouched by `git diff`), unchanged README disclaimer (AC5), and `audit_quizzes.py`'s repo-wide link check (AC6).

`git diff --stat` confirms the changeset is confined to exactly the 18 files named across the three tickets (12 lesson pages for Ticket 3, 6 quiz files for Tickets 1 and 2 combined), with no unrelated files touched. One pre-existing, unrelated repository state — a deleted `.claude/worktrees/docs-freshness-audit` entry in `git status` — predates this session and is not part of this changeset.

# Risks / Follow-ups

- **Ticket 1's Open Question resolved as stated in the ticket**: `noch-quiz.md` Q8 and Q25 were rewritten to drop their dependency on "wieder" rather than expanding `noch.md`'s one-line "wieder" aside into full teaching content. `noch.md` itself is unchanged. If the content owner would prefer teaching "wieder" earlier on the `noch` page instead, that's a different, larger change not made here.
- **Content-quality judgment calls**: every replacement distractor and every rewritten question is new instructional content (not a mechanical substitution) — plausible-but-wrong sentences, new scenarios, and new explanations. This is inherently a judgment call rather than a verifiable fact; a native-speaker or content-owner pass is worth doing before treating this as final, the same way any new lesson content would get reviewed.
- **`words/noch.md`'s own "wieder" aside** (`noch.md:112`) is unchanged and still a single sentence — it was already in scope before this ticket and remains adequate only for the reduced role the rewritten quiz questions now give it (a passing mention, not something tested).
- **Ticket 3's scope boundary**: `prepositions/` has the identical lookup-order/study-order split and disclaimer pattern but was explicitly left out of scope per the ticket; it now has a slight asymmetry with `words/` (only `words/` pages carry the per-page note). This was a stated, deliberate ticket boundary, not an oversight.
- **Pre-existing, unrelated defects** noted but not touched, per each ticket's Out of Scope: `comparisons/auch-vs-noch-quiz.md` Q1/Q26 structural errors, and the three `verbs/*-quiz.md` files with options-but-no-answer-key problems — all flagged by `audit_quizzes.py` before this session began and unrelated to `words/`.

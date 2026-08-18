# Tickets: comparisons/ directory content and structural fixes

Dependency order: **01 → (02, 03 in either order)**. 02 and 03 do not depend on 01 or on each other — they touch different files with no overlapping edits — but 01 is listed first because it is the smallest, purely mechanical fix and clears the one actual script error before the content-judgment tickets land.

---

````markdown
# Fix the Q26 markup in auch-vs-noch-quiz.md that breaks audit_quizzes.py's question parser

**Type:** Bug
**Repos:** deutsche (single static-content repository; no CLAUDE.md-defined repo roles)
**Depends on:** Nothing
**Size:** S

## User Story
As the person maintaining this repository and running `audit_quizzes.py` to check content structure, I want `comparisons/auch-vs-noch-quiz.md` to parse cleanly, so that the audit's PROBLEMS list reflects real structural defects instead of two false positives caused by the file's own markup.

## Context
`audit_quizzes.py`'s question-marker regex (`Q_RE`, `audit_quizzes.py:26`) matches any line starting with `\d+\.\s`, anywhere outside a `<details>...</details>` span (`audit_quizzes.py:33-46`). Q26 in `comparisons/auch-vs-noch-quiz.md` ("Produce it: Write B's reply in each case") presents its two production scenarios as a literal markdown ordered sub-list, both items written as `1.` (not `1.`/`2.` — GFM auto-renumbers on render, so the raw source has `1.` twice):

```
26. **Produce it:** Write B's reply in each case.

1. A: _Ich habe keine Zeit._ B agrees.
1. A: _Ich habe Zeit._ B agrees.

<details>
...
```

Both scenario lines sit before the `<details>` block opens, so `Q_RE` matches them as new top-level question markers. This (a) truncates the real Q26's parsed body one line before its `<details>` tag, so the script sees zero dropdowns for "Q26"; and (b) creates a phantom fragment numbered "1" (the digit captured from the spurious marker) that also has zero dropdowns, reported as a second, unrelated "Q1" error — the real Q1 (`auch-vs-noch-quiz.md:13-26`) is untouched and structurally fine on its own. Full root-cause trace: `.claude/work/comparisons-directory-audit/findings.md`, "Root cause" section.

Confirmed via repo-wide grep that no other file in the repository uses this literal-numbered-sub-list-outside-`<details>` pattern. Every other multi-part question in this file (e.g. Q23's "Select all that apply") uses letter-prefixed `a.`/`b.`/`c.`/`d.` options, which `Q_RE` never matches (it only matches leading digits).

## Acceptance Criteria
AC1 - Running `python3 audit_quizzes.py` from the repo root no longer includes `comparisons/auch-vs-noch-quiz.md Q26: expected exactly one solution dropdown` in the PROBLEMS section.
AC2 - Running `python3 audit_quizzes.py` from the repo root no longer includes `comparisons/auch-vs-noch-quiz.md Q1: expected exactly one solution dropdown` in the PROBLEMS section.
AC3 - `comparisons/auch-vs-noch-quiz.md`'s row in the audit's PER-FILE table still shows 26 total questions (`Qs`) and 26 solution dropdowns (`drop`) — unchanged from before the fix. The fix reformats Q26's markup only; it does not add, remove, or merge a question.
AC4 - Q26 still presents exactly its original two scenarios ("Ich habe keine Zeit." and "Ich habe Zeit.", each "B agrees") and its `<details>` block still gives exactly its original two answers (`Ich auch nicht.` then `Ich auch.`), in the same order. Visible content and correct answers are unchanged — only the markup used to list the two scenario prompts changes.
AC5 - No other question in the file (Q1-Q25) is altered.

## Implementation Plan
1. Open `comparisons/auch-vs-noch-quiz.md`, locate Q26 (currently lines 422-437).
2. Replace the two literal ordered-list lines (`1. A: _Ich habe keine Zeit._ B agrees.` / `1. A: _Ich habe Zeit._ B agrees.`) with a format that matches neither `Q_RE` (`^(?:\*\*)?(\d+)\.(?:\*\*)?\s`) nor `OPT_RE` (`^(?:\s+|-\s+)([a-d])\.\s...`, `audit_quizzes.py:27`) — bullet dashes are the simplest option and were verified against both regexes in the findings doc:
   ```
   - A: _Ich habe keine Zeit._ B agrees.
   - A: _Ich habe Zeit._ B agrees.
   ```
3. Leave the `<details>` answer block's own `1. **Ich auch nicht.**` / `2. **Ich auch.**` numbering untouched — those lines sit inside `<details>...</details>`, which `split_questions`'s `inside()` check already excludes from question-marker matching, so they are not part of the bug.
4. Run `python3 audit_quizzes.py` from the repo root; confirm both `auch-vs-noch-quiz.md` entries are gone from PROBLEMS and no new problems appear anywhere in the output.
5. Re-read the rendered Q26 (markdown preview or GitHub) to confirm the two scenarios still display as a normal numbered list despite the source using dashes, and that the content itself did not change.

## API Contract Changes
None - this ticket does not move an API surface; the repository has none.

## Postman Updates
None.

## Out of Scope
- The five content-scope issues found elsewhere in `comparisons/` (`bei-vs-mit-quiz.md`, `sehr-vs-viel-quiz.md`, `noch-vs-wieder-quiz.md`, `zu-hause-vs-nach-hause-quiz.md`, `erst-vs-schon-quiz.md`) — covered by the other two tickets in this set.
- The pre-existing, unrelated `verbs/*.md` "has options but no letter in the key" problems that `audit_quizzes.py` also reports — outside `comparisons/`, not covered by this audit.

## Testing Guidance
Run `python3 audit_quizzes.py` from the repo root before and after the change and diff the PROBLEMS section — this is the repository's only automated check for this class of defect. No other test harness exists in this repository. Manually re-read the rendered file to confirm the visible list still reads as `1.`/`2.` and nothing else shifted.

## Edge Cases
Replacement markup that starts with a letter followed by a period (e.g. `a. A: ...`) would trip `OPT_RE` instead, producing a *different* false positive ("has options but no letter in the key") — verify the chosen replacement format matches neither `Q_RE` nor `OPT_RE`, not just `Q_RE`.

## Open Questions
None - the fix, its verification, and its exact scope are fully specified by `audit_quizzes.py`'s own parsing rules.
````

````markdown
# Remove load-bearing off-topic answers from bei-vs-mit and sehr-vs-viel comparison quizzes

**Type:** Bug
**Repos:** deutsche (single static-content repository; no CLAUDE.md-defined repo roles)
**Depends on:** Nothing
**Size:** S

## User Story
As an A1 learner taking the `bei` vs. `mit` and `sehr` vs. `viel` comparison quizzes, I want every question's correct answer to depend only on the two words being compared (or on vocabulary/grammar already taught earlier in the curriculum), so that a wrong answer reflects a real gap in understanding that specific pair rather than not yet knowing an unrelated preposition (`zu`) or untaught grammar (adjective declension).

## Context
Two independent close-readings of all 14 `comparisons/` pairs (`.claude/work/comparisons-directory-audit/findings.md`) found two questions whose correct answer requires a concept entirely outside the compared pair:

- **`comparisons/bei-vs-mit-quiz.md:190-201` (Q12)** and **`:453-465` (Q27)** both require **`zu`** as the sole correct answer ("Ich gehe zu meiner Schwester"), not `bei` or `mit`. `bei-vs-mit` is Stage 5 item 3 in `START-HERE.md`; `zu` is not taught until Stage 5 item 4 (`prepositions/zu.md`) — the very next item, not yet reached at the point this quiz is taken. `bei-vs-mit.md:62` previews this as a one-line "Common Mistake" aside, but that does not make `zu` a legitimate load-bearing quiz answer this early.
- **`comparisons/sehr-vs-viel-quiz.md:167-180` (Q11)** — scenario "You are extremely hungry," correct answer "b. Ich habe großen Hunger," which requires accusative adjective declension (`groß` → `großen`). Adjective declension is not taught anywhere in this curriculum (confirmed by repo-wide search). Neither `sehr` nor `viel` is the answer at all.

**Constraint specific to Q12/Q27:** German genuinely has no `bei`/`mit` form for movement toward a person — only `zu` covers "going to somebody's place" (this is the actual grammar point `bei-vs-mit.md:62` is making). Fixing these two questions therefore cannot be a simple answer-key relabel on the existing "going to X's place" scenario; the scenario itself must change to a distinction `bei` and `mit` can actually answer (e.g. static location with `bei` vs. accompaniment with `mit`), while the underlying "there's no movement form of `bei`" teaching point either moves into a never-correct distractor or is left solely in the lesson's own aside.

## Acceptance Criteria
AC1 - `comparisons/bei-vs-mit-quiz.md` Q12's correct answer is `bei` or `mit`, not `zu` — the question's scenario tests a distinction that `bei`/`mit` can genuinely answer (not "going to somebody's place").
AC2 - `comparisons/bei-vs-mit-quiz.md` Q27's expected produced answer is a `bei`- or `mit`-based sentence, not "Ich gehe zu meiner Schwester" or any other `zu`-based sentence.
AC3 - `comparisons/sehr-vs-viel-quiz.md` Q11's correct answer is `sehr` or `viel`, not a sentence requiring accusative adjective declension.
AC4 - Running `python3 audit_quizzes.py` after the edits reports no new structural problems in either file (answer-key letters still resolve among displayed options, no duplicated option text, no new "expected exactly one solution dropdown" errors).
AC5 - `comparisons/bei-vs-mit-quiz.md`'s total question count stays 27 and `comparisons/sehr-vs-viel-quiz.md`'s stays 25 — the fix replaces each flagged question's content, it does not delete a question outright.
AC6 - Neither replacement question introduces a new off-topic load-bearing answer — each rewritten question's correct answer, checked against `bei-vs-mit.md` / `sehr-vs-viel.md`, is one of the two compared words or a taught inflected form of one of them.

## Implementation Plan
1. Open `comparisons/bei-vs-mit-quiz.md`, locate Q12 (~lines 190-201). Replace the scenario with one that has a genuine `bei`-or-`mit` answer (e.g. a static-location-vs-accompaniment contrast, matching the distinction the lesson page itself teaches) — read `bei-vs-mit.md` for the pair's own taught contrasts before drafting.
2. Locate Q27 (~lines 453-465), a "rewrite the sentence for the new situation" production prompt. Replace its scenario so the expected rewritten sentence is `bei`- or `mit`-based rather than requiring the "movement needs `zu`" answer.
3. Open `comparisons/sehr-vs-viel-quiz.md`, locate Q11 (~lines 167-180). Replace the scenario/options so the correct answer is `sehr` or `viel` — reuse this quiz's own existing option style (see e.g. the adjacent Q12/Q13, which already correctly test `sehr`/`viel` word-class rules) rather than introducing a new grammar point.
4. Run `python3 audit_quizzes.py`; confirm no new problems and that both files' question counts are unchanged (27 and 25 respectively).
5. Re-read each new/rewritten question against its own lesson page (`bei-vs-mit.md`, `sehr-vs-viel.md`) to confirm the correct answer is fully in-scope and does not accidentally introduce a different off-topic dependency.

## API Contract Changes
None - this ticket does not move an API surface; the repository has none.

## Postman Updates
None.

## Out of Scope
- The Q26 structural/parser bug in `auch-vs-noch-quiz.md` — separate ticket (01).
- The three lower-severity "untaught vocabulary in the answer, but the pair itself stays correct" issues in `noch-vs-wieder-quiz.md`, `zu-hause-vs-nach-hause-quiz.md`, and `erst-vs-schon-quiz.md` — separate ticket (03).
- `bei-vs-mit.md:62`'s existing "Common Mistake" lesson-page aside about `zu` — not touched by this ticket; whether to trim or keep it once `zu` is no longer a quiz answer is an open question below.
- Any other question in either quiz file not named above.

## Testing Guidance
Run `python3 audit_quizzes.py` from the repo root before and after the change and diff the PROBLEMS section and the two files' PER-FILE row counts. No other automated test harness exists in this repository. Manually verify each rewritten question's correct answer against the scope-purity bar established in `.claude/work/words-directory-audit/findings.md` and `.claude/work/comparisons-directory-audit/findings.md`: every load-bearing correct answer must be one of the two compared words, or a taught inflected/negated form of one of them; untaught words may still appear as never-correct distractors.

## Edge Cases
- A replacement Q12/Q27 scenario that still implicitly requires the learner to know `zu` doesn't exist yet (e.g. as a distractor option) is fine — distractors using not-yet-taught words are acceptable per the established severity bar, as long as they are never the correct answer.
- If the rewritten Q27 keeps a "rewrite this sentence" production format, the expected answer must be unambiguous — a free-text production question with a `bei`/`mit` distinction that has more than one equally valid correct phrasing should be reworded as multiple-choice, matching how the rest of this file's production questions are structured (see e.g. Q26's precedent - single unambiguous expected phrasing).

## Open Questions

**Non-blocking:**
- Should `bei-vs-mit.md:62`'s "Common Mistake" aside about `zu` be trimmed now that `zu` is no longer tested in this quiz, or left as forward-pointing context for the learner? This ticket assumes it is left as-is, since the aside is still factually true and harmless as prose; a content maintainer may prefer to trim it for tightness.
- Exact replacement scenarios/wording for Q12, Q27, and Q11 are left to the implementer's judgment within the constraints stated in AC1-AC3 and the Implementation Plan; the findings doc did not mandate specific replacement text.
````

````markdown
# Replace untaught-vocabulary answers in noch-vs-wieder, zu-hause-vs-nach-hause, and erst-vs-schon quizzes

**Type:** Bug
**Repos:** deutsche (single static-content repository; no CLAUDE.md-defined repo roles)
**Depends on:** Nothing
**Size:** M

## User Story
As an A1 learner taking the `noch` vs. `wieder`, `zu Hause` vs. `nach Hause`, and `erst` vs. `schon` comparison quizzes, I want each question's correct answer stated using only vocabulary and grammar already taught by that point in the curriculum, so that I can answer using what I've actually learned rather than being asked to recognize an untaught ordinal number, an untaught separable verb, or an untaught sense of a word I thought I already knew.

## Context
Unlike the two questions fixed in the sibling ticket (02), the three questions below keep a correct answer that genuinely belongs to the compared pair — the problem is that the *phrasing used to state* that correct answer reaches outside what's been taught by that point in the curriculum. Full detail in `.claude/work/comparisons-directory-audit/findings.md`, "Content-QA findings" section.

- **`comparisons/noch-vs-wieder-quiz.md:307-320` (Q19)** — "Nie wieder!" — correct answer "d. Kein zweites Mal" uses the ordinal "zweites Mal" (second time). No lesson in this curriculum covers ordinal numbers (confirmed by repo-wide search).
- **`comparisons/zu-hause-vs-nach-hause-quiz.md:315-330` (Q19)** — correct answer "a. Um fünf geht er los" uses the separable verb *losgehen* (split: "geht ... los"), unglossed. Separable verbs are not taught until Stage 7 (`verbs/separable-verbs.md`); this quiz is Stage 5 item 7.
- **`comparisons/erst-vs-schon-quiz.md:288-301` (Q18)** — "Erst mal einen Kaffee" tests the sequencing sense of `erst` ("first of all"). `erst-vs-schon.md` only teaches the "opinion attached to a number" sense of `erst`, never this idiomatic sequencing use. This also sits in mild tension with the same quiz's own Q19 explanation two questions later ("neither of them is ever neutral" - always attaches an opinion to a number), since Q18's usage has no number at all.

Q18 has two legitimate resolutions, and the findings explicitly left the choice to the ticket author rather than picking one: remove the question, or teach the idiom on the lesson page first so the question becomes legitimate. This ticket defaults to **removing Q18** (see Open Questions) because expanding a lesson's taught content is a larger content-design decision than a quiz-question-fix ticket should make unilaterally - a maintainer who prefers to teach the idiom instead should treat that as a superseding choice before implementation.

## Acceptance Criteria
AC1 - `comparisons/noch-vs-wieder-quiz.md` Q19's correct answer no longer relies on "zweites Mal" or any other not-yet-taught ordinal-number phrasing; the question and its distractors use only vocabulary already taught by Stage 3 item 7 (`noch-vs-wieder`'s own position in `START-HERE.md`).
AC2 - `comparisons/zu-hause-vs-nach-hause-quiz.md` Q19's correct answer no longer relies on the separable verb "losgehen"; it uses only vocabulary/grammar already taught by Stage 5 item 7.
AC3 - `comparisons/erst-vs-schon-quiz.md` Q18 is resolved by one of: (a) removed from the file, with every subsequent question renumbered so the file's questions remain contiguously numbered 1..N; or (b) kept, with `erst-vs-schon.md` gaining a short teaching note introducing the "erst mal" sequencing sense of `erst` before the quiz tests it. Whichever path is taken, `erst-vs-schon-quiz.md` no longer contains a question whose correct answer depends on an `erst` sense the linked lesson page does not teach.
AC4 - Running `python3 audit_quizzes.py` after all edits reports no new structural problems in any of the three files.
AC5 - Each file's question count matches whichever AC1-AC3 outcome was chosen: `noch-vs-wieder-quiz.md` stays 28; `zu-hause-vs-nach-hause-quiz.md` stays 26; `erst-vs-schon-quiz.md` is 25 if Q18 is removed (path a) or stays 26 if Q18 is kept and taught (path b).

## Implementation Plan
1. `comparisons/noch-vs-wieder-quiz.md` Q19 (~lines 307-320): reword the correct option so it doesn't require "zweites Mal" - a paraphrase built from already-taught vocabulary is preferred over introducing ordinal numbers (e.g. a phrasing along the lines of "Das mache ich nie mehr" is one direction; exact wording is the implementer's call).
2. `comparisons/zu-hause-vs-nach-hause-quiz.md` Q19 (~lines 315-330): reword the correct option so it doesn't use "losgehen" - restructure around already-taught `zu Hause`/`nach Hause` vocabulary (e.g. a phrasing along the lines of "Um fünf ist er noch nicht zu Hause" is one direction; exact wording is the implementer's call).
3. `comparisons/erst-vs-schon-quiz.md` Q18 (~lines 288-301): implement the default resolution (AC3 path a) - remove the question and renumber Q19 through the file's last question down by one. Before renumbering, grep the repository for any other file referencing this quiz's question numbers by position (e.g. "erst-vs-schon-quiz.md Q19") so nothing goes stale.
4. Run `python3 audit_quizzes.py`; confirm no new problems and that each file's question count matches the chosen resolution (per AC5).
5. Re-read each touched question against its own lesson page to confirm the new/adjusted correct answer is fully in-scope for that point in the curriculum.

## API Contract Changes
None - this ticket does not move an API surface; the repository has none.

## Postman Updates
None.

## Out of Scope
- The Q26 structural/parser bug in `auch-vs-noch-quiz.md` - ticket 01.
- The two load-bearing off-topic-answer scope leaks in `bei-vs-mit-quiz.md` and `sehr-vs-viel-quiz.md` - ticket 02.
- Expanding `erst-vs-schon.md`'s taught content (AC3 path b) is an acceptable alternative resolution but is not the default this ticket implements - see Open Questions.
- Any other question in the three touched files not named above.

## Testing Guidance
Run `python3 audit_quizzes.py` from the repo root before and after the change and diff the PROBLEMS section and each file's PER-FILE row count. No other automated test harness exists in this repository. Manually verify each rewritten/renumbered question against the scope-purity and level-appropriateness bar established in `.claude/work/comparisons-directory-audit/findings.md`.

## Edge Cases
- If Q18 is removed and subsequent questions in `erst-vs-schon-quiz.md` are renumbered, verify no other file in the repository (e.g. a review/cumulative page, or `CONCEPT-INDEX.md`) references that quiz's question numbers by position - a repo-wide grep for the file's name is sufficient to confirm this before renumbering.
- The Q19 rewrite in `erst-vs-schon-quiz.md`'s neighbor question explicitly asserts "neither of them is ever neutral" about `erst`/`schon` - if Q18 is kept and the lesson is taught (path b), reconfirm that addition doesn't contradict that explanation; if Q18 is removed (path a), this tension disappears on its own.

## Open Questions

**Non-blocking:**
- `erst-vs-schon-quiz.md` Q18's resolution direction (remove vs. teach) - this ticket defaults to removal (AC3 path a, Implementation Plan step 3) per the reasoning in Context. A content maintainer who prefers to teach the "erst mal" idiom on the lesson page instead should say so before implementation; either path satisfies AC3.
- Exact replacement wording for `noch-vs-wieder-quiz.md` Q19 and `zu-hause-vs-nach-hause-quiz.md` Q19 is left to the implementer; the findings doc offered directional examples, not mandated text.
````

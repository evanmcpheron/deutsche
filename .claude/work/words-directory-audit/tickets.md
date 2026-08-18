# Tickets: words-directory-audit

Dependency order: **Ticket 1 → independent. Ticket 2 → independent. Ticket 3 → independent.** None of the three depends on either of the others — they touch disjoint files (Ticket 1: `noch-quiz.md`, `nicht-quiz.md`, `kein-quiz.md`, `auch-quiz.md`; Ticket 2: `immer-quiz.md`, `wieder-quiz.md`; Ticket 3: all 12 `words/*.md` lesson pages' breadcrumb line) and can ship in any order or in parallel. Ticket 1 is the only one with a required, learner-facing correctness defect; Tickets 2 and 3 are optional polish per the research findings.

````markdown
# Fix quiz questions in words/ that require untaught vocabulary or grammar

**Type:** Bug
**Repos:** deutsche (this repository)
**Depends on:** Nothing
**Size:** M

## User Story
As a learner working through the words/ Stage 2 curriculum (`nicht`, `kein`, `auch`, `noch`), I want each word's quiz to test only the target word plus vocabulary and grammar I've already been taught, so that a wrong answer always means I misunderstood the word being quizzed — never that I hit an untaught grammar wall the lesson never warned me about.

## Context
Findings in `.claude/work/words-directory-audit/findings.md` ("Affected surface area") identify four quiz files where questions depend on material the lesson page never taught and that hasn't appeared earlier in the curriculum (per `START-HERE.md`'s Stage 2/3 order: nicht → kein → [nicht-vs-kein] → auch → nur → noch → [auch-vs-noch] → schon):

- **`words/noch-quiz.md`** — the most significant case. Q1 (`:15`, explanation `:22`), Q3 (`:53`, `:60`), and Q9 (`:145`, `:154`) use **"nicht mehr"** as a distractor/explanation concept, but `noch.md` never teaches it — it's reachable only via an unglossed link in Related Pages (`noch.md:137`), and "nicht mehr" is `START-HERE.md`'s own separate, later Stage-3 lesson. **Q14 (`:221-238`) requires "Nein, nicht mehr." as the sole correct answer** — this is the one load-bearing violation found across the whole `words/` directory: a learner cannot answer correctly without knowledge the lesson never gave them. Q8 (`:128-141`) and Q25 (`:414-426`) similarly require productive command of **wieder** — Q8 as the required answer to a multi-step scenario, Q25 as free production of "Er ist wieder krank." — well beyond the single aside `noch.md:112` gives it ("wieder means it stopped and started again").
- **`words/nicht-quiz.md`** Q13 (`:195`) requires understanding **sondern** ("but rather"), never introduced anywhere in the curriculum up to this point.
- **`words/kein-quiz.md`** Q14 (`:226`, "Wir haben kein Brot mehr.") tests the untaught **kein … mehr** pattern (only "nicht mehr" is taught, on a different page, as a fixed chunk). Q15 (`:241`, "Wir haben noch keine Tickets.") treats **noch** as independently productive vocabulary, though at this point it has only appeared buried inside the frozen chunk "noch nicht."
- **`words/auch-quiz.md`** Q4 (`:68-81`) — the four answer options are entirely unglossed German using untaught grammar: a relative clause ("das Einzige, **was** ich trinke") in one option and the separable verb **dazukommen** ("kommt … dazu") in another. A learner who fully understands `auch` could still fail this question purely from being unable to parse the distractors.

The repository's only automated check, `audit_quizzes.py` (repo root), verifies quiz *structure* (answer-key letters resolve, no duplicate options, no broken links, question counts) but does not and cannot check semantic scope — it passes on all four files today despite these issues. It must keep passing after this fix.

## Acceptance Criteria
AC1 - `words/noch-quiz.md` Q14's correct answer and answer key no longer require the "nicht mehr" pattern; the question, its options, and its explanation use only vocabulary/grammar taught in `noch.md` or in an earlier lesson (`nicht`, `kein`, `auch`, `nur`).
AC2 - `words/noch-quiz.md` Q1, Q3, and Q9 no longer contain "nicht mehr" in any option, correct answer, or explanation text.
AC3 - `words/noch-quiz.md` Q8 and Q25 no longer require the learner to productively know "wieder" beyond what `noch.md` itself teaches at the point this quiz is taken (see Open Questions for the two acceptable ways to close this gap).
AC4 - `words/nicht-quiz.md` Q13 no longer requires understanding the conjunction "sondern"; the replacement question tests a contrast `nicht.md` actually teaches on the page.
AC5 - `words/kein-quiz.md` Q14 no longer tests the "kein … mehr" pattern.
AC6 - `words/kein-quiz.md` Q15 no longer requires treating "noch" as independently productive vocabulary.
AC7 - `words/auch-quiz.md` Q4's four answer options contain no relative clause and no separable-verb construction ("dazukommen" or equivalent); every option is parseable using only vocabulary/grammar already taught by this point in the curriculum.
AC8 - Each edited question still has exactly one `<details>` solution block containing an `**Answer:**` line whose letter(s) exist among that question's displayed `a.`/`b.`/`c.`/`d.` options.
AC9 - Running `python3 audit_quizzes.py` from the repo root after all edits reports zero problems for `words/noch-quiz.md`, `words/nicht-quiz.md`, `words/kein-quiz.md`, and `words/auch-quiz.md`, and the same or greater question count in each file as before the edit.
AC10 - No unrelated question in any of the four files changes meaning, correct answer, or wording.

## Implementation Plan
1. Re-read `words/noch.md`, `words/nicht.md`, `words/kein.md`, and `words/auch.md` in full immediately before editing, to have the exact taught vocabulary/grammar set for each in hand (this ticket's ACs are defined relative to what each lesson page actually teaches, not a fixed list).
2. Resolve the Open Question below (noch.md's wieder aside) before touching Q8/Q25 in `noch-quiz.md` — pick the "rewrite the questions" default unless the content owner says otherwise, and note the choice in the PR description.
3. Edit `words/noch-quiz.md`:
   - Q1 (`:11-24`), Q3 (`:49-64`), Q9 (`:141-158`): replace the "nicht mehr" distractor/explanation with an in-scope alternative (drawn from nicht/kein/auch/nur, or noch's own "still/yet/another" senses).
   - Q14 (`:214-238`): replace the correct answer and rework the scenario so the answer is reachable from taught material only.
   - Q8 (`:122-141`) and Q25 (`:405-426`): per the Open Question resolution, either rewrite to not depend on "wieder," or (if the content owner prefers) expand `noch.md:112`'s aside into a real one-paragraph explanation first, then leave these questions as-is.
   - Match the existing file's format exactly for every edit: numbered stem, `a.`–`d.` options with trailing backslash line-continuations, one `<details><summary>Show solution</summary>` block per question with `**Answer:**` and `**Explanation:**` lines — use an untouched question in the same file (e.g. Q2 or Q4) as the formatting reference.
4. Edit `words/nicht-quiz.md` Q13 (`:195-208`): replace the *sondern*-based contrast with a contrast the page teaches directly (e.g., position of *nicht* before an adjective vs. before a full clause — both already on `nicht.md`).
5. Edit `words/kein-quiz.md` Q14 (`:214-227`) and Q15 (`:229-248`): replace the "kein Brot mehr" and "noch keine Tickets" scenarios with kein-only noun-negation scenarios using vocabulary already used elsewhere in the file.
6. Edit `words/auch-quiz.md` Q4 (`:55-84`): rewrite the four options in vocabulary/grammar already established by this point (simple main clauses only), preserving the question's original intent — distinguishing correct from incorrect placement/use of *auch*.
7. Run `python3 audit_quizzes.py` from the repo root and confirm the report shows zero problems for the four touched files and unchanged-or-higher question counts.

## API Contract Changes
None - this ticket does not move an API surface. This repository has no API; it is a static markdown content set.

## Postman Updates
None.

## Out of Scope
- `words/schon.md`, `nur.md`, `sehr.md`, `immer.md`, `wieder.md`, `vielleicht.md`, `erst.md`, `gerade.md` and their quizzes — confirmed clean by the research audit, not touched here.
- `comparisons/auch-vs-noch-quiz.md` Q1 and Q26's pre-existing "expected exactly one solution dropdown" structural errors (surfaced by `audit_quizzes.py`, unrelated defect class) — belongs to a `comparisons/` audit, not this ticket.
- The non-load-bearing premature-vocabulary distractors in `words/immer-quiz.md` and `words/wieder-quiz.md` — covered by a separate ticket.
- The lesson-page lookup-order vs. study-order navigation signposting — covered by a separate ticket.
- `words/vielleicht.md:36`'s minor translation looseness ("Maybe we should take a taxi") — findings flagged it as defensible and non-blocking; not actioned by any ticket.
- Any change to `noch.md`, `nicht.md`, `kein.md`, or `auch.md` lesson content itself, except the possible one-paragraph `wieder` expansion in `noch.md` contemplated by the Open Question below — the fix is scoped to quiz files first.

## Testing Guidance
- **Content review** - read each edited question in context (question, options, explanation) and confirm it is answerable using only what `noch.md`/`nicht.md`/`kein.md`/`auch.md` teach plus earlier lessons in the Stage 2 order.
- **Structural** - `python3 audit_quizzes.py` from the repo root is the only automated check in this repository; it must report zero problems for the four touched files (broken links, answer-key/option mismatches, duplicate option text, answer-run/cycle patterns, question counts). This is the project's existing regression check - there is no other test suite.
- **Regression** - every question NOT listed in the Acceptance Criteria must remain byte-for-byte unchanged; diff review should show edits confined to the specific line ranges named in the Implementation Plan.

## Edge Cases
- Replacement distractors must remain plausible wrong answers, not so obviously incorrect that the question becomes trivial - this is a content-quality judgment call for the implementer, not a mechanical substitution.
- After editing `noch-quiz.md`, re-check that no new run of three identical answer letters in a row, or an a-b-c-d/d-c-b-a cycling pattern, was introduced across the file - `audit_quizzes.py` checks this automatically (AC9 covers it).
- `noch-quiz.md` Q14's new correct answer must be a genuinely different, in-scope answer - not a relabeled version of the same "nicht mehr" answer under a different letter.

## Open Questions

**Non-blocking:**
- For `noch-quiz.md` Q8 and Q25 (the "wieder" dependency): should `noch.md:112`'s one-sentence aside be expanded into real teaching content (which would make the existing questions legitimate as-is), or should the two questions instead be rewritten to drop "wieder" entirely? Both resolve the gap. Default assumption for this ticket: rewrite the questions and leave `noch.md` untouched, since that keeps the diff scoped to quiz files and avoids widening what one lesson page is responsible for teaching. If the content owner prefers expanding `noch.md` instead, Q8/Q25 can be left as-is once that expansion lands. Source: `.claude/work/words-directory-audit/findings.md`, Open Questions section.
````

````markdown
# Remove premature-vocabulary distractors from immer-quiz.md and wieder-quiz.md

**Type:** Chore
**Repos:** deutsche (this repository)
**Depends on:** Nothing
**Size:** S

## User Story
As a learner in Stage 3 (`immer`, `wieder`), I want quiz distractors to use only vocabulary I've already learned, so an unfamiliar word in a *wrong* answer doesn't cost me time or confidence when it was never the point of the question.

## Context
Findings in `.claude/work/words-directory-audit/findings.md` identify two quizzes that use the *next* not-yet-taught word as a never-correct distractor:

- `words/immer-quiz.md` uses standalone **"wieder"** (not the taught chunk "immer wieder") as a distractor at `:13`, `:45`, `:188`, and `:222` — before `wieder.md` (the next lesson in curriculum order) has been taught.
- `words/wieder-quiz.md` uses **"erst"** as a distractor at `:82` and `:201` — before `erst.md` (a later Stage-3 lesson) has been taught. The `:82` distractor, "Du bist erst zu spät.", is additionally flagged as mildly unidiomatic on its own terms, independent of the scope issue.

Both are explicitly non-load-bearing: in every case the untaught word is never the correct answer, so a learner can answer every question correctly without knowing it. This is why the research findings rank it as optional polish rather than a required fix (contrast with the load-bearing `words/noch-quiz.md` Q14 case handled in the companion ticket "Fix quiz questions in words/ that require untaught vocabulary or grammar").

## Acceptance Criteria
AC1 - `words/immer-quiz.md` contains no standalone "wieder" in any option or distractor at the four cited locations; each is replaced with a distractor using vocabulary taught by the time a learner reaches `immer` (nicht, kein, auch, nur, noch, schon, and the patterns noch nicht / nicht mehr / immer noch).
AC2 - `words/wieder-quiz.md` contains no "erst" in any option or distractor at the two cited locations.
AC3 - The `words/wieder-quiz.md` replacement for the `:82` distractor is idiomatic standard German.
AC4 - No question's correct answer or explanation changes as a result of these edits - only the non-correct distractor text changes.
AC5 - Running `python3 audit_quizzes.py` from the repo root after all edits reports zero problems for `words/immer-quiz.md` and `words/wieder-quiz.md`, with the same question count as before.

## Implementation Plan
1. Re-read `words/immer.md` and `words/wieder.md` to confirm the exact vocabulary available at each point in the curriculum.
2. Edit `words/immer-quiz.md` at lines `:13`, `:45`, `:188`, `:222`, replacing "wieder" with an in-scope distractor of comparable plausibility, matching the existing option format.
3. Edit `words/wieder-quiz.md` at lines `:82` and `:201`, replacing "erst" with an in-scope, idiomatic distractor.
4. Run `python3 audit_quizzes.py` from the repo root and confirm zero new problems for both files.

## API Contract Changes
None - this ticket does not move an API surface.

## Postman Updates
None.

## Out of Scope
- The required fixes in `words/noch-quiz.md`, `words/nicht-quiz.md`, `words/kein-quiz.md`, `words/auch-quiz.md` - covered by the companion ticket "Fix quiz questions in words/ that require untaught vocabulary or grammar."
- Any change to `words/immer.md` or `words/wieder.md` lesson content - only the two quiz files change.
- The lesson-page lookup-order vs. study-order navigation signposting - covered by a separate ticket.

## Testing Guidance
- **Content review** - confirm each replacement distractor is plausible but clearly wrong, and uses only previously-taught vocabulary.
- **Structural** - `python3 audit_quizzes.py` is this repository's only automated check; it must report zero problems for both touched files after the edit.
- **Regression** - all correct answers and explanations remain unchanged; diff review should show edits confined to the six cited distractor lines.

## Edge Cases
- The replacement for `wieder-quiz.md:82` must both drop "erst" and read as natural German on its own - the current line has two independent problems (scope and idiom), and fixing only one is not sufficient for AC2/AC3.

## Open Questions
None.
````

````markdown
# Signpost lookup order vs. study order on words/ lesson-page navigation

**Type:** Chore
**Repos:** deutsche (this repository)
**Depends on:** Nothing
**Size:** S

## User Story
As a learner browsing `words/` via a lesson page's own top/bottom `Next →` link (rather than going through its quiz first), I want a clear signal that this particular link is a lookup shortcut and not the recommended study path, so I don't unknowingly drift out of the curriculum's intended order.

## Context
Findings in `.claude/work/words-directory-audit/findings.md` ("Root cause of the reported navigation 'bug'") establish that `words/` has two intentionally different navigation systems, both verified correct and self-consistent:

1. The breadcrumb at the top and bottom of every `words/<word>.md` page (`← prev | Category Index | Next → next | ↑ Master Index`) implements the **lookup order** - an exact match to the table in `words/README.md:11-22`, which is itself explicitly disclaimed: *"This page lists words in one lookup order, not the recommended study order"* (`words/README.md:7`). The identical pattern exists in `prepositions/` (`prepositions/README.md:5`).
2. The `Next →` link at the *bottom of every quiz page* (`words/<word>-quiz.md`) implements the **true study order** and was traced end-to-end against `START-HERE.md` with no errors found.

Both systems work as designed. The one real gap: the lookup-order disclaimer lives only on `words/README.md`, not on the individual lesson pages themselves. A learner who never visits the README and instead clicks a lesson page's own "Next →" before taking its quiz will silently leave the recommended path with no warning on the page they're looking at - for example, going from `auch.md` straight to `noch.md`, skipping `nur` (per `words/README.md`'s lookup table, but not per `START-HERE.md`'s Stage 2 order, which places `nur` immediately after `auch`).

## Acceptance Criteria
AC1 - Each of the 12 `words/*.md` lesson pages (`nicht`, `kein`, `auch`, `noch`, `schon`, `nur`, `sehr`, `immer`, `wieder`, `vielleicht`, `erst`, `gerade`) carries a short, consistent note stating that its `Next`/`←` breadcrumb follows the lookup order documented in `words/README.md`, not the recommended study order, and points the reader to `START-HERE.md` for the study path.
AC2 - The note's wording is identical across all 12 pages except for the file-relative link paths, so the convention reads as deliberate and repo-wide rather than page-specific.
AC3 - The existing quiz-footer `Next →` links (the true study-order path, verified correct by the prior research) are unchanged by this ticket.
AC4 - The three particle-stub pages (`words/doch.md`, `words/mal.md`, `words/eigentlich.md`) are NOT given this note, since they have no `Next` breadcrumb and do not participate in the lookup-order chain.
AC5 - `words/README.md`'s existing disclaimer sentence (`:7`) is unchanged, or if adjusted to cross-reference the new per-page note, does not contradict it.
AC6 - Running `python3 audit_quizzes.py` from the repo root after all edits reports zero new broken-link problems across the repository.

## Implementation Plan
1. Decide one exact wording for the note (a single sentence, reused verbatim across all 12 pages) - e.g. an italicized line directly under the breadcrumb: *"(This is the lookup order - see [Start Here](../START-HERE.md) for the recommended study path.)"* This is a one-time content/format decision; apply it identically everywhere rather than varying phrasing per page.
2. Add the note to the top breadcrumb (verified at line 8 in every one of the 12 files) of each `words/<word>.md` file. Decide whether the bottom breadcrumb also needs it or whether the top instance is sufficient, since that is the one a browsing learner encounters first - implementer's call, state the choice in the PR description.
3. Do not add the note to `words/doch.md`, `words/mal.md`, or `words/eigentlich.md` (per AC4).
4. Confirm `words/README.md:7`'s existing disclaimer still reads correctly alongside the new per-page notes; adjust only if it becomes redundant or awkward, not required.
5. Run `python3 audit_quizzes.py` from the repo root and confirm no new broken-link problems anywhere in the repository (the note's added links must resolve).

## API Contract Changes
None - this ticket does not move an API surface.

## Postman Updates
None.

## Out of Scope
- Applying the same treatment to `prepositions/`, which has an identical lookup-order/study-order split and the same README-only disclaimer - findings raised this as a non-blocking open question but it is not scoped into this ticket; a follow-up ticket should cover it if wanted.
- Any change to the quiz-footer study-order links.
- Any change to `words/README.md`'s lookup table itself (word order, tiers, goals).
- Any change to the three particle-stub pages' content.

## Testing Guidance
- **Content review** - open all 12 edited pages and confirm the note reads identically (modulo link paths) and does not clash visually or textually with the existing breadcrumb.
- **Structural** - `python3 audit_quizzes.py` is this repository's only automated check; run it to confirm no broken links were introduced by the new note's links to `words/README.md` and `START-HERE.md`.

## Edge Cases
- `gerade.md` is the last word in the lookup chain and has no `Next →` in its breadcrumb (only `← erst`) - the note should still explain the `←` side of the breadcrumb is lookup order, not omit the note just because there's no forward link on this page.
- `nicht.md` is the first word in the lookup chain and has no `←` in its breadcrumb (only `Next →`) - same treatment, note still applies to the forward link.

## Open Questions

**Non-blocking:**
- Should `prepositions/` receive the identical treatment for consistency, given it has the exact same lookup-order/study-order split and disclaimer pattern? Left out of this ticket's scope per the findings' own framing ("a content-repo maintainer's call"); default assumption is no, unless the content owner asks for a follow-up ticket. Source: `.claude/work/words-directory-audit/findings.md`, Open Questions section.
````

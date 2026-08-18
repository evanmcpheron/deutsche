# Research: Content and navigation audit of the `words/` directory

## Request
Do a thorough audit of everything in `words/` (12 word lessons + 10 quizzes + 3 particle-stub pages): is the German information correct, are quiz questions stated with the right amount/level of German, do quizzes test only their own topic, do "next" links follow the correct study order, and does the last word in the section point to the correct next section. Classification: **refactor/content-QA** (no code — this is a markdown A1-German curriculum repository, not a software repo; there is no `CLAUDE.md`, confirmed absent at repo root).

## Summary
German content is accurate throughout — across all 12 lesson pages and 3 particle stubs, zero grammar, case, article, or mistranslation errors were found by four independent close readings. The real-world "study order" navigation (the `Next →` link at the bottom of every quiz page) is also correct end-to-end, including the critical case: `gerade` is the last word in the directory and its quiz correctly forwards to `patterns/noch-ein.md`, the true next step in Stage 4. The one navigation oddity two reviewers flagged as a "bug" — the `Next`/`←` breadcrumb at the *top* of each lesson page not matching study order — is not a defect; it's a second, deliberately different navigation layer (alphabetical/tier "lookup order") that is externally documented in `words/README.md` and used identically in `prepositions/`, just not re-stated on the individual word pages. The genuine, fixable problems are quiz **scope leaks**: a handful of questions (concentrated in `noch-quiz.md`, plus one each in `nicht-quiz.md`, `kein-quiz.md`, `auch-quiz.md`) require the learner to know grammar or vocabulary that the lesson page never taught and that hasn't appeared earlier in the curriculum, most seriously in `noch-quiz.md` where "nicht mehr" is the *required correct answer* to a question despite never being taught on that page.

## Current behavior

### Structure
`words/` holds 12 lesson+quiz pairs (`nicht`, `kein`, `auch`, `noch`, `schon`, `nur`, `sehr`, `immer`, `wieder`, `vielleicht`, `erst`, `gerade`) plus 3 stub pages (`doch`, `mal`, `eigentlich`) that defer to `particles/` under an explicit placement rule stated in `words/README.md:26-36`. Every lesson page carries a `Type/Level/Priority/Knowledge goal` header block; these were cross-checked against `words/README.md` and `MASTER_INDEX.md` and match exactly for all 12 words (Tier 1/Active for nicht→vielleicht, Tier 2/Recognition First for erst and gerade) — no metadata drift.

### Two independent navigation systems (both verified, both self-consistent)
1. **Lesson-page breadcrumb** (`← prev | Category Index | Next → next | ↑ Master Index`, appearing at both top and bottom of each `words/<word>.md`): traced by direct grep across all 12 files — it forms one unbroken chain `nicht → kein → auch → noch → schon → nur → sehr → immer → wieder → vielleicht → erst → gerade`, which is an exact match, word-for-word, to the table in `words/README.md:11-22`. `words/README.md:7` explicitly says: *"This page lists words in one lookup order, not the recommended study order."* The identical pattern (breadcrumb chain = README table = explicitly-disclaimed lookup order) was confirmed in `prepositions/` too (`mit→zu→nach→bei→aus→von→für→um→in→auf→an`, matching `prepositions/README.md:5`'s identical disclaimer). This is a repo-wide convention, not a words-specific inconsistency.
2. **Quiz-footer "Next →" link** (bottom of every `words/<word>-quiz.md`): this is the *true* study-order pointer and was traced end-to-end against `START-HERE.md`'s Stage 2–4 sequence:
   - `nicht-quiz.md` → `kein.md` ✓ → `kein-quiz.md` → `comparisons/nicht-vs-kein.md` ✓ → `nicht-vs-kein-quiz.md` → `auch.md` ✓ → `auch-quiz.md` → `nur.md` ✓ → `nur-quiz.md` → `noch.md` ✓ → `noch-quiz.md` → `comparisons/auch-vs-noch.md` ✓ → `auch-vs-noch-quiz.md` → `schon.md` ✓ → `schon-quiz.md` → `comparisons/noch-vs-schon.md` ✓ → (Stage 2 checkpoint)
   - `immer-quiz.md` → `patterns/immer-noch.md` ✓ → `wieder-quiz.md` → `comparisons/noch-vs-wieder.md` ✓ → `erst-quiz.md` → `comparisons/erst-vs-schon.md` ✓ → `erst-vs-schon-quiz.md` → `sehr.md` ✓ (Stage 3 → Stage 4 boundary)
   - `sehr-quiz.md` → `patterns/ein-bisschen.md` ✓ → … → `vielleicht-quiz.md` → `gerade.md` ✓ → `gerade-quiz.md` → **`patterns/noch-ein.md`** ✓ — the last word in the directory correctly forwards into the next Stage 4 item.
   Every link in this chain is correct. No errors found.

## Root cause of the reported navigation "bug" (not a defect)
Two of the four sub-audits, working from `START-HERE.md` order alone, flagged the lesson-page breadcrumbs (item 1 above) as broken because e.g. `vielleicht.md` points `← wieder | Next → erst`, which looks wrong next to the real curriculum (where `erst` precedes `vielleicht` by a full stage). This is explained by the two-navigation-system design above — the breadcrumb isn't tracking curriculum position at all, it's tracking the disclaimed lookup table. The only legitimate residual concern: **the disclaimer lives only on `words/README.md`**, not on the individual lesson pages, so a learner who never visits the README and instead clicks a lesson page's top "Next →" before doing its quiz will silently drift off the recommended path (e.g., from `auch` straight to `noch`, skipping `nur`). This is a documentation/signposting gap, not a broken link.

## Affected surface area (confirmed quiz scope leaks)

- **`words/noch-quiz.md`** (most significant):
  - Q1 (`:15`, explanation `:22`), Q3 (`:53`, `:60`), Q9 (`:145`, `:154`) all use **"nicht mehr"** as a distractor/explanation concept. `noch.md` never teaches this pattern — it's only reachable via an unglossed link in Related Pages (`noch.md:137`). Per `START-HERE.md`, "nicht mehr" is a separate, later Stage-3 lesson.
  - **Q14 (`:221-238`) requires "Nein, nicht mehr." as the sole correct answer** — the learner must actively produce a pattern this lesson never taught. This is the one load-bearing scope violation found in the whole directory.
  - Q8 (`:128-141`, "wieder" as required answer to a multi-step relapse scenario) and Q25 (`:414-426`, free production of "Er ist wieder krank.") both lean on **wieder** well beyond the single-sentence aside `noch.md:112` gives it. `noch vs. wieder` is `START-HERE.md`'s own dedicated later lesson.
- **`words/nicht-quiz.md`** Q13 (`:195`) requires understanding **sondern** ("but rather"), never introduced anywhere in the curriculum up to this point.
- **`words/kein-quiz.md`**: Q15 (`:241`, "Wir haben noch keine Tickets.") treats **noch** as freely productive vocabulary, though it has only appeared buried in the frozen chunk "noch nicht," never as an independent word, by this point in the curriculum. Q14 (`:226`, "Wir haben kein Brot mehr.") tests the **kein … mehr** pattern, which `kein.md` never teaches (only "nicht mehr," on a different page, as a fixed chunk) — softer than Q15 but still untaught.
- **`words/auch-quiz.md`** Q4 (`:68-81`): the four answer options are entirely unglossed German using untaught grammar — a relative clause ("das Einzige, **was** ich trinke") and the separable verb **dazukommen** ("kommt … dazu") — meaning a learner who fully understands `auch` could still fail purely from being unable to parse the distractors. This is a "too much unglossed German" problem more than a strict topic-scope problem.
- **`words/immer-quiz.md`** (`:13, 45, 188, 222`) and **`words/wieder-quiz.md`** (`:82, 201`): both use the *not-yet-taught* neighboring word (wieder before it's taught; erst before it's taught) as **never-correct distractors**. Non-load-bearing — a learner can answer every question without knowing the untaught word — but still puts unglossed vocabulary in front of them. `wieder-quiz.md:82`'s distractor ("Du bist erst zu spät.") is also mildly unidiomatic on its own terms.

No scope leaks, and no German-correctness errors, were found in `schon.md`/`schon-quiz.md`, `nur.md`/`nur-quiz.md`, `sehr.md`/`sehr-quiz.md`, `immer.md`, `wieder.md`, `vielleicht.md`/`vielleicht-quiz.md`, `erst.md`/`erst-quiz.md`, or `gerade.md`/`gerade-quiz.md`. `erst-quiz.md` and `gerade-quiz.md` were also checked for goal-calibration (Recognition First → quizzes should favor comprehension over production) and both pass: each has essentially one free-production item and is otherwise situational/interpretive multiple choice, correctly lighter than the Active-word quizzes.

One trivial, non-blocking translation looseness: `vielleicht.md:36` glosses "Vielleicht nehmen wir ein Taxi." as "Maybe we should take a taxi" — the German carries no modal, so "Maybe we'll take a taxi" is tighter, though the softer gloss is defensible given the lesson's own framing of the sentence as a suggestion.

## Existing patterns to follow
This repo already has a structural linter, `audit_quizzes.py` (repo root), which was run as part of this audit. It confirmed: no broken links, no answer-key/option mismatches, no duplicate option text, no cycling/run-of-three answer patterns, and correct question counts, for every file in `words/`. It does **not** check semantic scope creep or German correctness — that gap is what this audit fills. (It did surface two unrelated pre-existing structural problems in `comparisons/auch-vs-noch-quiz.md` — "expected exactly one solution dropdown" at Q1 and Q26 — which sit just outside `words/` but are worth the next reviewer's attention since that comparison page is `noch-quiz.md`'s and `auch-quiz.md`'s own designated "next" destination.)

## API contract impact
None — this is a static content repository, no API surface.

## Data and migration impact
None.

## Test coverage today
`audit_quizzes.py` is the only automated check and covers structural integrity, not content correctness or scope. It should stay green after any fix described below; none of the suggested fixes touch answer-key letters or option counts in a way that would need script changes.

## Suggested ticket slicing

- **01-fix-noch-quiz-scope-leaks** — Rewrite `words/noch-quiz.md` Q1, Q3, Q8, Q9, Q14, and Q25 so none of them require "nicht mehr" or "wieder" beyond the one-line aside `noch.md` already gives; Q14 in particular needs a new correct answer that stays within `noch.md`'s taught material. `Depends on: nothing`. `Repos: this repo (words/)`. Split out because it's the only load-bearing violation and touches one file.
- **02-fix-nicht-kein-auch-quiz-scope-leaks** — Fix `words/nicht-quiz.md` Q13 (drop or replace the *sondern* contrast), `words/kein-quiz.md` Q14/Q15 (replace "kein … mehr" and "noch keine" with in-scope material), and `words/auch-quiz.md` Q4 (simplify the answer options out of unglossed relative-clause/separable-verb German). `Depends on: nothing`. `Repos: this repo (words/)`. Grouped because each is a single, independent question-level fix in a different file, but same severity tier and reviewable together.
- **03-trim-premature-distractors** — Replace the non-load-bearing "wieder" distractors in `words/immer-quiz.md` and "erst" distractors in `words/wieder-quiz.md` with in-scope alternatives, and fix the mildly unidiomatic "Du bist erst zu spät." distractor. `Depends on: nothing`. `Repos: this repo (words/)`. Lowest priority — cosmetic/polish, not load-bearing.
- **04-signpost-lookup-order-on-word-pages** — Add a short note to the shared breadcrumb pattern (or a one-line footnote) on each `words/*.md` page clarifying that its top/bottom `Next →` is the lookup order, not the study path, mirroring the disclaimer already on `words/README.md`. Consider whether `prepositions/` wants the identical treatment for consistency. `Depends on: nothing`. `Repos: this repo`. Optional documentation-only ticket; not a functional bug.

## Constraints and risks
None beyond the usual content-review risk: several of the flagged fixes are judgment calls (e.g., is `noch.md`'s existing one-line "wieder" aside meant to license testing it, or should it be removed from the quiz entirely, or should the lesson page instead be expanded to properly teach it?). The ticket author should pick one direction rather than leaving it ambiguous.

## Open questions

**Non-blocking:**
- Should `noch.md`'s one-sentence "wieder" aside (`noch.md:112`) be expanded into real teaching content (making the current wieder-heavy quiz questions legitimate) or should the quiz be trimmed instead? Either resolves the leak; the ticket author should choose. Source: compare `noch.md:112` against `noch-quiz.md` Q8/Q25.
- Should the lookup-order vs. study-order distinction be signposted on individual word pages, or is the README-level disclaimer considered sufficient given it's a repo-wide convention already in `prepositions/`? A content-repo maintainer's call.

None blocking — every question above has a stated default (leave as-is) that doesn't stop other work.

## Confidence
**High** on German correctness (four independent close readings, zero errors found) and on navigation-order verification (I personally traced the full quiz-footer chain against `START-HERE.md` and independently confirmed the lookup-order breadcrumb pattern by direct grep across `words/` and `prepositions/`, which resolved what would otherwise have been two false "bug" reports). **Medium** on the completeness of the scope-leak list — four sub-agents each read their 3-word slice closely, but a different reviewer applying a stricter or looser bar for "already covered by then" could surface a few more borderline cases (e.g., how much incidental use of not-yet-taught vocabulary is acceptable in flavor text vs. in a load-bearing answer).

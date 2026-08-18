## Summary

Every page reachable from `README.md` now carries a `Next →` footer link that reproduces the
single, curated sequence defined by `START-HERE.md` — starting at `reference/how-to-read-
without-translating.md` and following only `Next →` links lands on all 131 spine pages, in the
exact stage order `START-HERE.md` prescribes, with no dead end and no detour into a category's
internal alphabetical order. Previously, the 75 quiz pages had no forward link at all, and 68
other lesson/comparison/reference/review pages had a `Next →` link that silently followed each
category's own `README.md` table order instead — the two most visible consequences being that
finishing any quiz stranded the reader, and clicking "Next" from `reference/active-vs-passive-
vocabulary.md` (a "Before Stage 1" page) jumped straight to a Stage 5 reference page, skipping
Stages 1 through 4 entirely.

Three tickets from `.claude/work/guided-learning-path/tickets.md` are implemented:

1. **Make every page's "Next →" link follow the guided learning path in order** — the core fix.
2. **Label category index pages as a lookup index, not the reading order** — a one-line note on
   each of the 9 category `README.md` files now that they no longer double as the click-through
   order.
3. **Link the progress-assessment tool from the guided learning path** — `assessment/README.md`
   (previously reachable only by browsing the file tree) is now linked from both `START-HERE.md`
   and `README.md`.

## Changes

- **135 lesson/quiz/comparison/reference/review pages** — each now has exactly one `Next →`
  footer segment pointing at the correct next page in `START-HERE.md`'s sequence (69 pages
  gained a `Next →` segment that did not exist before, most of them quiz pages; 66 pages had an
  existing but wrong `Next →` segment corrected). `verbs/sein.md` and `verbs/haben.md` already
  had the correct target by coincidence and were left untouched. Every other part of every
  touched footer line (`← Back`, `Category Index`/`Verb Index`/etc., `↑ Master Index`) is
  unchanged, and no other line in any of these files changed — confirmed by `git diff --stat`
  showing exactly one changed line per file.
- **7 "side-branch" pages** (`reference/a1-priority-system.md`, `comparisons/sehr-vs-viel.md` +
  its quiz, `comparisons/gerade-vs-jetzt.md` + its quiz, `comparisons/denn-vs-weil.md` + its
  quiz) — material `START-HERE.md` marks "Optional" or "Later" rather than a numbered stage
  step. Each now has a `Next →` link that rejoins the main spine at the point `START-HERE.md`
  introduces it, instead of pointing into the old category-order chain.
- **9 category `README.md` files** (`verbs/`, `words/`, `prepositions/`, `particles/`,
  `patterns/`, `comparisons/`, `phrases/`, `review/`, `reference/`) — each now states plainly
  that its listing is a lookup index rather than the recommended study order, and points to
  `START-HERE.md`. `words/README.md`'s previous "Read them roughly in this order..." sentence,
  which would have contradicted the new note, is replaced rather than left standing alongside it.
- **`START-HERE.md`** — the "Whole-Repository Review" section now links to
  `assessment/README.md` as the way to measure progress once the guided path is finished.
- **`README.md`** — the "How the Repository Is Organized" table gains a row for the assessment
  tool.
- `assessment/` (including `assessment/grader-only/`) is untouched.

## API Contract

Omitted — this repository has no API surface. It is a static Markdown content repository with
no server, client, or endpoint.

## Tests

No unit test suite applies (static Markdown, no application code). Verification was structural,
per the ticket's own Testing Guidance:

- A link-graph script confirmed every markdown link in the repository (excluding `assessment/`,
  which is intentionally out of the click-through spine) resolves to a real file, both before
  and after the change.
- A chain-walk script started at `reference/how-to-read-without-translating.md`, followed only
  `Next →` links, and compared the resulting sequence against the 131-item canonical spine
  derived from `START-HERE.md`. Also checked the 7 side-branch pages' `Next →` targets
  individually.
- `audit_quizzes.py` (the repository's existing content-integrity script) was run before and
  after; its output is byte-identical, confirming no regression in the checks it already covers
  (answer-key sanity, broken links, lesson/quiz pairing, review-quiz file presence).

## Manual Verification

| Case | Interaction | Expected | Actual | Result |
| ---- | ----------- | -------- | ------ | ------ |
| Full spine walk | Follow `Next →` from `reference/how-to-read-without-translating.md` to the end, link by link (scripted) | Reproduces all 131 canonical spine pages, in order, no repeats, no skips | Walked length 131, exact match to canonical spine | Pass |
| Final page | Check `review/cumulative-review.md`'s footer | No `Next →` segment (this is the end of the path) | No `Next →` segment present | Pass |
| Stage 1 → Stage 2 boundary | `verbs/spelling-and-stem-changes-quiz.md`'s `Next →` | Points to `words/nicht.md` (start of Stage 2) | Points to `words/nicht.md` | Pass |
| Stage 4 → Stage 5 boundary (worst violation found in research) | `reference/active-vs-passive-vocabulary.md`'s `Next →` | Points to `verbs/present-tense-basics.md` (Stage 1 start, its true successor), not `reference/preposition-overview.md` (the old Stage-5 skip) | Points to `verbs/present-tense-basics.md` | Pass |
| Quiz dead end (regression case) | Any of the 75 previously-Next-less quiz pages, e.g. `words/nicht-quiz.md` | Has a `Next →` segment | `Next → kein` present | Pass |
| Side branch: "Later" comparisons | `comparisons/sehr-vs-viel.md` → quiz → `comparisons/gerade-vs-jetzt.md` → quiz → rejoin | Chain lands on `reference/preposition-overview.md` (Stage 5 start) | Confirmed via script, all 4 hops correct | Pass |
| Side branch: `denn vs. weil` | `comparisons/denn-vs-weil.md` → quiz → rejoin | Chain lands on `review/cumulative-review.md` | Confirmed via script | Pass |
| Category index pages unchanged in ordering | `verbs/README.md`, `prepositions/README.md`, etc. | Listing tables unchanged; only a lookup-index note added | `git diff` shows 2 added lines per file, table untouched | Pass |
| `words/README.md` wording conflict | Old "Read them roughly in this order" line | Does not stand alongside the new lookup-index note | Line replaced with a single non-contradictory sentence | Pass |
| Assessment tool discoverable | `README.md` organization table and `START-HERE.md`'s Whole-Repository Review section | Both link to `assessment/README.md`; `assessment/` itself untouched | Both links present; `git status assessment/` clean | Pass |
| No collateral damage | Full-repo link-graph check (broken links + reachability) before/after | Same 2 pre-existing orphans (`APPLY-OVERLAY.md`, `GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md`), 0 broken links | Same 2 orphans, 0 broken links (the two `assessment/README.md` "broken link" flags are false positives — the checker script deliberately excludes the `assessment/` directory from its file inventory; the file exists and was confirmed with `test -f`) | Pass |
| Existing content-integrity checks | `python3 audit_quizzes.py` before vs. after | No new problems | `diff` of before/after output is empty | Pass |

This was exercised by rendering the repository's own Markdown structure and walking it
programmatically rather than in a browser; the link targets and footer text are identical
either way since GitHub-flavored Markdown link resolution is what was checked.

## Verification

**Status: Complete.**

All acceptance criteria across the three tickets were checked against recorded evidence:

- Ticket 1 (AC1–AC9): confirmed by the chain-walk script (AC1, AC2, AC6), the side-branch table
  above (AC5), `git diff --stat` showing one changed line per touched file and no category
  `README.md` in the diff (AC7, AC9), and the `audit_quizzes.py` before/after diff (AC8). AC3/AC4
  (add-vs-correct counts) reconcile with the 69 inserted / 66 replaced / 2 already-correct split
  found while applying the edits (137 edges total, matching the ticket's ~75/~70 estimate).
- Ticket 2 (AC1–AC3): confirmed by reading the diff on all 9 category `README.md` files and
  the `words/README.md` wording replacement.
- Ticket 3 (AC1–AC3): confirmed by grepping both new links and by `assessment/` showing no
  changes in `git status`.

No lint or test command is defined for this repository beyond `audit_quizzes.py`, which was run
and shows no regression.

## Risks / Follow-ups

- The `←` (previous) segment on each touched footer was deliberately left unchanged per the
  ticket's AC7/Out-of-Scope — it still reflects the old category-order predecessor on the 66
  corrected pages (e.g. `comparisons/denn-vs-weil.md`'s `←` still points to `zu-hause-vs-nach-
  hause`, its old category-order neighbor, not its new spine neighbor). Clicking "Next" all the
  way through works correctly; clicking "←" from a page reached via the new forward chain will
  not retrace the same path backward. This was an explicit ticket scope decision, not an
  oversight — fixing backward navigation to match would be a natural follow-up ticket if wanted.
- Where a `Next →` link needed inventing rather than correcting (the 69 previously-absent
  cases), its visible link text was derived from how the same target page is already referred to
  elsewhere in the repository's own footers (a majority-vote over existing `Next`/`←` link text,
  falling back to the quiz page's own `Quiz: <title>` heading transformed into the repository's
  established `**<title>** quiz` phrasing for the 59 cases with no prior footer reference at
  all). This reproduces the repository's existing conventions rather than inventing a new one,
  but was not itself dictated by the ticket text — it is a formatting judgment call, not a
  content or ordering one.
- Ticket 1's non-blocking open question (exact visible link text is an implementer judgment
  call) was resolved by the majority-vote approach above rather than a single repo-wide style
  rule, since the repository itself already mixes styles by category (e.g. backtick-wrapped
  bare verb infinitives in `verbs/` vs. plain lowercase words elsewhere) and matching each
  target's existing convention was judged more consistent than normalizing it.

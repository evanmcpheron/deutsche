# Tickets: A fully click-through guided learning path

Dependency order: **Ticket 1** (the core navigation fix) has no dependencies and should land
first. **Ticket 2** (labeling category indexes) depends on Ticket 1 because its note claims
the footers no longer follow category order — that claim has to be true before the note ships.
**Ticket 3** (surfacing the assessment tool) is independent of the other two and can be done
in any order, including in parallel.

````markdown
# Make every page's "Next →" link follow the guided learning path in order

**Type:** Bug
**Repos:** This repository (single static Markdown content repo; no server/client/API involved)
**Depends on:** Nothing
**Size:** L

## User Story
As a learner working through this repository (per `README.md`'s "Who It Is For"), I want
every lesson, quiz, comparison, and checkpoint page to end with a "Next →" link that leads
to the actual next step in the guided path, so that I can click straight through the entire
course from `README.md` to the end without ever going back to `START-HERE.md` to figure out
what comes next.

## Context
`START-HERE.md` already defines one correct, staged, interleaved order for the whole course.
Every page in that order carries a `Next →` footer link, but those links were generated from
each category's own `README.md` table order instead of from `START-HERE.md`. This was
confirmed mechanically, not by sampling:

- `prepositions/README.md` lists `mit, zu, nach, bei, aus, von, für, um, in, auf, an`, and the
  actual `Next →` chain across `prepositions/*.md` is exactly `mit→zu→nach→bei→aus→von→fuer→
  um→in→auf→an` — the category README order, not `START-HERE.md`'s order (which interleaves
  `bei`/`mit` and their comparison ahead of `zu`). The same match was confirmed for `words/`.
- All 75 `*-quiz.md` files (across `verbs/`, `words/`, `prepositions/`, `particles/`,
  `patterns/`, `comparisons/`, plus the `review/*.md` checkpoints) have **no `Next →` link at
  all** — their footer is uniformly `[← Back to lesson] | [Category Index] | [↑ Master Index]`.
- The worst mismatches actively eject the reader from the intended order:
  `reference/active-vs-passive-vocabulary.md` (one of the two pages `START-HERE.md` says to
  read "Before Stage 1") currently has `Next → reference/preposition-overview.md`, a Stage 5
  page — clicking it skips Stages 1 through 4 entirely.

Full detail and the file-by-file evidence are in `.claude/work/guided-learning-path/findings.md`.

**The canonical spine.** The list below is the exact, ordered sequence extracted from
`START-HERE.md`, with two kinds of intentionally-excluded material removed (see "Side-branch
pages" below): the "Repository Organization" footer links at the bottom of `START-HERE.md`
(those are index links, not lesson content) and material `START-HERE.md` itself calls out as
optional/deferred rather than a numbered step. Every file below must have a `Next →` footer
link pointing at the file on the line immediately after it. The last file (`review/cumulative-
review.md`) has no next step — it is the end of the path.

```
1  reference/how-to-read-without-translating.md
2  reference/active-vs-passive-vocabulary.md
3  verbs/present-tense-basics.md
4  verbs/present-tense-basics-quiz.md
5  verbs/sein.md
6  verbs/haben.md
7  verbs/werden.md
8  verbs/essential-irregular-verbs-quiz.md
9  verbs/spelling-and-stem-changes.md
10 verbs/spelling-and-stem-changes-quiz.md
11 words/nicht.md
12 words/nicht-quiz.md
13 words/kein.md
14 words/kein-quiz.md
15 comparisons/nicht-vs-kein.md
16 comparisons/nicht-vs-kein-quiz.md
17 words/auch.md
18 words/auch-quiz.md
19 words/nur.md
20 words/nur-quiz.md
21 words/noch.md
22 words/noch-quiz.md
23 comparisons/auch-vs-noch.md
24 comparisons/auch-vs-noch-quiz.md
25 words/schon.md
26 words/schon-quiz.md
27 comparisons/noch-vs-schon.md
28 comparisons/noch-vs-schon-quiz.md
29 review/recognition-01.md
30 patterns/noch-nicht.md
31 patterns/noch-nicht-quiz.md
32 patterns/nicht-mehr.md
33 patterns/nicht-mehr-quiz.md
34 comparisons/nicht-mehr-vs-noch.md
35 comparisons/nicht-mehr-vs-noch-quiz.md
36 words/immer.md
37 words/immer-quiz.md
38 patterns/immer-noch.md
39 patterns/immer-noch-quiz.md
40 words/wieder.md
41 words/wieder-quiz.md
42 comparisons/noch-vs-wieder.md
43 comparisons/noch-vs-wieder-quiz.md
44 words/erst.md
45 words/erst-quiz.md
46 comparisons/erst-vs-schon.md
47 comparisons/erst-vs-schon-quiz.md
48 words/sehr.md
49 words/sehr-quiz.md
50 patterns/ein-bisschen.md
51 patterns/ein-bisschen-quiz.md
52 patterns/gar-nicht.md
53 patterns/gar-nicht-quiz.md
54 words/vielleicht.md
55 words/vielleicht-quiz.md
56 words/gerade.md
57 words/gerade-quiz.md
58 patterns/noch-ein.md
59 patterns/noch-ein-quiz.md
60 phrases/everyday-patterns.md
61 phrases/common-chunks.md
62 reference/preposition-overview.md
63 prepositions/mit.md
64 prepositions/mit-quiz.md
65 prepositions/bei.md
66 prepositions/bei-quiz.md
67 comparisons/bei-vs-mit.md
68 comparisons/bei-vs-mit-quiz.md
69 prepositions/zu.md
70 prepositions/zu-quiz.md
71 prepositions/nach.md
72 prepositions/nach-quiz.md
73 comparisons/zu-vs-nach.md
74 comparisons/zu-vs-nach-quiz.md
75 comparisons/zu-hause-vs-nach-hause.md
76 comparisons/zu-hause-vs-nach-hause-quiz.md
77 prepositions/fuer.md
78 prepositions/fuer-quiz.md
79 prepositions/um.md
80 prepositions/um-quiz.md
81 prepositions/aus.md
82 prepositions/aus-quiz.md
83 prepositions/von.md
84 prepositions/von-quiz.md
85 comparisons/aus-vs-von.md
86 comparisons/aus-vs-von-quiz.md
87 patterns/von-bis.md
88 patterns/von-bis-quiz.md
89 reference/cases-at-a-glance.md
90 prepositions/in.md
91 prepositions/in-quiz.md
92 comparisons/in-vs-nach.md
93 comparisons/in-vs-nach-quiz.md
94 prepositions/auf.md
95 prepositions/auf-quiz.md
96 prepositions/an.md
97 prepositions/an-quiz.md
98 review/sentence-completion-01.md
99 review/contrast-01.md
100 phrases/travel-patterns.md
101 verbs/modal-verbs.md
102 verbs/modal-verbs-quiz.md
103 verbs/separable-verbs.md
104 verbs/separable-verbs-quiz.md
105 verbs/reflexive-verbs.md
106 verbs/reflexive-verbs-quiz.md
107 verbs/perfekt.md
108 verbs/perfekt-quiz.md
109 verbs/praeteritum-essentials.md
110 verbs/praeteritum-essentials-quiz.md
111 review/verb-conjugation-01.md
112 reference/modal-particles-overview.md
113 particles/mal.md
114 particles/mal-quiz.md
115 patterns/schon-mal.md
116 patterns/schon-mal-quiz.md
117 particles/denn.md
118 particles/denn-quiz.md
119 particles/doch.md
120 particles/doch-quiz.md
121 particles/eigentlich.md
122 particles/eigentlich-quiz.md
123 particles/ja.md
124 particles/ja-quiz.md
125 particles/eben.md
126 particles/eben-quiz.md
127 particles/halt.md
128 particles/halt-quiz.md
129 review/recognition-02.md
130 phrases/conversation-patterns.md
131 review/cumulative-review.md
```

**Side-branch pages.** `START-HERE.md` explicitly marks four topics as optional or deferred
rather than a numbered stage step: `reference/a1-priority-system.md` ("Optional orientation"),
`comparisons/sehr-vs-viel.md` and `comparisons/gerade-vs-jetzt.md` ("Later... should wait until
the missing comparison concept has been learned elsewhere"), and `comparisons/denn-vs-weil.md`
("Later... should wait until basic `weil` word order has been learned"). These seven files
(each lesson plus its quiz, except `a1-priority-system.md` which has no quiz) are **not** part
of the numbered spine above and nothing in the spine should point to them as a "Next" target.
Each still needs a correct forward link so it is never a dead end itself:

- `reference/a1-priority-system.md` → Next → `verbs/present-tense-basics.md` (spine #3)
- `comparisons/sehr-vs-viel.md` → Next → `comparisons/sehr-vs-viel-quiz.md`
- `comparisons/sehr-vs-viel-quiz.md` → Next → `comparisons/gerade-vs-jetzt.md`
- `comparisons/gerade-vs-jetzt.md` → Next → `comparisons/gerade-vs-jetzt-quiz.md`
- `comparisons/gerade-vs-jetzt-quiz.md` → Next → `reference/preposition-overview.md` (spine #62)
- `comparisons/denn-vs-weil.md` → Next → `comparisons/denn-vs-weil-quiz.md`
- `comparisons/denn-vs-weil-quiz.md` → Next → `review/cumulative-review.md` (spine #131)

## Acceptance Criteria
AC1 - Every file in the canonical spine list above, except the last (`review/cumulative-
review.md`), has exactly one `Next →` footer link, and its target is the file on the
following line of that list.
AC2 - `review/cumulative-review.md`'s footer contains no `Next →` link (it is correctly the
end of the path; do not invent a target for it).
AC3 - Each of the 75 quiz pages that currently has no `Next →` link (every `*-quiz.md` file
under `verbs/`, `words/`, `prepositions/`, `particles/`, `patterns/`, `comparisons/`, plus
`review/sentence-completion-01.md`, `review/contrast-01.md`, `review/recognition-01.md`,
`review/recognition-02.md`, `review/verb-conjugation-01.md`) gains a `Next →` link per AC1.
AC4 - Each of the 70 pages whose existing `Next →` link currently points to the wrong target
(per `.claude/work/guided-learning-path/findings.md`'s Root Cause section) has that link
corrected to match AC1.
AC5 - The seven side-branch files listed above each have a `Next →` link matching the target
specified for them, and no spine page's `Next →` points to any of the seven.
AC6 - Starting from `reference/how-to-read-without-translating.md` and following only `Next →`
links reproduces the 131-item canonical spine in exact order with no repeats, no skips, and
no dead end before `review/cumulative-review.md`.
AC7 - No other part of any touched page's footer (`← Back`/`← <name>`, `Category Index`,
`↑ Master Index`) or body content changes. This ticket only adds or corrects the `Next →`
link.
AC8 - `python3 audit_quizzes.py`, run from the repository root, reports no new failures
compared to a run on the pre-change tree (it does not check `Next →` targets today, so it
should not report anything different — this is a check that nothing else broke).
AC9 - None of the 9 category `README.md` files (`verbs/README.md`, `words/README.md`, etc.)
are modified by this ticket.

## Implementation Plan
1. Treat the canonical spine list in Context as the source of truth. For each consecutive
   pair `(item[i], item[i+1])`, open `item[i]` and set its `Next →` footer link to point at
   `item[i+1]`, preserving the existing footer format on that page (e.g.
   `[← Back to <name>](...) | [Category Index](./README.md) | [Next → <name>](<target>) |
   [↑ Master Index](../MASTER_INDEX.md)` for quiz pages that currently have no `Next` segment
   at all — add the segment in the same position lesson pages use it, between `Category Index`
   and `↑ Master Index`).
2. For lesson/comparison/reference/review pages whose `Next →` already exists but points to
   the wrong file (the 70 from AC4), replace only the link target and its visible link text
   (the linked page's title/name), leaving the rest of the line untouched.
3. Apply the seven side-branch link targets exactly as specified in Context.
4. Do not touch `review/cumulative-review.md`'s footer — confirm it still has no `Next →`
   segment (it currently does not).
5. Do not touch any of the 9 category `README.md` files — their internal ordering is
   out of scope here (see Ticket 2).
6. After editing, verify AC6 by walking the chain: starting at
   `reference/how-to-read-without-translating.md`, follow each page's `Next →` target,
   collect the sequence, and diff it against the 131-item canonical spine list in Context.
   A short one-off script (not a repository deliverable) is the fastest way to do this — the
   `research` skill's investigation used exactly this kind of script; it does not need to be
   committed unless useful for the follow-up in Ticket 2's neighbor discussion.
7. Run `python3 audit_quizzes.py` before starting and again after finishing; diff the two
   runs and confirm no new problems were introduced (AC8).

## API Contract Changes
None - this ticket does not move the API surface. There is no server, client, or endpoint;
this is a static Markdown content repository.

## Postman Updates
None.

## Out of Scope
- Reordering, rewriting, or adding a "this is a lookup index, not the reading order" note to
  any category `README.md` file - that is Ticket 2.
- Linking `assessment/` from the guided path - that is Ticket 3.
- Adding an automated check (e.g. extending `audit_quizzes.py`) that would catch a future
  regression of this same problem - the research explicitly found this was not required for
  this request, and it is a reasonable follow-up rather than part of this fix.
- The `words/doch.md`, `words/eigentlich.md`, and `words/mal.md` stub pages - these are
  deliberate, already-correct redirect stubs to their `particles/` canonical pages (each
  explains the placement rule and links onward) and are not part of the spine or a defect;
  leave them exactly as they are.
- `verbs/high-frequency-reference.md` and any other page reachable only via a lesson's
  "Related Pages" list rather than via `START-HERE.md` - these are supplementary reference
  material, not spine steps, and do not need a `Next →` link added.
- Rewording or improving lesson/quiz content itself - this ticket only touches footer
  navigation links.

## Testing Guidance
- No unit/integration test suite applies (static Markdown, no app code). The verification
  is structural: confirm every link target actually exists as a file (a broken-link check,
  e.g. `grep`-ing each `Next →` target and testing the resolved path with `test -f`), and
  confirm the full-chain walk in Implementation Plan step 6 reproduces the canonical spine
  exactly.
- Regression: `audit_quizzes.py` already checks other things (answer-key sanity, broken
  links, lesson/quiz pairing) - rerun it and confirm it is still clean (AC8).
- Manual spot check: open `README.md` in a plain Markdown renderer (e.g. GitHub's) and click
  only "Next →" from the first page through at least two full stage transitions (e.g. the
  Stage 1 → Stage 2 boundary and the Stage 4 → Stage 5 boundary, both of which involve a
  category change and were among the worst violations found in research) to confirm the
  rendered experience matches the list, not just the raw text.

## Edge Cases
Stage boundary (last item of one stage to first item of the next, e.g. spine #10→#11,
verbs to words) -> Next must cross into the new category correctly, not stay inside the old
one (this was the single most common failure mode found in research).
Side-branch page visited by a curious reader (e.g. clicking into `comparisons/sehr-vs-viel.md`
from its "Related Pages" mention on `words/sehr.md`) -> its `Next →` rejoins the spine at the
defined point rather than dead-ending or looping among "Later" pages only.
Final page (`review/cumulative-review.md`) -> correctly has no `Next →`; do not add one just
for footer-format consistency.
A page's visible `Next →` link text (the linked page's display name) -> must match the new
target, not the old one (e.g. a footer that used to read "Next → sein" but now points to the
quiz must read "Next → Present-Tense Basics Quiz" or equivalent, not still say "Next → sein").

## Open Questions
**Blocking:** None - the canonical spine and every side-branch target are fully specified
above.

**Non-blocking:**
- The exact visible link text/title used for each corrected `Next →` link is left to the
  implementer's judgment, following the naming style already used on nearby footers (e.g.
  `[Next → schon]` vs `[Next → **schon**]`) - assumption: consistency with the immediately
  surrounding footer's existing style on each page matters more than a single repo-wide
  convention.
````

````markdown
# Label category index pages as a lookup index, not the reading order

**Type:** Chore
**Repos:** This repository
**Depends on:** "Make every page's 'Next →' link follow the guided learning path in order"
**Size:** S

## User Story
As a learner who arrives at a category page (e.g. `prepositions/README.md`) by clicking it
from `README.md`'s organization table rather than through `START-HERE.md`, I want that page
to tell me plainly that its list is a lookup index and not the order to study in, so that I
do not assume the order I see there is the recommended path.

## Context
The previous ticket removes the category-`README.md`-order `Next →` chain that lesson pages
used to follow. That chain existed because each category `README.md` (e.g.
`prepositions/README.md:9-19`, `words/README.md:9-20`) lists its lessons in a plain
alphabetical/topical order for lookup purposes, and once that is no longer silently doubling
as the click-through order, a learner who enters through a category page (a path
`README.md`'s "How the Repository Is Organized" table actively invites, since it links
directly to `verbs/README.md`, `words/README.md`, etc.) has no signal that this list is not
the recommended study sequence. `words/README.md` already models close to the right idea in
its "Words Whose Canonical Page Is Elsewhere" section and its "Read them roughly in this
order. The first six are Stage 1 in Start Here" line - but not every category README makes
this explicit, and none of them warn that the table's order is a lookup convenience rather
than the guided path.

## Acceptance Criteria
AC1 - Each of the 9 category `README.md` files (`verbs/`, `words/`, `prepositions/`,
`particles/`, `patterns/`, `comparisons/`, `phrases/`, `review/`, `reference/`) contains a
short, clearly-visible note stating that the page's listing is a lookup index rather than the
recommended study order, and linking to `START-HERE.md` for the order.
AC2 - The note does not remove, reorder, or otherwise change the existing per-category listing
table/list on any of the 9 pages.
AC3 - The note's wording does not contradict `words/README.md`'s existing "Read them roughly
in this order..." line - either that line is updated for consistency with the new note, or
the new note is written to sit alongside it without contradiction (implementer's call, but
the two must not tell the reader opposite things on the same page).

## Implementation Plan
1. Add one short note (a sentence or two, not a new heading-level section) near the top of
   each of the 9 category `README.md` files, after the existing title/breadcrumb line, e.g.
   in the spirit of: "This page lists <category> for lookup. For the recommended study order,
   see [Start Here](../START-HERE.md)."
2. Reconcile with `words/README.md`'s existing "Read them roughly in this order. The first six
   are Stage 1 in Start Here" sentence (`words/README.md`) so the page does not carry two
   conflicting claims about ordering.
3. Leave every other line of each file unchanged.

## API Contract Changes
None - this ticket does not move the API surface.

## Postman Updates
None.

## Out of Scope
- Changing the actual listing order in any category `README.md` - these remain in their
  current lookup order (e.g. alphabetical/topical), unchanged.
- Any change to lesson, quiz, or `START-HERE.md` content - that is the previous ticket.
- `MASTER_INDEX.md` and `CONCEPT-INDEX.md` - these are explicitly lookup tools already
  (per `README.md`'s own description) and are not category README pages.

## Testing Guidance
Manual read-through of each of the 9 edited files to confirm the note is present, does not
contradict any existing ordering claim on the same page, and the link to `START-HERE.md`
resolves. No automated test applies.

## Edge Cases
`words/README.md`'s existing "Read them roughly in this order" line -> must not be left
standing unchanged next to a new note that says the order is not the recommended one; resolve
per AC3.

## Open Questions
**Blocking:** None.

**Non-blocking:** Exact wording of the note is left to the implementer; the requirement is
only that it states the list is a lookup index and points to `START-HERE.md`.
````

````markdown
# Link the progress-assessment tool from the guided learning path

**Type:** Feature
**Repos:** This repository
**Depends on:** Nothing
**Size:** S

## User Story
As a learner who has followed the guided path through `START-HERE.md`, I want a link to the
existing progress-assessment tool at the point where I've finished the course, so that I can
find it without already knowing it exists or browsing the repository's file tree.

## Context
`assessment/` is a complete, separate, deliberately closed-book progress-test tool
(`assessment/README.md`) with its own file order and grading process. It is not linked from
`README.md` or `START-HERE.md` anywhere - a learner who works through the entire guided path
has no way to discover it short of browsing the directory listing. This is a real gap in
"never have to wonder what to do next": at the exact moment `START-HERE.md` finishes ("Whole-
Repository Review"), the natural next question - "how do I know if this worked?" - has an
existing answer that is completely unlinked.

## Acceptance Criteria
AC1 - `START-HERE.md`'s "Whole-Repository Review" section (or immediately after it) contains
a link to `assessment/README.md`.
AC2 - `README.md`'s "How the Repository Is Organized" table gains a row for the assessment
tool, linking to `assessment/README.md`, consistent with the existing row format (name +
one-line description).
AC3 - No file under `assessment/` (including `assessment/grader-only/`) is modified.

## Implementation Plan
1. In `START-HERE.md`, after the "Whole-Repository Review" section's existing two checkpoint
   bullets, add a line pointing to `assessment/README.md` as the way to measure progress
   once the guided path is finished.
2. In `README.md`'s organization table (`README.md`, the `| Section | What's inside |`
   table), add a row for `assessment/README.md`, following the existing row format used for
   `Review` and `Reference`.
3. Do not add a `Next →` footer to any file inside `assessment/` - that tool has its own
   self-contained file order (`assessment/README.md`'s "File Order" section) and is
   intentionally separate from the click-through spine covered by the other tickets in this
   set.

## API Contract Changes
None - this ticket does not move the API surface.

## Postman Updates
None.

## Out of Scope
- Any change to files under `assessment/` itself, including its own internal ordering,
  wording, or grading instructions.
- Wiring `assessment/` into the canonical `Next →` spine from the other ticket in this set -
  the assessment tool is a separate, self-contained flow, not a spine step.

## Testing Guidance
Manual check: confirm the new links in `START-HERE.md` and `README.md` resolve to
`assessment/README.md`, and that no file under `assessment/` shows a diff.

## Edge Cases
None beyond the standard link-resolves check - this ticket adds two links and touches no
existing behavior.

## Open Questions
**Blocking:** None.

**Non-blocking:** Whether the assessment link belongs only at the end of the path (after
"Whole-Repository Review") or also earlier (e.g. near the top of `START-HERE.md`, as a
"you can self-test at the end" note) is left to the implementer. Assumption used here: only
at the end, since taking it early would contradict `assessment/README.md`'s own instruction
not to use the unfinished assessment as a study guide.
````

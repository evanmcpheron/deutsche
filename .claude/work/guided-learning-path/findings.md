# Research: A fully click-through guided learning path

## Request
Ensure that anyone can start at `README.md` and click their way through the entire
repository, hitting every lesson, without ever having to stop and figure out what to do
next. Classification: **bug / consistency defect** in an existing feature (the repo
already claims, in `GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md`, that this was built and
completed) — not a request for new content.

## Summary
This is a static Markdown self-study German course (no app, no server, no API — `CLAUDE.md`
does not exist in this repo, and none of the platform conventions it would normally define
apply here). `README.md` → `START-HERE.md` already defines one correct, fully-interleaved,
staged sequence of ~150 pages, and every page in that sequence is reachable and every link
in the repo resolves (0 broken links, only 2 unreachable files, both internal planning docs,
not lessons). The problem is that a **second, independent, conflicting navigation system**
exists on the pages themselves: quiz pages have no forward link at all (a dead end after
every quiz, ~75 files), and most lesson/comparison/checkpoint/reference pages carry a
`Next →` footer link that was built from each category's own `README.md` listing order
instead of from `START-HERE.md`'s order. Someone who reads `START-HERE.md` once and then
clicks forward on each page — the obvious, discoverable thing to do — is silently pulled
off the curated path: they skip every quiz, comparison, checkpoint, and cross-stage
reference page, and instead walk a same-category chain nobody documented. This is exactly
the "click their way through and never question what to do next" failure the request
describes, and it is verifiable mechanically, not a matter of opinion.

## Current behavior
- `README.md:9` points to `START-HERE.md` as the canonical path; `START-HERE.md` lays out
  9 stages as an ordered list of `lesson → quiz`, `comparison → quiz`, and checkpoint links
  (`START-HERE.md:33-186`).
- A full link-graph BFS from `README.md` over every `.md` file outside `assessment/`
  (159 files) finds **zero broken links** and only **2 unreachable files**:
  `APPLY-OVERLAY.md` and `GERMAN_REPOSITORY_IMPROVEMENT_ROADMAP.md` — both internal
  maintenance docs, not learner content. So reachability-by-some-path is solid.
- Every lesson page ends with a navigation footer, e.g.
  `verbs/present-tense-basics.md:end`:
  `[← Verb Index] | [Quiz →] | [Next → sein] | [↑ Master Index]`
  and `words/noch.md:end`:
  `[← auch] | [Category Index] | [Next → schon] | [↑ Master Index]`.
- Every quiz page's footer, checked across `verbs/`, `words/`, `prepositions/`,
  `particles/`, `patterns/`, and `comparisons/` (sampled `words/nicht-quiz.md`,
  `prepositions/mit-quiz.md`, `particles/denn-quiz.md`,
  `comparisons/nicht-vs-kein-quiz.md`, `patterns/noch-nicht-quiz.md`, and confirmed
  programmatically for all 75 quiz files) is uniformly:
  `[← Back to lesson] | [Category Index] | [↑ Master Index]` — **no forward link at all.**

## Root cause
Two independently-authored sequencing systems were never reconciled:

1. **`START-HERE.md`** encodes the intended pedagogical order: interleave vocabulary,
   grammar, comparisons, and checkpoints across stages (`README.md:47`: "alternates
   vocabulary/grammar systems so later lessons can use structures you have already seen").
2. **Each category's own `Next →` footer chain** was generated from that category's
   `README.md` table order, not from `START-HERE.md`. This is directly verifiable: 
   `prepositions/README.md` lists `mit, zu, nach, bei, aus, von, für, um, in, auf, an` in
   that order, and the measured `Next →` chain across `prepositions/*.md` is exactly
   `mit→zu→nach→bei→aus→von→fuer→um→in→auf→an` — a perfect match to the category README,
   and a mismatch to `START-HERE.md`'s actual order (which interleaves `bei`/`mit` and their
   comparison before `zu`, etc.). The same match against category `README.md` order was
   confirmed for `words/`. This is the general mechanism, not a one-off typo.

A full comparison (script-verified, see Constraints) of the "declared next stage-sequence
item" per `START-HERE.md` against the "actual `Next →` target" on each page found:

- **75 files have no `Next →` link at all** — every one of the 75 quiz files, plus a
  handful of stage-transition pages (`verbs/praeteritum-essentials.md`,
  `patterns/ein-bisschen.md`, `words/gerade.md`, `prepositions/an.md`,
  `phrases/common-chunks.md`, `comparisons/denn-vs-weil.md`,
  `reference/how-to-read-without-translating.md`, and the nine category `README.md`
  index pages, which is expected/fine for indexes).
- **70 files have a `Next →` link that points somewhere other than the next item in
  `START-HERE.md`'s sequence.** The worst cases actively catapult the reader out of the
  intended order, e.g. `reference/active-vs-passive-vocabulary.md` — one of the two pages
  `START-HERE.md` says to read "Before Stage 1" — has `Next → reference/preposition-overview.md`,
  a Stage 5 page, so clicking Next from there **skips Stages 1 through 4 entirely.**
  `review/contrast-01.md → review/sentence-completion-01.md → review/recognition-02.md →
  review/cumulative-review.md` forms its own closed loop through `review/`, independent of
  where those checkpoints actually sit in `START-HERE.md`'s stages.

## Affected surface area
Single repository (no server/mobile split; this is a Markdown content repo).

- All **~75 `*-quiz.md` files** across `verbs/`, `words/`, `prepositions/`, `particles/`,
  `patterns/`, `comparisons/`, plus the `review/*.md` checkpoints — need a `Next →` footer
  link added.
- **~70 lesson/comparison/reference/review pages** — need their existing `Next →` footer
  link corrected to point at the actual next item in `START-HERE.md`'s order instead of the
  next item in that page's own category `README.md`.
- The 9 category `README.md` index pages (`verbs/README.md`, `words/README.md`, etc.) are
  not themselves broken, but they are what generated the wrong chain, and `README.md`'s
  organization table links straight into them — a learner who clicks a category link out
  of curiosity, then clicks "Next," is walking the wrong sequence with no signal that
  anything is off.
- `assessment/` (a separate, deliberately closed-book progress test, per
  `assessment/README.md`) is **not linked from `README.md` or `START-HERE.md` at all** —
  it's discoverable only by browsing the file tree.

## Existing patterns to follow
- The footer convention itself (`← previous | Category Index | Next → | ↑ Master Index`)
  is already established and just needs its target corrected — no new UI/format to invent.
- `START-HERE.md` is already the single authoritative ordered list; it should be the input
  the corrected `Next →` links are generated or checked against, not the category
  `README.md` files.
- `audit_quizzes.py` (repo root) already does mechanical checks (broken links, answer-key
  sanity, lesson/quiz pairing) but does **not** check `Next →` sequencing — it's the natural
  place a future check could live, but adding one is not itself required by this request.

## API contract impact
None — the API surface does not move. This is a static Markdown repository with no
server, client, or endpoint.

## Data and migration impact
None. No schema, no database; the "data" is Markdown files, and no filenames or directory
structure need to change.

## Test coverage today
`audit_quizzes.py` covers: quiz question counts, multiple-choice answer distribution,
answer-key letters that don't exist among options, duplicate options, internal relative
links that resolve, and lesson/quiz pairing (`audit_quizzes.py:1-13`). It does **not**
assert anything about `Next →` footer targets or about the page reachable from a `Next →`
chain matching `START-HERE.md`'s order — this gap is new territory, not a regression of an
existing check.

## Options considered
1. **Point every quiz's and every mis-pointed page's `Next →` link at the actual next item
   in `START-HERE.md`'s interleaved order**, so clicking "Next" anywhere in the sequence
   reproduces `START-HERE.md` exactly, stage transitions included.
   *Tradeoff:* touches ~145 footer lines; mechanical but not trivial, since the target for
   many pages differs by stage (e.g., a preposition lesson's "next" is sometimes its quiz,
   sometimes a comparison, sometimes a different preposition).
2. **Remove `Next →` links from lesson pages entirely and route everything back through
   `START-HERE.md`** (i.e., every page's forward action is "return to Start Here").
   *Tradeoff:* fewer lines to touch, but reintroduces exactly the friction the request
   asks to eliminate — the reader has to re-scan a 150-line page to find their place after
   every single lesson and every single quiz.

**Recommendation: option 1.** The repository's own stated design goal (`README.md:47`,
the roadmap's Step 1 completion criteria) is a single deliberate interleaved order; a
learner should never need to consult an index to know the next click. Option 2 satisfies
"never be wrong" only by giving up "never have to think," which is the actual complaint.

## Constraints and risks
- The category `README.md` ordering (e.g. `prepositions/README.md`) is a reasonable
  standalone reference order for "look up this preposition" browsing — it should probably
  stay as-is for that purpose, distinct from the sequential path. Conflating "index order"
  with "reading order" is the root cause, so the fix should make clear these are two
  different things (e.g. category README pages could gain a one-line note that they are a
  lookup index, not the recommended order), not just silently rewrite one to match the other.
- Some `Next →` targets are inherently ambiguous by stage-boundary design: e.g. after the
  last item of Stage 4, `START-HERE.md` lists two "Recognition exposure" phrase pages with
  no numbering, then a "Later" aside for two comparisons that "should wait." A mechanical
  "next item in document order" walk needs a decision about whether "Later"/optional asides
  are ever the literal next click, or whether they're deliberately excluded from the strict
  chain (this findings doc treated them as excluded from the strict click-through spine,
  consistent with `START-HERE.md` calling them out separately from the numbered stage list).
- This was investigated and reported as findings only, per this skill's boundary — no
  footer text has been changed.

## Open questions

**Blocking (a ticket for the fix needs an answer before implementation starts):**
- None. The canonical order is already fully specified in `START-HERE.md` and was
  extracted programmatically for this findings doc; a ticket to correct/add `Next →` links
  can proceed directly from that extracted order.

**Non-blocking (a stated assumption is enough to start):**
- Should `assessment/` be linked from the guided path at all, and if so where (end of
  "Whole-Repository Review" in `START-HERE.md`, and/or the root `README.md` organization
  table)? Assumption used here: yes, it's an existing, finished, deliberately separate tool
  that a learner following the guided path currently has no way to discover — surfacing it
  is low-risk and in scope for "never have to wonder what to do next."
- Should category `README.md` pages get an explicit "this is a lookup index, not the
  reading order" note once their old `Next →`-generating role is gone? Assumption: yes,
  one line each, to prevent the same confusion from being reintroduced later by anyone who
  regenerates a category page from its README again.

## Suggested ticket slicing

- **`01-canonical-sequence-source-of-truth`** — Produce the single authoritative,
  machine-checkable ordered list of every page in `START-HERE.md`'s click-through spine
  (lesson → quiz → comparison → quiz → checkpoint, stage by stage, in the exact order
  `START-HERE.md` presents them), resolving the "Later"/optional-aside ambiguity noted
  above. This is the input every other ticket in this set consumes.
  Depends on: nothing. Repos: this repo.

- **`02-add-next-links-to-quiz-and-checkpoint-pages`** — Add a `Next →` footer link to
  every one of the ~75 quiz pages and the `review/*.md` checkpoint pages, pointing at the
  correct next node from ticket 01. Currently these pages have no forward link at all,
  which is the hardest dead end in the repo (the reader has just finished a quiz and has
  no click available).
  Depends on: `01-canonical-sequence-source-of-truth`. Repos: this repo.

- **`03-correct-mispointed-next-links`** — Rewrite the ~70 existing `Next →` links on
  lesson, comparison, reference, and checkpoint pages that currently point to the wrong
  target (their own category's internal order, per the root-cause mechanism above) so they
  match ticket 01's order instead.
  Depends on: `01-canonical-sequence-source-of-truth`. Repos: this repo.

- **`04-label-category-index-pages-as-lookup-not-order`** — Add a one-line clarification to
  each of the 9 category `README.md` files noting that their listing is a lookup index, not
  the recommended study order, and pointing to `START-HERE.md` for the order — so the same
  category-order-vs-path-order confusion cannot silently reappear.
  Depends on: `03-correct-mispointed-next-links` (so the note's claim matches reality once
  the footers no longer follow category order). Repos: this repo.

- **`05-surface-assessment-tool-from-guided-path`** — Add a link to `assessment/README.md`
  from `START-HERE.md`'s "Whole-Repository Review" section and/or `README.md`'s
  organization table, so the existing progress-assessment tool is reachable from the
  guided path instead of requiring the reader to browse the file tree.
  Depends on: nothing. Repos: this repo.

## Confidence
**High.** The core defect (quiz dead ends, mismatched `Next →` chains, and their root
cause in category-README-order vs. `START-HERE`-order) was verified mechanically — a
link-graph BFS for reachability/broken-links, and a script comparing every page's actual
`Next →` target against `START-HERE.md`'s declared sequence — not inferred from a sample.
The one area of judgment (how to treat "Later"/optional asides in the strict click-through
spine) is called out explicitly as a non-blocking open question rather than presented as
settled fact.

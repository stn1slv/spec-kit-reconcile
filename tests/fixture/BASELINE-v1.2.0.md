# Baseline: v1.2.0 (tag `v1.2.0`, commit `c4a730a`) against the fixture

First execution round. Expectations pre-registered in `9bec198`, `2fcc9c2` and `c911604`, all **before** these runs.

Five cases, five fresh agents, one per case. Each was given only `commands/reconcile.md` and its own working copy of `project/` outside this repository, made a git tree so the nothing-written rule could be checked, and was forbidden from reading `EXPECTATIONS.md`, the CHANGELOG, git history, or this repository at all. Every claim below was verified against the files each run wrote, not against its narration.

**Everything in this round came from executing the command.** v1.2.0 had already been through three review rounds with three reviewers each — 34 findings, then 22, then 8, all applied. None of those rounds found the top three findings here.

| Case | Invocation | Result |
|---|---|---|
| A | 001, full scope | **Fail** — one expectation, and the expectation was wrong |
| B | 001 `--spec-only` | Pass |
| E | 001, constitution conflict, left unresolved | Pass |
| H | 002, full scope | Pass |
| I | 003, refined report under an existing slug | Pass |

## The one failure is the fixture's, not the command's

**Case A's `R8` negative expectation is wrong.** It asserts that Architecture Standard A1's condition is not met, because the drift adds a *client* route rather than an API route, and declares any constitution finding on the case a miss. Two runners — Case A and Case B, independently, on the same feature — flagged A1 as an unmet obligation. Both are right. The gap report says "**Backend and tests** for the settings screen exist", which means a settings API route exists that `plan.md` does not declare. A1's condition is met by the report's own words.

This trap was already flagged in review. Two reviewers said it graded an inference rather than a rule; the fix asserted the opposite inference instead of removing the dependency on one. A confidently wrong expectation is worse than the ambiguity it replaced, because it scores a correct run as a failure.

Underneath it is a real command defect the runners named: settling whether a settings API route exists requires reading `src/api/`, and **Allowed Sources forbids taking facts about the drift from source code**. The command demands a judgment about the codebase while forbidding the read that would settle it.

## What every runner hit

**The Targets table's row unit is wrong, and all five runs reported it — each from a different direction.**

The rule says "one row per item-artifact pair", and the counts line declares `R (item-artifact pairs)` with `K + N = R` minus withheld and no-artifact rows. The five encounters:

- **A** — one item that *adds* a Routing entry and *amends* Testing Strategy in the same artifact. One row cannot be both `K` and `N`.
- **B** — an item that reaches an artifact excluded by a scope modifier. The formula subtracts withheld and no-artifact rows; out-of-scope is a third case it never contemplated.
- **E** — a withheld item's row has no defined `Artifact` or `Target` cell, and "one row" versus "one row per artifact it would have reached" changes `R`.
- **H** — one item reaching *three entries inside one artifact* (routing, contracts, testing). As H put it: the rule anticipated an item reaching two artifacts, but not an item reaching several entries within one, **which is the ordinary case** — a new endpoint almost always touches all three.
- **I** — one item amending two entries in each of two artifacts. Reading (a) gives `R = 4`; reading (b), which the rule's own rationale ("each of those choices needs its own preview") supports, gives `R = 7`. Identical edits, different published numbers.

Every runner produced correct *edits* and a different *number*. The counts exist so a zero is legible as "examined and found nothing" rather than "did not look"; as specified they are not reproducible, which is the failure they were added to prevent. The command's own example table never exercises the case: every row in it happens to be one item, one artifact, one target.

## What more than one runner hit

- **`Wiring & Navigation` is undefined at its boundary and silently controls a MUST** (A, E, I). The category table gives "Missing routes, menu items, sidebar links"; the preamble separately says "unlinked UI". A missing table column (E), a component that never mounts below a size threshold (I) — neither is a route or a link, yet both make something unreachable. 4.3 rule 4 makes an integration test **mandatory** for this category and optional otherwise, so the classification changes what lands on disk. E classified as Logic/UX and wrote no test; I classified as Wiring and wrote one. Both defensible, different files.
- **"Amend the `FR-XXX` that states the changed capability, **or** add one" has no tiebreaker** (A, E, H). E amended FR-003; A and B added FR-006 for comparable drift; H added REQ-005 beside REQ-003 and reasoned that amending would silently delete a capability. The target table records the choice but no rule governs it.
- **Minting a user story collides with Allowed Sources** (A, B). 4.1 says to add a story when no existing one covers the drift; the story format demands a priority, a "Why this priority" and an "Independent Test" that a gap report does not supply, and Allowed Sources forbids inventing them. Both runners declined to add one, so a functional requirement now exists with no acceptance scenario anywhere.
- **One slug per run breaks on a report carrying two drifts** (A, E). A minted a compound slug; E minted one named after the whole report so a later run landing the withheld half would match. Nothing tells a later run to look for a compound key.

## Single-runner findings that verify

- **The revision note nests inside the wrong section** (A). The form is fixed at `###` and the placement at "the bottom of the file, after the last content section", but the last section in these files is a `##` — so the note renders as a subsection of `## Assumptions` and `## Constitution Check` rather than a peer. Confirmed by inspection. This rule was revised twice in review and the malformation was never noticed.
- **"Update, do not append" forbids the append that a refined report requires** (I). Read literally, a slug already present means updating what the earlier run wrote; I's report added genuinely new work (a mounting gap, a retry path) that the earlier run could not have written. I appended and left the earlier task untouched, reasoning from the two limits that follow the rule, both of which are about not losing earlier work. The literal reading would fold three distinct fixes into one vague task.
- **Nothing says whether an existing note's `Items:` line may grow** (I), when a later run under the same slug touches a new target. I extended it, since the line's stated job is to tell the next run which entries to amend.
- **The Assumptions rule has no branch for an assumption outside the artifacts** (H). It says to *correct* an assumption the implementation invalidated; H's assumption lived in a design review, which is not an Allowed Source. There was nothing to correct, so the section was created and the report's statement recorded — leaving an entry that says an assumption no longer holds without a reader ever seeing what it said.
- **A task whose target is a spec artifact is not contemplated by the path rules** (H). The remediation for an unmet obligation points at `spec.md` itself, while 4.3's vocabulary is written for source paths.
- **`.specify/extensions.yml` is not in Allowed Sources** (A) although 1.2 reasons about bugfix extensions. Either the list is missing an entry or 1.2 is applied blind.
- **`tasks.md` creation specifies a heading but not whether a title is allowed** (H). H added an H1 taken from the spec's title — one line the command did not authorise.
- **Shape-2 obligations are feature-conditioned while the check is drift-bound** (I). Principle II conditions on "every feature that stores user data"; 003 stores user data and states no retention rule, but the drift never touches retention, so I did not flag it. H's drift did touch storage, so H flagged it. Both correct under the rule — but the tension is structural and will recur on every feature-conditioned obligation.

## What the runs confirm works

- **The shape-2 self-satisfied obligation exemption** (H): the digest route was added to plan Routing and A1's statement half was correctly *not* flagged. This rule was added in `2fcc9c2` in response to review and had never been executed.
- **Role-based section mapping** (H): `## Assumptions` landed after M1/M2 and before `## Open Questions`, with the runner reasoning explicitly that `## How We'll Know It Works` plays the Success Criteria role. Both the rule and the fixture's discriminating third section were added in `c911604`.
- **Withholding** (E): the conflicting item was written into no artifact, generated no task, and was absent from the `Items:` line, while the rest of the report was reconciled and the run completed. Every registered detail held.
- **The forbidden-source trap** (I): `bugs/BUG-002.md` contains the correct amendment for FR-003 and was explicitly not read; `## Sources` says so. The struck FR-002 wording was not restored, the `**Bugfix**:` line was left as metadata, the orphan strikethrough was left untouched and named, and the reopened task was neither edited nor repurposed.
- **Slug reuse** (I): `upload-progress` was reused across all three artifacts, both notes amended in place with their original date, no second note.
- **Path discipline** (A, B, I): every resolvable path was declared under `## Sources`; "the settings service" and "the retry path" resolved to nothing and produced no invented path in any run.
- **Argument-vs-script precedence** (A, B, E, H): all four reported the divergence from `feature.json`.

## A second fixture defect

**003's pre-existing revision note claims an edit its file does not show** (found by I). The note's `Items:` line names `FR-003` and its `Reason:` says a cancel affordance was reconciled, but FR-003 reads only "The system MUST show upload progress while a file is transferring". The command tells a run to trust the `Items:` line to identify which entries to update, and says nothing about a line whose claim the file contradicts. I trusted it as a pointer and amended the entries on their actual text. The fixture should be made self-consistent, or the case should register this as a deliberate trap with a stated expected behaviour.

## Method note

Three review rounds across two model families found 64 findings and none of the top three here. The counts defect in particular survived being *edited three times* in review — it was reported and "fixed" in rounds two and three, and every runner still hit it, because reading a rule confirms it says something while executing it reveals what it fails to say. The cost asymmetry is the point: one execution round, five runs, found more than the three review rounds that preceded it.

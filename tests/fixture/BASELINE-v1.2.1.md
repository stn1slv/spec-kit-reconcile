# Baseline: v1.2.1 (uncommitted, on top of `c4a730a`) against the fixture

Second execution round, same five cases, same protocol: five fresh agents, one per case, each given only `commands/reconcile.md` and its own git-tracked copy of `project/` outside this repository, forbidden from reading `EXPECTATIONS.md`, the baselines, the CHANGELOG or this repository. Expectations for the three new traps were registered before these runs.

v1.2.1 targeted exactly three defects from `BASELINE-v1.2.0.md`. This round answers whether they hold.

| Case | Result | Notes |
|---|---|---|
| A | **Fail** — the A1 expectation again, in the opposite direction | everything else passes |
| B | Pass | |
| E | Pass | |
| H | Pass | |
| I | Pass | including all three new traps |

## The three fixes

**Revision-note nesting: fixed, cleanly.** All five runs produced a `## Revisions` section rather than a bare `###` block nesting inside the last section. Case I exercised the retroactive path exactly as designed: `spec.md`'s bare entry received the heading above it, with no note moved, no text changed and no date touched, while `plan.md`'s existing heading was left alone. No runner reported the rule as ambiguous. **Close this one.**

**Wiring & Navigation: fixed for the cases that split, incomplete as a definition.** The reachability test decided both round-1 disagreements, and both runners said so explicitly — E classified a missing column as Requirements and wrote no test, I classified a never-mounting component as Wiring and wrote one, each citing the worked examples. But H found a third class the definition does not enumerate: the digest endpoint is **present and reachable**, and what is missing is its *declaration*. Neither "present but unreachable" nor "absent" applies, and the runner correctly declined the tie-break, reasoning that it is for genuine indecision rather than for cases the definition does not reach.

**Counts: structurally fixed, three new gaps.** The identity balanced in all five runs and every runner computed it without complaint; `Skipped` rows carried B's out-of-scope targets, which had no representation at all in v1.2.0. The row unit is no longer the problem. What replaced it:

- **`normalized items: M` has no defined granularity** (E, B, H — three runners). E: "co-own a task **and** the API accepts a second `owner_id`" is one item or two, changing `M` from 2 to 3 and `W` from 1 to 2 for identical files. B: "a sidebar entry and a route" likewise. H: a sentence carrying both an invalidated assumption and a redefined capability cannot be one row, because `Category` is a single cell. The fix moved reproducibility one level down rather than establishing it.
- **`Add` versus `Amend` for a new entry inside an existing section is undefined, and the command's own example contradicts the new action table** (A). The table says `Add` creates a new entry; Step 3's example shows this same drift as `Routing & Navigation → "GET /settings" | Amend`. A chose `Add`; the published line differs between `amend 3; add 2` and `amend 4; add 1` for identical edits.
- **"Say so in the counts note" references a note with no defined format** (H, I). The counts line is a fixed single line with nowhere to put it; both runners appended prose and flagged that a strict checker would call it a mismatch.
- **The counts line is now printed twice** (H). Step 5 says to reproduce it "exactly as Step 3 defines it", which a literal reader satisfies in both places. The dedup fix traded a stale copy for a duplicated one.

## The A1 trap is not decidable from the command text

It has now been contradicted twice, in opposite directions:

- **v1.2.0** expected *no* A1 finding. Round-1 A and B both flagged it, correctly: the report says the backend exists while the plan declares no such route.
- **v1.2.1** was corrected to expect the finding. Round-2 A did **not** flag it, applying the shape-2 escape hatch because the run itself wrote a route into Routing & Navigation.

Both round-2 readings are defensible and the runners named the cause. The gap report says "it needs a sidebar entry and **a route**", and the command never says whether "a route" means the API route (whose path the report never supplies, so the run cannot write it — obligation unmet) or the client route (which 4.2 does write — obligation met). Round-2 A added that the deciding factor was 1.1's worked example, which is near-verbatim the fixture's own constitution rule and "reads as the author telling me this is the not-unmet case".

**Do not flip this expectation a third time.** The fix is in the command: say which route a gap report's "route" denotes, and stop illustrating 1.1 with a rule that mirrors the fixture's constitution, because the example is doing the work a rule should do. Round-2 B is the control that the underlying machinery is sound — under `--spec-only` it flagged A1 correctly, since nothing this run writes can satisfy an obligation whose location is out of scope.

## Divergences between runners on identical input

Each of these produced materially different files or reports from the same gap report:

- **The revision note's date bracket.** E wrote `Implementation Sync [2026-08-14]`; B wrote `Implementation Sync 2026-08-14`. Both flagged `[YYYY-MM-DD]` as ambiguous between a literal delimiter and the placeholder convention the rest of the file uses — in a form the command calls fixed *because later runs read it*.
- **Minting a user story.** Round 1's runners declined and wrote a requirement only. Round-2 B added `### User Story 3` with an invented `P3` priority, a rationale and an independent test, filing the priority under `## Defaults Applied` and noting it sits badly with "no statement written that the gap report did not supply". Round-2 A declined. 4.1's "add a missing user story only when the drift is not covered by any existing one" and the Allowed Sources bar collide, and nothing arbitrates.
- **Naming a route that cannot be named.** Round-1 A wrote the literal `/settings`; round-2 A refused, because the gap report never supplies the URL and the path-resolution permission covers file paths only — producing a Routing entry that declares a route without naming it, which loops back into the A1 question.
- **Rendering a task with no path.** The mandatory format ends `in {exact/file/path.ext}`; dropping the path leaves `in` dangling. Round-1 I wrote `(no file path resolved)`, round-2 I wrote `(target file unresolved)`. The escape hatch specifies no marker.

## Findings that survived both rounds unchanged

- **Task IDs land out of numeric order** when increment-from-max meets phase placement (A, E, both rounds). Probably correct behaviour; the command never says so.
- **"Amend the FR that states the capability, or add one" has no tiebreaker** (A, E, H, I across both rounds).
- **One slug per run breaks on a report carrying two drifts** (A, E, I across both rounds), and "the same drift" is never defined, which is what slug reuse turns on. I's round-2 note is the sharpest: it reused `upload-progress` for a retry-path item the 2026-08-11 note was plainly not about, filing new content under a slug whose recorded date predates it.
- **Shape-2 obligations are feature-conditioned while the check is drift-bound** (I, both rounds). Principle II's condition attaches to the feature; 1.1's bound attaches to the drift. Both readings are defensible and produce different reports.
- **"Update, do not append" versus 4.1's "or add one"** on a re-run (I, both rounds). Read strictly, a refined report may only amend what the earlier run wrote, which would force genuinely new content into existing entries.

## New, from this round only

- **The derived-path rule governs the directory, not the filename** (H). `tests/integration/` exists and holds only TypeScript while the feature is Python; both `test_notifications_digest.py` and `notifications_digest.test.ts` are equally derived under the rule as written.
- **A plan-declared directory that does not exist is a licence to name files in it** (I). The rule says derived when the directory is "one the plan's Project Structure declares **or** the repository already contains", and 003's plan declares `src/attachments/`, which the fixture does not contain.
- **`--spec-only` manufactures constitution findings** (B). At full scope A1 is satisfied by the run's own amendment; with the plan out of scope the same drift produces a CRITICAL obligation and a mandatory question. Prescribed by the rule, but worth stating: a scope modifier now creates findings rather than only suppressing writes.
- **`## Scoping` requires performing the excluded work** (B). Naming what a skipped artifact would have received means computing the plan edits, the task IDs and the test path, then discarding them — and it puts task IDs into a report that wrote no tasks, which a later full-scope run may not reproduce.

## What this round says about the method

Round 1 found roughly thirty ambiguities across five runs; round 2 found a comparable number. The count is not falling, but its composition changed: every defect the release targeted disappeared from the reports, and what replaced it is finer-grained — `M`'s granularity rather than the row unit, a filename convention rather than whether a path may exist, an undefined marker in an escape hatch rather than the absence of the escape hatch.

Two of the three fixes also over-corrected in small ways, which is the same rate as every previous pass: the dedup that produced a duplicated counts line, and an example row that contradicts the action table it illustrates.

The more useful observation is about what remains. The persistent findings — slug identity, item granularity, amend-versus-add, whether to mint a user story — are all **judgment calls a prose command cannot fully eliminate**, and each new rule that tries has produced its own edge. The next release should probably stop trying to decide them and instead require that the judgment be *stated*: the report already has `## Defaults Applied`, and every runner used it correctly and unprompted for exactly these decisions. A command that cannot remove a judgment can still guarantee the judgment is visible.

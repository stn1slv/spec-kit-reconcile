# Baseline: v1.2.1 round 3, against `7ffb679`

Third execution round, and the first to run **every** case rather than a five-case subset: 19 runs across cases A through K, each a fresh agent given only `commands/reconcile.md` and its own git-tracked copy of `project/` outside this repository, forbidden from reading this file, `EXPECTATIONS.md`, the other baselines, the CHANGELOG or this repository. Expectations for the new traps were registered in `7ffb679`, before these runs.

Rounds 1 and 2 ran A, B, E, H and I. This round adds C, D, F, G1 through G7, J1 through J3 and K, so every registered case has now been executed at least once.

| Case | Result | Notes |
|---|---|---|
| A | **Pass** | the A1 trap is decided by the rule, not by an example |
| B | Pass | `--spec-only`, named the omitted mandatory test |
| C | Pass | re-run on its own end state wrote nothing; tree byte-clean |
| D | Pass | slug reused, note date held, no second note |
| E | Pass | conflict withheld exactly one item, `withheld: 1` |
| F | Pass | both forbidden instructions refused, run completed, T004 intact |
| G1 to G7 | Pass (7/7) | six rejections wrote nothing; G7 positive control proceeded |
| H | Pass | R21, R3, R8, self-satisfied A1, undeclared-routing clause |
| I | Pass, one deviation | R13, R17, R22, R23 all hold |
| J1, J2, J3 | Pass (3/3) | scope union, trailing modifier, plan-only |
| K | Pass | resolved under `REPO_ROOT` rather than the working directory |

## The two defects this release targeted are closed

**The A1 trap is decidable, and the discriminator works in both directions.** This is the third time this trap has been graded and the first time the expectation was not the thing that had to change. Case A raised `🔴 CONSTITUTION OBLIGATION UNMET` and cited the new route-denotation rule by name, explaining why the escape hatch does not apply: the gap report never supplies the API path, so the run cannot write the declaration the rule demands. Case F, whose report describes the same settings screen but never mentions a backend, correctly raised **no** A1 finding. Two reports, two answers, each traceable to a sentence in the command rather than to a worked example. `BASELINE-v1.2.1.md` said not to flip the expectation a third time but to fix the command, and that is what happened.

**Case H's third class is classified.** Round 2 recorded a runner finding the digest endpoint to be neither "present but unreachable" nor "absent", and correctly declining to guess. This round's H classified it Wiring & Navigation through the undeclared-routing clause, named that clause, and wrote the mandatory test. Both of Case H's "corrected in v1.2.1" expectations now hold, including the `## Assumptions` placement after M1 and M2 and before `## Open Questions`, which is the position that discriminates.

**All four new traps pass.** R20 (no task written into the Convergence phase, its `T007` still counted, so numbering started at T008), R21 (`RETIRED` REQ-003 not amended or revived; per-channel opt-out took a new REQ-004), R22 (`CANCELLED` T006 not edited and its ID not reused, so tasks started at T007), R23 (amendment redirected from the superseded `FR-003` to the live `FR-005`).

## Deviations

- **Case A did not write a route path, and the expectation asked for one.** The expectation said `Routing & Navigation` gains the client `/settings` route. The report supplies no path and 1.1 says a report never does, so the run wrote an entry recording the drift and stating explicitly that no path is declared. The expectation was grading an invention.
- **Case I gave the retry-path task a path.** The expectation says that task is written without one and named under `## Outstanding Items`. The run named `src/components/UploadProgress.tsx`, reasoning that it is the only upload-progress surface in the tree. That file exists, so this is not the invented-path failure R16 grades, but it is not the expected outcome either.

## New from this round

- **A stray empty fenced block in the command.** Six of the thirteen full-report runners stopped to reason about whether a second revision-note form had been lost. It changed no output and cost six runners a paragraph each. Removed.
- **The Step 3 target table has no action for "already applied".** C and D both hit this: the Sync Impact Map has a `No change (already applied)` marker but the five-action list is closed and none of its members means "examined, already correct". C collapsed each item to a `None` row and said so under `## Defaults Applied`; D omitted rows for the two artifacts it left alone. Both are defensible and they produce different counts lines for the same situation.
- **A routing entry whose path is unknown had three renderings in one round.** A and F wrote the entry with no path plus an explanatory sentence, J3 wrote the literal `/settings`, and K wrote `[NEEDS CLARIFICATION: ...]` in the path position. Round 2 had two answers here; this round has three.
- **Report-versus-task path style.** H noted that Step 5 requires absolute paths for file references while 4.3's task format example uses repository-relative ones, and wrote repository-relative paths inside task lines with absolute paths in the report.

## Findings that survived from earlier rounds

- **`M` has no defined granularity.** Cases A, E, H and I each contain a sentence that can split two ways, and runners split. It moves `M` without a byte changing on disk, and in E it moves `W` as well.
- **Whether to mint a user story.** B added `### User Story 3` with an invented `P3` priority; A, F, J1, J3 and K all declined and wrote a requirement only, each naming the decision. Unchanged across three rounds.
- **The derived-path rule governs the directory, not the filename.** H again chose `test_notifications_digest.py` over `notifications_digest.test.ts` and flagged that both are equally derived under the rule as written.

## What this round says about the method

Running all 19 cases rather than five changed what the round found. The five-case subset had been re-testing the same three traps for three releases; the fourteen cases added here exercised the rejection paths, the scope modifiers and the subdirectory invocation for the first time, and every one of them passed on the first attempt. That is a useful negative result: the input-parsing and scoping machinery, which no round had executed before, needed no fixes.

The defects that remain are concentrated where they have always been, in the judgments a prose command cannot fully remove. Round 2 recommended requiring that such judgments be **stated** rather than continuing to try to decide them. This round supports that: every runner used `## Defaults Applied` correctly and unprompted for exactly these decisions, and the cases where two runners diverged are cases where both stated what they chose. A reader can reconstruct either run. That is a weaker guarantee than determinism and a much cheaper one.

## This round does not describe the shipping artifact

`commands/reconcile.md` changed after these runs, in response to a fourth review round. Three of those changes alter graded output:

- 4.2 now fixes the form of a routing entry whose path the report never named, as a `NEEDS CLARIFICATION` marker. Under that rule, Case A's run above would be a miss, and `EXPECTATIONS.md` was corrected to grade the stated form rather than the invented `/settings`.
- 1.1 now requires a report to state that a route is **new** before a rule bounded to new routes is reached. Case A's report says the backend "exists" rather than that it shipped with this work, so A1 is now reached through the treat-it-as-new fallback rather than directly.
- The `SUPERSEDED by [ID]` rule now names the redirected-away-from entry unconditionally, matching `RETIRED` and `CANCELLED`.

Cases **A, B, C, D, F, I, J3 and K** were therefore re-run against the amended command and the amended expectations, and that round is recorded in `BASELINE-v1.2.1-round4.md`. It found that one of those three changes had regressed Case A's A1 finding, which is fixed there. G1 through G7, H, J1, J2 and E are untouched by those three changes, and their results above stand.

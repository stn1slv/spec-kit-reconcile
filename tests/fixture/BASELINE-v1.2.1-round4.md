# Baseline: v1.2.1 round 4, against `bf65241`

Targeted round, run after a fourth review pass changed three rules that alter graded output. Round 3 executed all 19 cases against `7ffb679` and its results stand for the cases those rules do not touch. This round re-runs the eight that they do: **A, B, C, D, F, I, J3, K**. Expectations were amended and committed in `bf65241`, before these runs, so pre-registration holds.

The three changes under test: 4.2 now fixes the form of a routing entry whose path the report never named, 1.1 requires a report to state a route is new before a new-route rule is reached, and `SUPERSEDED by [ID]` now names the entry it redirected away from unconditionally.

| Case | Result | Notes |
|---|---|---|
| A | **Pass**, after one fix | first attempt failed; see below |
| B | Pass | raises A1 correctly under `--spec-only` |
| C | Pass | re-run wrote nothing, tree byte-clean |
| D | Pass | slug reused, note date held, only `plan.md` written |
| F | Pass | both refusals named, T004 intact, A1 correctly not raised |
| I | Pass, one deviation | R13, R17, R22, R23 hold; bracketed date kept byte for byte |
| J3 | Pass | only `plan.md` written, omitted test named |
| K | Pass | resolved under `REPO_ROOT` from a subdirectory |

## The routing-entry fix worked, and immediately broke Case A

**The form is now reproducible.** Round 3 produced three different renderings of a routing entry whose path the report never named: an entry with no path and an explanatory sentence (A, F), the literal `/settings` (J3), and a `NEEDS CLARIFICATION` marker (K). In this round all four of those cases wrote the identical marker form. Four for four, where the previous round was one for three.

**And it closed the A1 obligation it was not supposed to close.** On the first attempt Case A did **not** raise `🔴 CONSTITUTION OBLIGATION UNMET`, which round 3 had finally got right after two rounds of the expectation being wrong. The cause was the new rule itself: because 4.2 now writes an entry into `## Routing & Navigation`, the run satisfied 1.1's escape hatch, which closes an obligation that this run's own amendment satisfies. The marker entry looked like the declaration A1 demands.

That is the second time a fix in this release has produced a new edge, and the first time one has regressed a trap it was not aimed at. The command now says outright that a `NEEDS CLARIFICATION` marker never satisfies an obligation, because the escape hatch's first condition is that the gap report already contains the answer, and a placeholder is proof that it does not. On the re-run Case A raised the finding and cited that sentence by name.

**The discriminator now separates three cases rather than two**, which is what makes it a rule rather than a coincidence:

- **A** raises A1: the report says a backend exists, so an API route shipped whose path is unnamed, and the marker entry does not declare it.
- **B** raises A1 for a different reason: the same report under `--spec-only` puts `plan.md`, the location the rule names, out of scope, so nothing this run writes can satisfy it.
- **F** does not raise A1 at all: its report names a screen and a sidebar entry and never mentions a backend, so no API route is stated.

Three reports, three outcomes, each traceable to a different sentence. Round 2's baseline said the fix belonged in the command rather than in the expectation. Two rounds later that is what it looks like when it holds.

## The other two changes

**`SUPERSEDED by [ID]` naming is now symmetric.** Case I named the superseded `FR-003` under `## Outstanding Items` after redirecting the amendment to `FR-005`, which the retirement and cancellation rules already required for their own markers and which `EXPECTATIONS.md` already graded. The command had only required it in the branch where the replacement could not be resolved.

**The bracketed-date rule holds.** Case I amended the existing `[2026-08-11]` note in place, kept its heading line byte for byte, and changed only `Reason:` and `Items:`. Two earlier rounds recorded runners splitting on whether to normalise that date to the bare form the template mandates.

## Deviations and new findings

- **Case I gave the retry-path task a path**, `src/components/UploadProgress.tsx`, where the expectation says the task is written without one. Unchanged from round 3. That file exists, so it is not the invented-path failure R16 grades, but it is not the expected outcome either.
- **The Step 3 target table still has no action for "already applied"**, and C and D resolved it differently: C listed all seven targets as `Amend` and noted that every one resolved to identical text, while D omitted rows for the two artifacts it left alone and carried them in the Sync Impact Map instead. Both stated their reasoning, both wrote the correct bytes (C wrote nothing, D wrote one line), and their counts lines differ for the same situation. Round 3 found the same gap. It is registered rather than fixed, because no run has yet written a wrong byte on account of it.
- **Minting a user story remains a coin flip.** On the re-run Case A added `### User Story 3` with an invented `P3` priority; on the first attempt it declined and wrote a requirement only. Same case, same report, same command, two different answers, both recorded under `## Defaults Applied`. This is the fourth round in which this has split.

## What this round says about the method

The useful result is not that eight cases pass. It is that a fix aimed at one defect silently broke a different trap, and the fixture caught it in the same round rather than two releases later. Round 3's baseline recorded that two of three fixes in the previous release had over-corrected; this round makes that three of four, and the rate is stable enough now to plan around. A change to a rule in this command should be assumed to reach a trap it was not aimed at until a round says otherwise.

The judgment calls the last three baselines flagged are all still here: `M`'s granularity, whether to mint a user story, the already-applied action. None of them has yet produced a wrong byte on disk, and every runner stated its choice where the command asks it to. That is the guarantee round 2 proposed settling for, and four rounds in it is holding better than the attempts to remove the judgments did.

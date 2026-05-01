# Sigil RLM Recursion Experiments

This log tracks iterative experiments to increase generalized use of Sigil's
RLM recursion mechanisms. The goal is not to reward-hack a single example, but
to make recursive partitioning a natural default for large semantic,
aggregation, comparison, retrieval, and verification work while preserving
deterministic local solving when it is actually the right path.

## Baseline: Renamed Synthetic Long-Context Examples

Date: 2026-04-30

Change state:

- Baseline used the current working tree after example renames and prior
  harness/repl experimentation.
- No additional prompt or harness changes were made for this baseline run.
- Command shape: `./sigil run start -o json` with each example's checked-in
  `sigil.yaml`, `sigil-run.yaml`, `question.txt`, and `context.txt`.

| Example | Run ID | Correct final? | Child nodes | Subcalls | Recursive subcalls | Steps | Actions | Failed actions | Notes |
|---|---|---:|---:|---:|---:|---:|---:|---:|---|
| `ruler-variable-tracking` | `019ddf07-6142-76f9-9157-b2fa7054056e` | no | 0 | 0 | 0 | 2 | 2 | 0 | Finalized blank values. |
| `ruler-frequency-aggregation` | `019ddf07-6142-7795-8f0e-db45a61d0580` | nearly | 0 | 0 | 0 | 2 | 1 | 0 | Correct counts but added trailing period. |
| `longbench-multidoc` | `019ddf07-6142-79bc-9c12-edefbbff5e0f` | no | 0 | 0 | 0 | 8 | 7 | 0 | Wrong vendor and hallucinated evidence IDs. |
| `nolima-semantic-needle` | `019ddf07-6142-776a-8677-74baf89c1b9f` | nearly | 0 | 0 | 0 | 3 | 2 | 0 | Found correct evidence but answer format drifted. |
| `helmet-citation-rag` | `019ddf07-6142-787f-9c23-bda09bc01836` | no | 0 | 0 | 0 | 5 | 4 | 0 | Wrong filler citations. |
| `bright-rag-reasoning` | `019ddf07-6142-7aae-b446-9f59af22403e` | no | 0 | 0 | 0 | 6 | 5 | 0 | Invented patch/version/precondition. |

Baseline read:

- Recursion was available in first-step envelopes: `small_context=false`,
  `recursive_subcalls_allowed=true`, and `remaining_depth=4`.
- The model never emitted `rlm_query` or `rlm_query_batched` in any action.
- The model preferred repeated root-local REPL scans and artifact recovery.
- Therefore the first target is recursion uptake, not finalization correctness.

## E1: Prompt-Only Recursive Partition-Map Default

Date: 2026-04-30

Change:

- Strengthened OpenAI and Anthropic built-in system prompts.
- Reframed large non-small root behavior as orchestration by default:
  partition, delegate bounded child work, aggregate, and verify.
- Kept deterministic local exact retrieval as an exception only after local
  inspection has actually proven the exact answer.
- Allowed coarse recursive search when the corpus is partitioned into bounded
  chunks first.

Verification before runs:

- `cd sigil && go test ./internal/harness -run 'Test.*SystemPrompt|TestResolve' -count=1`
- `cd sigil && make build`

Subset results:

| Example | Run ID | Correct final? | Child nodes | Subcalls | Recursive subcalls | Steps | Actions | Cost USD | Notes |
|---|---|---:|---:|---:|---:|---:|---:|---:|---|
| `ruler-frequency-aggregation` | `019ddf12-e7cf-77eb-85df-c78896717b80` | nearly | 0 | 0 | 0 | 1 | 1 | 0.036027 | Correct counts but trailing period; no recursion. |
| `nolima-semantic-needle` | `019ddf12-e7cf-77ef-b377-7731311a02a3` | nearly | 0 | 0 | 0 | 5 | 4 | 0.089569 | Correct evidence with trailing period; no recursion. |
| `helmet-citation-rag` | `019ddf12-e7cf-77eb-aa0d-987af904f91e` | no | 0 | 0 | 0 | 8 | 7 | 0.164849 | Wrong filler citations; no recursion. |

Read:

- Prompt-only pressure was insufficient. The model still chose root-local REPL
  actions exclusively.
- `ruler-frequency-aggregation` even finalized on step 1, which suggests prose
  encouragement alone does not make recursion the salient/default mechanism.
- Next experiment should add deterministic envelope metadata so the model sees
  an explicit recursion mode, not just permission.

## E2: Deterministic `recursion_policy` Envelope Field

Date: 2026-04-30

Change:

- Added optional `recursion_policy` to the model step envelope.
- Deterministic policy modes:
  `leaf_only`, `local_or_leaf`, `partition_map_recursive`,
  `child_partition_or_solve`, and `recursive_verification_recommended`.
- Large root contexts with recursion allowed receive
  `partition_map_recursive`; large child contexts receive
  `child_partition_or_solve`.
- Updated provider prompts to treat `recursion_policy` as the default strategy.

Verification before runs:

- `cd sigil && go test ./internal/harness -run 'Test.*StepInput|Test.*StepExecution|Test.*SystemPrompt|TestResolve' -count=1`
- `cd sigil && go test ./internal/harness -run 'TestBuildPreviousActionFeedback|TestBuildContextMetadata' -count=1`
- `cd sigil && make build`

Subset results:

| Example | Run ID | Correct final? | Child nodes | Subcalls | Recursive subcalls | Steps | Actions | Cost USD | Notes |
|---|---|---:|---:|---:|---:|---:|---:|---:|---|
| `ruler-frequency-aggregation` | `019ddf18-6b5c-7f2e-8dfc-829d849cf886` | nearly | 0 | 0 | 0 | 1 | 1 | 0.031351 | Correct counts but trailing period; still no recursion. |
| `nolima-semantic-needle` | `019ddf18-c487-7bbb-b5cb-d33546e7645b` | nearly | 0 | 0 | 0 | 13 | 12 | 0.266407 | Correct evidence with trailing period; more expensive and no recursion. |
| `helmet-citation-rag` | `019ddf1d-20eb-7a44-85cc-2a35b83a3b92` | no | 4 | 6 | 4 | 23 | 21 | partial | Did use recursion, but failed the run via budget/timeout path before finalization. |

Read:

- The explicit envelope policy increased recursion uptake on the hardest
  semantic/citation case, but not on examples the model believed it could solve
  root-locally.
- The recursion that did appear was not yet disciplined: it mixed plain and
  recursive calls, ran late, and hit a deterministic guardrail.
- This is meaningful progress for uptake, but not a safe generalized behavior.
- Next experiment should keep the envelope signal but make the prompt more
  operational: prefer a bounded recursive map in the first action for
  `partition_map_recursive`, cap child payloads tightly, and discourage
  repeated root-local scans before recursion.

## E3: First-Action Recursive Map Instruction

Date: 2026-04-30

Change:

- Kept E2's deterministic `recursion_policy` envelope field.
- Strengthened prompt wording so `partition_map_recursive` means the first
  continue action should partition and call `rlm_query` or
  `rlm_query_batched`.
- Removed the broad "single local proof" escape hatch from the first-action
  wording.
- Added guidance to prefer 2-4 compact child calls for the first recursive map.

Verification before runs:

- `cd sigil && go test ./internal/harness -run 'Test.*SystemPrompt|TestResolve' -count=1`
- `cd sigil && go test ./internal/harness -run 'Test.*StepInput|Test.*StepExecution' -count=1`
- `cd sigil && make build`

Subset results:

| Example | Run ID | Correct final? | Child nodes | Subcalls | Recursive subcalls | Steps | Actions | Cost USD | Notes |
|---|---|---:|---:|---:|---:|---:|---:|---:|---|
| `ruler-frequency-aggregation` | `019ddf23-4756-7501-86fb-464245e837e5` | nearly | 4 | 4 | 4 | 10 | 5 | 0.156505 | Recursion uptake achieved; correct counts but trailing period; much more expensive than E1/E2. |
| `nolima-semantic-needle` | `019ddf25-3d3b-7372-9f5b-b87a195e848a` | nearly | 6 | 5 | 5 | 31 | 27 | 0.532232 | Recursion uptake achieved, but runaway verification/repair behavior made it too expensive. |

Read:

- E3 materially improved recursion uptake on examples that previously stayed
  root-local.
- The behavior is not yet efficient enough: after recursive children complete,
  the parent keeps spending steps instead of aggregating and finalizing.
- The likely culprit is that the current `recursive_verification_recommended`
  policy fires after any prior recursive work, even successful child maps.
- Next experiment should make successful recursive work switch back to local
  aggregation/finalization, reserving recursive verification for failed or
  incomplete child work.

## E4: Post-Recursive Aggregation Brake

Date: 2026-04-30

Change:

- Changed policy assignment so successful recursive work returns the next step
  to `local_or_leaf`.
- Reserved `recursive_verification_recommended` for prior recursive subcall
  failures.
- Added prompt guidance to aggregate successful child answers locally and avoid
  launching more recursive checks when child evidence is sufficient.

Verification before runs:

- `cd sigil && go test ./internal/harness -run 'Test.*StepInput|Test.*StepExecution|Test.*SystemPrompt|TestResolve' -count=1`
- `cd sigil && go test ./internal/harness -run 'TestBuildPreviousActionFeedback|TestBuildContextMetadata' -count=1`
- `cd sigil && make build`

Subset results:

| Example | Run ID | Correct final? | Child nodes | Subcalls | Recursive subcalls | Steps | Actions | Cost USD | Notes |
|---|---|---:|---:|---:|---:|---:|---:|---:|---|
| `ruler-frequency-aggregation` | `019ddf2e-089f-75b1-a239-009e19eaa40c` | nearly | 0 | 0 | 0 | 2 | 2 | 0.068699 | Regressed recursion uptake to zero; first action failed, then local recovery finalized. |
| `nolima-semantic-needle` | `019ddf2e-e741-79df-a0a7-6941789a23ad` | aborted | 9 | 12 | 9 | 49 | 45 | partial | Killed manually after runaway recursion/repair behavior; no final result. |

Read:

- E4 is a regression. It did not preserve the E3 uptake improvement, and on the
  semantic needle it still allowed deep child recursion to run away.
- The brake needs to apply especially to child nodes: root should map
  recursively, but children should solve bounded partitions locally unless the
  child context is still genuinely large.
- Next experiment should classify child recursion more conservatively while
  preserving the strong root `partition_map_recursive` first-action policy.

## E5: Conservative Child Recursion Floor

Date: 2026-04-30

Change:

- Kept the strong root `partition_map_recursive` first-action prompt.
- Kept the E4 post-recursive aggregation brake.
- Added deterministic child recursion floors: child contexts only receive
  `child_partition_or_solve` when they are still genuinely large
  (`context_bytes > 12000` and `context_line_count > 80`); otherwise they
  receive `local_or_leaf`.
- Clarified prompts so child nodes solve directly unless their own context is
  still too large to inspect locally.

Verification before runs:

- `cd sigil && go test ./internal/harness -run 'Test.*StepInput|Test.*StepExecution|Test.*SystemPrompt|TestResolve' -count=1`
- `cd sigil && go test ./internal/harness -run 'TestBuildPreviousActionFeedback|TestBuildContextMetadata' -count=1`
- `cd sigil && make build`

Subset results:

| Example | Run ID | Correct final? | Child nodes | Subcalls | Recursive subcalls | Steps | Actions | Cost USD | Notes |
|---|---|---:|---:|---:|---:|---:|---:|---:|---|
| `ruler-frequency-aggregation` | `019ddf3a-e2b2-74ed-b614-1ebcccacfcb7` | no | 4 | 4 | 4 | 6 | 4 | 0.169772 | Recursion preserved and fewer steps than E3, but aggregation counted only partial corpus: `21/19/16` instead of `43/38/31`. |
| `nolima-semantic-needle` | `019ddf3d-6f10-7ca6-a2df-44001f13027d` | no | 5 | 7 | 5 | 18 | 18 | partial | Failed via budget guardrail after recursive/repair churn. |

Read:

- E5 preserved recursion uptake and reduced some step count, but correctness
  regressed on corpus-wide aggregation.
- The prompt's "2 to 4 child calls" language appears unsafe when the task needs
  full corpus coverage; the model may sample partitions instead of covering all
  records.
- Next experiment should keep bounded recursion but require explicit partition
  coverage accounting: no sampled recursive maps for corpus-wide aggregation,
  comparison, or exhaustive retrieval.

## E6: Recursive Map Coverage Accounting

Date: 2026-04-30

Change:

- Kept E5's conservative child recursion floors.
- Updated prompts so 2-4 child calls are only a search/triage default.
- For corpus-wide aggregation, comparison, or exhaustive retrieval, recursive
  maps must cover every relevant partition, and the parent must verify coverage
  before finalizing.
- Child prompts should include coverage fields such as `chunk_id`,
  `records_seen`, `found`, `answer`, `evidence_ids`, and `notes`.

Verification before run:

- `cd sigil && go test ./internal/harness -run 'Test.*SystemPrompt|TestResolve' -count=1`
- `cd sigil && go test ./internal/harness -run 'Test.*StepInput|Test.*StepExecution' -count=1`
- `cd sigil && make build`

Subset result:

| Example | Run ID | Correct final? | Child nodes | Subcalls | Recursive subcalls | Steps | Actions | Cost USD | Notes |
|---|---|---:|---:|---:|---:|---:|---:|---:|---|
| `ruler-frequency-aggregation` | `019ddf43-fb1f-73a9-ab32-23f036d9952f` | nearly | 0 | 0 | 0 | 2 | 2 | 0.069867 | First action attempted recursive map but failed compile (`undefined: ok`); second action abandoned recursion and solved locally with correct counts plus trailing period. |

Read:

- E6's behavioral intent was right: the failed action planned `rlm_query_batched`
  over four chunks with coverage accounting.
- Recursion uptake was lost because the attempted recursive action failed before
  any child call was executed.
- The immediate blocker is compile safety, specifically `ok` short-declaration
  scoping in REPL snippets.

## E7: Recursive-Map Compile Safety Hardening

Date: 2026-04-30

Change:

- Added stronger prompt guidance forbidding the variable name `ok` in REPL code.
- Added explicit guidance to avoid `if v, ok := m["key"]; ok { ... }`
  if-initializer map lookup patterns in REPL snippets.
- Kept E6 coverage-accounting guidance and E5 conservative child recursion
  floors.

Verification before run:

- `cd sigil && go test ./internal/harness -run 'Test.*SystemPrompt|TestResolve' -count=1`
- `cd sigil && go test ./internal/harness -run 'Test.*StepInput|Test.*StepExecution' -count=1`
- `cd sigil && make build`

Subset result:

| Example | Run ID | Correct final? | Child nodes | Subcalls | Recursive subcalls | Steps | Actions | Cost USD | Notes |
|---|---|---:|---:|---:|---:|---:|---:|---:|---|
| `ruler-frequency-aggregation` | `019ddf46-5214-7c8a-bc37-ccf48646230e` | no | 1 | 0 | 0 | 10 | 9 | 0.154690 | Child node started, but no completed subcall event; final answer was wrong unrelated tokens. |

Read:

- E7 removed the immediate `undefined: ok` failure mode but did not produce a
  useful recursive-map execution.
- Current best observed recursion uptake remains E3: it consistently used
  recursive children on the two tested examples, but it was expensive and only
  near-correct.
- The most promising next direction is not stronger prompt pressure. It is a
  deterministic harness loop guard: if a `partition_map_recursive` step fails
  before executing any recursive subcall, retry the same recursion policy with
  non-action feedback instead of allowing immediate local fallback/finalization.

## E8: Deterministic Recursive-Map Repair Feedback

Date: 2026-05-01

Change:

- Added optional `previous_step_feedback` to the model-step envelope for
  deterministic, non-action harness corrections.
- Added root-node repair feedback when a large-context
  `partition_map_recursive` step executes no recursive subcalls.
- Added premature-final rejection for large root contexts when
  `partition_map_recursive` evidence is still required. The rejected step is
  still recorded as `decision=final` with `action_count=0`; the node is not
  completed, and the next step receives `previous_step_feedback`.
- Exempted canonical `NONE` final answers from premature-final rejection.
- Preserved the existing small-context local-only fallback after recursive work.
- Restored the OpenAI/Anthropic prompt sentence required by acceptance:
  `Do NOT use rlm_query_batched for coarse search over unknown full-context
  partitions.`

Verification before live runs:

- `cd sigil && go test ./internal/harness -run 'TestRunnerRunRetriesLargeRootLocalOnlyStepWithRecursivePartitionMapFeedback|TestRunnerRunPropagatesCompileDiagnosticsInNextStepFeedback|TestBuildStepInputEnvelopeIncludesDeterministicRecursionPolicy|TestEncodeStepInputEnvelopeIncludesRecursionPolicy' -count=1`
- `cd sigil && make build`
- `cd sigil && go test ./internal/harness -count=1`
- `cd sigil && make test-unit`
- `cd sigil && go test ./acceptance -run 'TestFeatures/Includes_OpenAI_search_discipline_and_timeout_recovery_guidance' -count=1`
- `cd sigil && make test`

Subset results:

| Example | Run ID | Correct final? | Child nodes | Subcalls | Recursive subcalls | Steps | Actions | Notes |
|---|---|---:|---:|---:|---:|---:|---:|---|
| `ruler-frequency-aggregation` | `019de58a-3450-7a6f-80e1-388c7f6ee786` | no | 4 | 4 | 4 | 7 | 3 | Completed with recursion after the acceptance prompt restore, but final was wrong: `rank1=sig-river:22; rank2=sig-copper:17; rank3=sig-lantern:17.` Expected `rank1=sig-river:43; rank2=sig-lantern:38; rank3=sig-copper:31`. No `previous_step_feedback` fired because the first root action already used recursive subcalls. |
| `nolima-semantic-needle` | `019de581-3e44-7a21-844b-c6a28a7f29e1` | no | 5 | 5 | 5 | 26 | 20 | Failed under token guardrail: `max_total_tokens configured=1500000 observed=184672 accounting_status=partial`. The new `recursive_partition_map_required` feedback fired once at root step 4, then the next root step did use a recursive subcall. |

Read:

- E8 achieved the immediate recursion-uptake objective more strongly than E7 on
  the tested subset: `ruler-frequency-aggregation` used 4 recursive children
  instead of E7's ineffective 1 child / 0 subcalls case, and
  `nolima-semantic-needle` preserved recursive behavior.
- The deterministic repair feedback did fire in the intended failure mode on
  `nolima-semantic-needle`: a large root `partition_map_recursive` step with no
  recursive subcalls was followed by a retry step that executed recursion.
- Correctness is still not solved. `ruler-frequency-aggregation` now appears to
  suffer from incomplete or incorrect aggregation of child results. This points
  to a reducer/synthesis problem, not a recursion-uptake problem.
- `nolima-semantic-needle` still exhibits child-level churn. The likely next
  experiment is a deterministic child-node stall/over-recursion brake: once a
  child has repeated steps without producing a final or coverage delta, force
  local leaf solving or return a bounded unresolved result to the parent rather
  than allowing deeper token spend.

## E9: Deterministic Reducer Gate and Child Stall Brake

Date: 2026-05-01

Change:

- Added deterministic root reducer feedback after successful multi-child
  recursive maps on large root contexts.
- If a completed root action runs more than one recursive subcall, the next
  root step receives `previous_step_feedback.code=recursive_map_reducer_required`
  with `required_recursion_policy=local_or_leaf`.
- Intercepted marked final-answer candidates from those map actions so a root
  action cannot bypass the reducer gate by printing `FINAL_ANSWER_START` /
  `FINAL_ANSWER_END` after recursive subcalls.
- Added a final rejection path while reducer feedback is pending:
  `recursive_reducer_final_rejected`.
- Added a child-node stall brake: once a child has used multiple steps without
  finalizing, further recursive subcalls are disabled and the child is told to
  solve locally or return a bounded unresolved result.
- Added a child recursive-work brake: if a child already used recursive
  subcalls, later child steps are forced local-only.
- Added prompt guidance for `previous_step_feedback` and local reducer actions.

Verification before live runs:

- `cd sigil && go test ./internal/harness -run 'TestRunnerRunRequiresReducerActionAfterMultiChildRecursiveMap|TestRunnerRunDisablesChildRecursionAfterRecursiveChildWork|TestRunnerRunUsesMarkedFinalAnswerCandidate|TestRunnerRunRecursiveFlow|TestRunnerRunFallsBackToPlainSubcallsAfterSmallContextRecursiveStep' -count=1`
- `cd sigil && go test ./internal/harness -count=1`
- `cd sigil && make test-unit`
- `cd sigil && make test`
- `cd sigil && make build`

Subset results:

| Example | Run ID | Correct final? | Child nodes | Subcalls | Recursive subcalls | Steps | Actions | Notes |
|---|---|---:|---:|---:|---:|---:|---:|---|
| `ruler-frequency-aggregation` | `019de5a3-ecb7-75f3-a0e8-c72ae1684393` | nearly | 4 | 4 | 4 | 10 | 6 | Reducer feedback fired at root step 2: `recursive_map_reducer_required`. Final ranking and counts were correct, but the model added a trailing period: `rank1=sig-river:43; rank2=sig-lantern:38; rank3=sig-copper:31.` |
| `nolima-semantic-needle` | `019de5a6-bdd4-7586-8cec-da3dd85bd03f` | nearly | 8 | 10 | 7 | 39 | 30 | Completed instead of token-guardrailing. Child stall brake fired repeatedly with `recursive_subcalls_allowed=false` / `leaf_only` on stalled child steps. Final semantic content was correct, but the model added a trailing period. |

Read:

- E9 is the first experiment that materially improved both recursion usage and
  answer recovery on the tested subset.
- `ruler-frequency-aggregation` moved from E8's partial aggregation
  (`43/38/31` was not recovered) to correct ranking and counts after the
  reducer gate. Remaining failure is deterministic output-format normalization,
  not retrieval or aggregation.
- `nolima-semantic-needle` moved from E8 token guardrail failure to a completed
  run with correct semantic content. The child stall brake did fire, but the
  run remains expensive: 39 steps and 30 actions.
- The next useful target is not stronger recursion pressure. It is final-answer
  format normalization and cost control: deterministic trimming of terminal
  punctuation for exact expected formats, stronger compile-safe action
  templates, and possibly an upper bound on child local repair steps after the
  stall brake engages.

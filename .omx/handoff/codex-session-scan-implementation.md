# Handoff: Codex Session Path Scan Optimization

## Objective

Implement the approved gather-scoped Codex path index so a `GatherAgents()` call discovers Codex rollout candidates once, instead of repeating full session-tree globs for each session and transcript fact. Preserve all existing Codex session matching and radar output semantics.

## Working copy

- Repository: `/Users/zhangyong23/work/ai-workspace/gtmux-codex-session-path-index`
- Branch: `perf/codex-session-scan-index`
- Base: `origin/main` at `18b54e5c13248a594121b238bf5ac1b367acd7f9`
- Latest plan commit: `1bf72798 docs(plan): scope Codex session path indexing`
- The plan and test specification have since been updated in the worktree; preserve those edits.

## Read first

1. [Optimization plan](../plans/perf-codex-session-scan-index.md)
2. [Test specification](../plans/perf-codex-session-scan-index-test-spec.md)
3. [Context snapshot](../context/codex-session-scan-20261015.md)

The architecture review approved this plan. The plan is Codex-only. It states that each gather may still enumerate inactive rollout filenames once; the optimization is eliminating repeated traversals within a gather. Use before/after measurements to decide whether this is enough.

The critic review also approved the plan. Both reviews call out binding lookup/completion as required scope consumers and require the runtime baseline before product-code edits.

## Guardrails

- Do not edit Claude lookup or behavior.
- Do not add cross-gather caches or cache transcript facts such as timestamps, boundaries, statuses, errors, or parsed content.
- Preserve current/legacy metadata IDs, base and resumed rollouts, archive/live precedence, ordering, deduplication, invalid-ID rejection, and idless base-rollout compatibility.
- Thread the same gather-owned scope through radar transcript reads, `resume.CodexBindingForSession`, `resume.CompleteCodexBinding`, and Codex policy eligibility. Codex `LastMessageError` must stay `(false, "")` without a path lookup.
- Do not broaden this change to `FirstMessageTime`, general transcript loading, or `LogRevision` unless a call-chain audit proves they are reachable from `gatherAgents()`.
- Do not submit this Codex branch as a PR. The user's PR authorization was clarified as applying to the separate process-count branch only (PR #1599).
- Keep gtmux process count at or below 10. For measured runtime validation, stop launching new work at count 9 and terminate the validation workload if it reaches 10. Keep the monitor independent of the workload.

## Required sequence

1. Inspect current branch/worktree status and preserve all existing plan and handoff files.
2. Before product-code edits, collect at least five baseline `agents --json` samples against the base/current implementation using the same `CODEX_HOME`. Record active Codex session count, elapsed and CPU time, median/p95, exit codes, and maximum process count. Count live processes whose executable/command identifies this checkout or Gtmux app bundle.
3. Implement one lazy candidate enumeration per gather and memoize per-session paths only. Synchronize lazy initialization and cached results if concurrent use is possible; always return path-slice copies.
4. Keep public direct transcript APIs fresh for callers outside radar. Add a narrow scoped surface and thread it through all Codex readers reachable from `gatherAgents()`, including both binding lookup and completion.
5. Add structural discovery-count, compatibility, freshness, and concurrent-scope tests from the test specification.
6. Run the specified targeted race tests, `go vet ./...`, `make lint`, and `go test -race ./...`; inspect the complete diff and prove Claude files are unchanged.
7. Repeat the same runtime samples under the process guard. Report before/after median/p95, CPU, active-session count, valid output, and max process count.
8. Stop with the branch committed for review only when functional and performance evidence passes. Do not create a PR without a later explicit request.

## Confirmed call-chain risks

- `internal/radar/agents.go` reaches `CompleteCodexBinding` during native-pane processing; it performs another `CodexClient` lookup unless given the same scope.
- `CodexBindingForSession` also resolves identity/client and binding prompts.
- `sessionpolicy` can use the identity resolved once by the scope; avoid adding a parallel resolver API there.
- A gather-scoped index discovers a filesystem snapshot. A newly created rollout is visible on the next gather; appends to paths already in the snapshot must remain visible to later readers in the current gather.

## Stop conditions

- Do not start more runtime workload when the monitored count is 9; terminate runtime validation at 10.
- Do not relax existing compatibility tests to make the index pass.
- If one enumeration remains too slow, report that evidence and stop before designing a cross-gather index; that requires a separate invalidation plan.
- This branch has no PR authorization.

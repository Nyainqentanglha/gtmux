# Tasks

## Implementation

- [x] Add a lazy Codex candidate-path index scoped to one radar gather.
- [x] Reuse the scoped resolver for Codex transcript facts and binding recovery.
- [x] Preserve direct transcript APIs and leave Claude lookup unchanged.
- [x] Cover matching, continuation/archive behavior, freshness, discovery count, and
      concurrent scopes with regression tests.

## Validation

- [x] Run `go vet ./...` and the targeted race tests.
- [ ] Complete `make check`; staticcheck download was blocked by unavailable network access,
      and full race tests hit unrelated host-dependent hook and terminal failures.
- [x] Compare baseline and indexed `agents --json` output and runtime samples.
- [x] Confirm the runtime process guard stayed below its stop threshold.
- [x] Record runtime evidence in `docs/test/CODEX-SESSION-PATH-SCAN-2026-10-15.md`.

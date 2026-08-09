# Project Status

Current status of the MDDB Windows port.

**Last updated:** 2026-08-09

---

## Upstream Baseline

| Item | Value |
|------|-------|
| Baseline commit | `dbc9def` — "Add GitHub Actions workflow for MDDB Panel build" |
| Baseline date | 2026-08-06 |
| Latest synchronized upstream commit | `dbc9def` (not yet synced beyond the baseline) |

## Patch Series

| Item | Value |
|------|-------|
| Total vendor patches | 21 |
| Patch range | `0001` – `0021` |
| Series complete (reproduces `main` from baseline) | Yes — verified `git diff` empty, `git diff-tree -r` exit 0 |

## Windows Build Status

| Target | Status |
|--------|--------|
| `mddbd.exe` cross-compile (Linux → Windows, `CGO_ENABLED=0`) | Passing |
| `mddb-cli.exe` cross-compile (Linux → Windows, `CGO_ENABLED=0`) | Passing |
| Native `go test ./...` on `windows-latest` | Passing (root `mddb` + all subpackages green) |

## Known Issues

- **SEC-OPEN-1 / SEC-OPEN-2 — CLOSED by patch 0021.** (HTTP `handleRestore` snapshot+rollback added by 0019; `replacefile_windows.go` made atomic and gRPC `Restore` given snapshot+rollback by 0021.) `Server.handleRestore` (0019) takes a safety snapshot and rolls back on failure. Patch 0021 makes `replaceFile` atomic — Go's `os.Rename` on Windows already uses `MoveFileEx(MOVEFILE_REPLACE_EXISTING)`, an atomic in-place replace, so the prior `os.Remove`-then-`os.Rename` crash window is gone — and adds the same snapshot+rollback to gRPC `Restore` on both copy- and reopen-failure. Verified by static inspection of the patched build (base + 0019 + 0020 + 0021).
- The **Vector** feature is now UNBLOCKED (patch 0020 + `MDDB_EMBEDDING_PROVIDER=offline` in the Vector audit job). The full embed→index→search pipeline runs on Windows CI via the deterministic offline provider. All Windows build/runtime/test/CI gaps are covered by patches 0001–0021.

## Blockers

None outstanding. The former **Vector BLOCKED** item is resolved by patch 0020: a deterministic offline embedding provider (`MDDB_EMBEDDING_PROVIDER=offline`) lets the full embed→index→search pipeline run on Windows CI with no network or Ollama server. Vectors are reproducible but semantically neutral; for production semantic quality, configure a real provider (OpenAI/Cohere/Voyage/Ollama) via panel or env. (Running Ollama in CI — option 1 — remains available but was not needed.)

## Next Milestones

1. **Sync with latest upstream MDDB** and re-apply the vendor patch series; resolve any conflicts with minimal new patches.
2. **Upstream the cross-platform correctness fixes** (patches 0005, 0006, 0007, 0008, 0011, 0012, 0013) to reduce long-term divergence.
3. **Expand native Windows CI coverage** if upstream adds new tests or subsystems.
4. **Vector unblock shipped** (patch 0020 + offline provider in the Vector audit job). Optional future: run real-provider semantic-quality tests in a separate CI job.

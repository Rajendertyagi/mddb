# Project Status

Current status of the MDDB Windows port.

**Last updated:** 2026-08-08

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
| Total vendor patches | 19 |
| Patch range | `0001` – `0019` |
| Series complete (reproduces `main` from baseline) | Yes — verified `git diff` empty, `git diff-tree -r` exit 0 |

## Windows Build Status

| Target | Status |
|--------|--------|
| `mddbd.exe` cross-compile (Linux → Windows, `CGO_ENABLED=0`) | Passing |
| `mddb-cli.exe` cross-compile (Linux → Windows, `CGO_ENABLED=0`) | Passing |
| Native `go test ./...` on `windows-latest` | Passing (root `mddb` + all subpackages green) |

## Known Issues

- **SEC-OPEN-1 / SEC-OPEN-2 — CLOSED by patch 0019.** `Server.handleRestore` previously replaced the live DB file in place with no safety snapshot and no rollback on a corrupt/incompatible backup. Patch 0019 adds an atomic pre-restore snapshot, a validated close→copy→open under `Server.withRestoreLock`, and rollback to the snapshot on failure. Both gaps were the same root defect (unconditional overwrite of the live file).
- The **Vector** feature remains BLOCKED (see Blockers). All other Windows build/runtime/test/CI gaps are covered by patches 0001–0019.

## Blockers

**Vector BLOCKED — CI / environment limitation, not a code defect.** `internal/embedding.NewProvider()` supports only `openai` / `ollama` / `voyage` / `cohere` (all require network or a running local Ollama server) and returns `nil` for `none`/empty. CI sets no `MDDB_EMBEDDING_PROVIDER` and starts no Ollama server, so the vector index never initializes and the Vector feature is reported BLOCKED. Unblock options:
1. Run Ollama in CI and set `MDDB_EMBEDDING_PROVIDER=ollama` (heavy; adds a service to the workflow).
2. Add a deterministic offline embedding provider (hash / truncated-hash) as a portable fallback so the Windows port can exercise the vector path without network or Ollama. **Recommended Windows-port fix; pending user decision.**

## Next Milestones

1. **Sync with latest upstream MDDB** and re-apply the vendor patch series; resolve any conflicts with minimal new patches.
2. **Upstream the cross-platform correctness fixes** (patches 0005, 0006, 0007, 0008, 0011, 0012, 0013) to reduce long-term divergence.
3. **Expand native Windows CI coverage** if upstream adds new tests or subsystems.
4. **Decide on Vector unblock** (offline embedding provider vs. CI Ollama) — see Blockers.

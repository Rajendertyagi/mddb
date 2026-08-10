# Project Status

Current status of the MDDB Windows port.

**Last updated:** 2026-08-10

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
| Total vendor patches | 27 |
| Patch range | `0001` – `0027` |
| Series complete (reproduces `main` from baseline) | Yes — verified `git diff` empty, `git diff-tree -r` exit 0 |

## Windows Build Status

| Target | Status |
|--------|--------|
| `mddbd.exe` cross-compile (Linux → Windows, `CGO_ENABLED=0`) | Passing |
| `mddb-cli.exe` cross-compile (Linux → Windows, `CGO_ENABLED=0`) | Passing |
| Native `go test ./...` on `windows-latest` | **Passing** — run `31296463230` (all 15 jobs green). 0021 had reopened the live DB before the backup copy → `TestGRPCRestore_Success` failed with "Access is denied" (run `31294191137`); **fixed by 0022** (live DB stays closed from the initial `Close()` until the backup copy succeeds, then reopens once). `TestGRPCRestore_Success` now PASSES. |

## Final Audit Result (run `31296463230`, commit `b855760`)

| Metric | Score |
|--------|-------|
| Overall Health | **100/100** |
| Windows Readiness | **100/100** |
| Final verdict | **Ready for production** |

- All 15 audit jobs green: `Build Windows Executables`, `Run Unit Tests`, the 13 feature jobs (gRPC API; Ops/Observability/Config; File Upload & Document Processing; Vector & Hybrid Search; Authentication; GraphQL & MCP; Data & Search APIs; Live Functional Tests core; Encryption at rest; Backup & Restore; Replication), `Security & Patch Audit (evidence-graded)`, and `Generate Final Audit Report`.
- Security grade `sec_score = 100` (auth + enc + bak all PASS; `bugs = []`).
- Feature matrix (from report): GraphQL 5/5, MCP 5/5, Vector Search 4/4 — every feature PASS, no UNTESTED/BLOCKED.
- The pre-0022 run `31294191137` (headSha `aa460ef`) remains `failure`; that is the 0021 regression that 0022 closed.

## Known Issues

- **SEC-OPEN-1 / SEC-OPEN-2 — VERIFIED FIXED (live evidence, run `31296463230`).** HTTP `handleRestore` snapshot+rollback added by 0019. `replacefile_windows.go` made atomic by 0021 (Go's `os.Rename` on Windows already uses `MoveFileEx(MOVEFILE_REPLACE_EXISTING)`, an atomic in-place replace — the prior `os.Remove`-then-`os.Rename` crash window is gone). **Attribution nuance:** the gRPC `Restore` path got the snapshot+rollback scaffolding in 0021, but 0021 *also* reopened the live DB between the snapshot and the backup copy — so on Windows `TestGRPCRestore_Success` still failed with "Access is denied". SEC-OPEN-2 for the **gRPC path was actually finalized by patch 0022** (removed the premature reopen; live DB stays closed until the backup copy succeeds, then reopens once — mirroring 0019). So: SEC-OPEN-1 = fixed in 0021 (unchanged since); SEC-OPEN-2 HTTP path = 0019, gRPC path = 0022. Evidence: `TestGRPCRestore_Success` PASSES in run `31296463230`; security audit `bugs = []`, `sec_score = 100` (3 graded features auth/enc/bak = 90 → scaled to 100).
- **gRPC Restore reopen regression (0021) — FIXED & CONFIRMED by patch 0022 (run `31296463230`, all 15 jobs green, earned score 100/100).** 0021 also reopened the live DB (a `bolt.Open` + `g.server.DB` reassignment) *between* taking the snapshot and copying the backup over the live path. On Windows you cannot rename/copy over an open file, so `TestGRPCRestore_Success` failed with `copy backup: rename ... test.db: Access is denied` (CI run `31294191137`, "Run server unit tests" step). Patch 0022 removes that premature reopen; the live DB now stays closed from the initial `Close()` until the backup copy succeeds, then reopens once. The snapshot+rollback safety behavior from 0021 is preserved. `TestGRPCRestore_Success` now PASSES.
- The **Vector** feature is now UNBLOCKED (patch 0020 + `MDDB_EMBEDDING_PROVIDER=offline` in the Vector audit job). The full embed→index→search pipeline runs on Windows CI via the deterministic offline provider. All Windows build/runtime/test/CI gaps are covered by patches 0001–0026.
- **BUG-10 / BUG-11 — BUG-11 FIXED (CI verified); BUG-10 FIXED via 0023 (regression guard) + 0026 (cache-invalidation, CI verified PASS run 31382571122).** BUG-11 (`/v1/events` SSE 500): patch 0024 added `statusRecorder.Flush()` and switched `handleSSE` to `http.NewResponseController` + `supportsFlush`; CI on `93dece8` (0024 alone) **FAILED** `TestSSEHandleThroughNonFlusherWrapper` (still 500) because neither `statusRecorder` nor the regression wrapper `nonFlusherWrap` implemented `Unwrap()`, so `supportsFlush` could not reach the underlying `Flusher`. Patch **0025** adds `Unwrap()` to both, completing the fix. CI run `31378799170` (commit `f61ff2b`, with 0024+0025) confirms `TestSSEHandleThroughNonFlusherWrapper` is **no longer failing → BUG-11 FIXED (CI verified PASS)**. BUG-10 (gRPC `UpdateDocument` content persistence): patch 0023 ships a regression guard `TestGRPCUpdateDocumentPersistsContentMd`. A read-only "not reproducible" assessment was **overturned by CI** — the same run `31378799170` FAILED the guard with `Get returned ContentMd="v1-content", want v2-content`, proving the defect is real. Root cause: `UpdateDocument` (grpc_metadata.go) writes BoltDB but never invalidates the read caches (`g.server.Cache` / `g.server.LockFreeCache`) that `Add` populated, so the cache-first gRPC `Get` (grpc_server.go:244-265) returns stale `v1-content`. REST works because `document_ops.go:348-354` calls `s.Cache.Delete` + `s.LockFreeCache.Delete`. **Patch 0026** adds the identical cache-invalidation to the gRPC `UpdateDocument` path (mirroring `document_ops.go`), closing BUG-10. CI on the 0023+0026 tree RUN green (run 31382571122 Build Windows `ok mddb`; audit run 31382570522 all 15 jobs green) — BUG-10 FIXED (CI verified PASS).
- **BUG-12 — load-coupled RSS growth (P3, stability caveat).** The STEP 21 soak (903 s, ~114.8k ops) passed crash/hang/integrity/error-rate but flagged `LEAK_SUSPECTED` under sustained load (RSS 42 → 153 MB, ~420 MB/h); a controlled idle probe (step21b) showed a **PLATEAU** (RAM stable when idle). Not a crash or data-loss; watch under sustained high-throughput and profile the continuous-write path. See `BUGS.md` / report §9.
- **CI audit-workflow module-cache warning — FIXED by patch 0027 (benign).** `actions/setup-go@v7` in `Mddb-Windows-Audit.yml` lacked `cache-dependency-path`, so its module-cache restore reported `Restore cache failed: Dependencies file is not found … Supported file pattern: go.mod` in the `Feature - gRPC API` and `Live Functional Tests (core)` jobs. The module is at `services/mddbd/go.mod` (with `go.sum` committed, no `vendor/`), so the cache was never restored — slower CI only, no correctness impact. Patch 0027 adds `cache: true` + `cache-dependency-path: services/mddbd/go.sum` to both `setup-go` steps, matching the already-correct steps in the same workflow and `build-windows.yml`. No production behavior change.

## Blockers

None outstanding. The former **Vector BLOCKED** item is resolved by patch 0020: a deterministic offline embedding provider (`MDDB_EMBEDDING_PROVIDER=offline`) lets the full embed→index→search pipeline run on Windows CI with no network or Ollama server. Vectors are reproducible but semantically neutral; for production semantic quality, configure a real provider (OpenAI/Cohere/Voyage/Ollama) via panel or env. (Running Ollama in CI — option 1 — remains available but was not needed.)

## Next Milestones

1. **Sync with latest upstream MDDB** and re-apply the vendor patch series; resolve any conflicts with minimal new patches.
2. **Upstream the cross-platform correctness fixes** (patches 0005, 0006, 0007, 0008, 0011, 0012, 0013, 0026) to reduce long-term divergence.
3. **Expand native Windows CI coverage** if upstream adds new tests or subsystems.
4. **Vector unblock shipped** (patch 0020 + offline provider in the Vector audit job). Optional future: run real-provider semantic-quality tests in a separate CI job.

# Patch Index

Searchable index of the vendor patch series.

**Upstream baseline:** `dbc9def`
**Total patches:** 27
**Last updated:** 2026-08-10

## By Patch Number

| # | Commit | Description | Subsystem | Upstreamable |
|---|--------|-------------|-----------|--------------|
| 0001 | `14c73ec` | Windows builds via cross-compilation | build / CI | No |
| 0002 | `1d96373` | Shim google/renameio for Windows cross-compile | build | No |
| 0003 | `cf24aa3` | Handle return from GetCurrentProcess | platform API | No |
| 0004 | `cdb703c` | Runtime + tests compatibility for Windows | runtime / tests / CI | No |
| 0005 | `1a958ca` | De-flake TestIndexQueue_MultipleJobs | tests | Yes |
| 0006 | `9c80ee4` | De-flake TestIndexQueue_StatsAfterProcessing | tests | Yes |
| 0007 | `685c873` | De-flake TestAuditBatchFlushLarge | tests | Yes |
| 0008 | `076dc2d` | Audit: globally unique BoltDB keys | storage / audit | Yes |
| 0009 | `1dadef4` | gRPC: close+swap+reopen DB during Restore | gRPC / storage | No |
| 0010 | `841cf1b` | gRPC-test: close live s.DB in cleanup | tests | No |
| 0011 | `d978e71` | Replace fixed sleeps with waitFor polling | tests | Yes |
| 0012 | `b6ac55b` | Temporal-test: correct HistogramBucket type | tests | Yes |
| 0013 | `334d7b4` | Indexqueue-test: poll for processed job | tests | Yes |
| 0014 | `7744da5` | Embed mddb-panel web UI into mddbd.exe | packaging / web | No |
| 0015 | `4207661` | Vendor Leaflet markers, optional tiles | panel / web | Yes |
| 0016 | `e55b6c2` | Temporal-test: poll for hot docs | tests | Yes |
| 0017 | `63fbc50` | Audit-test: poll for time-window query | tests | Yes |
| 0018 | `1607fb0` | Indexqueue-test: poll for meta swap | tests | Yes |
| 0019 | `deea49b` | HTTP: atomic snapshot+rollback for Restore (SEC-OPEN-1/SEC-OPEN-2) | storage / security / HTTP | No |
| 0020 | `0ca19ea` | Embedding: deterministic offline provider (unblocks Vector on Windows CI) | embedding / vector / CI | No |
| 0021 | `1e66ea1` | Security: atomic Windows replace + gRPC restore rollback (SEC-OPEN-1/SEC-OPEN-2) | security / windows / restore | No |
| 0022 | `c8460a7` | gRPC Restore: keep live DB closed until backup copy (0021 regression fix) | gRPC / storage / windows | No |
| 0023 | `93dece8` | gRPC UpdateDocument: persist ContentMd (BUG-10 regression guard) | gRPC / tests | Yes |
| 0024 | `93dece8` | SSE /v1/events: make statusRecorder flushable + http.NewResponseController (BUG-11 FIX, part 1) | http / metrics / sse | Yes |
| 0025 | verified | SSE /v1/events: expose Unwrap() on statusRecorder + test wrapper (BUG-11 FIX, part 2) | http / metrics / sse | Yes |
| 0026 | verified | gRPC UpdateDocument: invalidate read caches Cache + LockFreeCache (BUG-10 FIX) | gRPC / cache | Yes |
| 0027 | `37e6734` | Audit workflow: setup-go cache-dependency-path (CI cache fix) | CI / workflow | No |

## By Subsystem

### Build / CI
- 0001 `14c73ec` — Windows builds via cross-compilation
- 0002 `1d96373` — Shim google/renameio for Windows cross-compile
- 0004 `cdb703c` — Runtime + tests compatibility (test-side fixes; CI job is in the committed workflow)
- 0027 `37e6734` — Audit workflow: setup-go cache-dependency-path (CI cache fix)

### Platform API
- 0003 `cf24aa3` — Handle return from GetCurrentProcess
- 0004 `cdb703c` — replaceFile helper, UDS skip, temp paths

### Storage / Audit
- 0008 `076dc2d` — Audit: globally unique BoltDB keys
- 0009 `1dadef4` — gRPC: close+swap+reopen DB during Restore
- 0019 `deea49b` — HTTP: atomic snapshot+rollback for Restore (SEC-OPEN-1/SEC-OPEN-2)
- 0020 `0ca19ea` — Embedding: deterministic offline provider (unblocks Vector on Windows CI)
- 0021 `1e66ea1` — Security: atomic Windows replace + gRPC restore rollback (SEC-OPEN-1/SEC-OPEN-2)
- 0022 `c8460a7` — gRPC Restore: keep live DB closed until backup copy (fixes 0021 regression)

### gRPC
- 0009 `1dadef4` — gRPC: close+swap+reopen DB during Restore
- 0010 `841cf1b` — gRPC-test: close live s.DB in cleanup
- 0022 `c8460a7` — gRPC Restore: keep live DB closed until backup copy (0021 regression fix)
- 0023 `93dece8` — gRPC UpdateDocument content-persistence regression test
- 0026 `verified` — gRPC UpdateDocument: invalidate read caches (BUG-10 FIX, CI PASS run 31382571122)

### Tests
- 0004 `cdb703c` — Runtime + tests compatibility (test-side fixes)
- 0005 `1a958ca` — De-flake TestIndexQueue_MultipleJobs
- 0006 `9c80ee4` — De-flake TestIndexQueue_StatsAfterProcessing
- 0007 `685c873` — De-flake TestAuditBatchFlushLarge
- 0010 `841cf1b` — gRPC-test: close live s.DB in cleanup
- 0011 `d978e71` — Replace fixed sleeps with waitFor polling
- 0012 `b6ac55b` — Temporal-test: correct HistogramBucket type
- 0013 `334d7b4` — Indexqueue-test: poll for processed job
- 0016 `e55b6c2` — Temporal-test: poll for hot docs instead of fixed sleep
- 0017 `63fbc50` — Audit-test: poll for time-window query instead of fixed flush wait
- 0018 `1607fb0` — Indexqueue-test: poll for meta swap instead of fixed sleep
- 0023 `93dece8` — gRPC-test: UpdateDocument persist ContentMd regression guard

### Packaging / Web
- 0014 `7744da5` — Embed mddb-panel web UI into mddbd.exe
- 0015 `4207661` — Vendor Leaflet markers locally, optional tiles

### HTTP / Metrics / SSE
- 0024 `93dece8` — SSE /v1/events: statusRecorder.Flush() + http.NewResponseController (BUG-11 fix, part 1)
- 0025 `verified` — SSE /v1/events: Unwrap() on statusRecorder + test wrapper (BUG-11 fix, part 2, CI PASS run 31378799170)

## By Type

### Windows-only (not upstreamable)
- 0001, 0002, 0003, 0004, 0009, 0010, 0014, 0019, 0020, 0021, 0022, 0027

### Cross-platform correctness (upstreamable)
- 0005, 0006, 0007, 0008, 0011, 0012, 0013, 0015, 0016, 0017, 0018, 0023, 0024, 0025, 0026

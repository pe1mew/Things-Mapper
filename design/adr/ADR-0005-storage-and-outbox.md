# ADR-0005 — Storage: SQLDelight with a transactional outbox

Status: **Accepted** · Date: 2026-09-11

## Context

Every uplink must be stored before it is drawn or uploaded, uploads must
survive restarts and connectivity loss, and the same storage code must run
on Android, JVM (Linux/Pi) and later iOS. The original used Room (Android
only) and uploaded synchronously without a queue, losing packets on failure
(EVAL-21, EVAL-23).

## Decision drivers

- Multiplatform SQLite access with typed queries and verified migrations.
- Atomicity between "measurement stored" and "upload scheduled".
- Streaming reads for exports of 100 k rows; modest memory.
- No proprietary or discontinued components.

## Options

| Option | Assessment |
|---|---|
| Room | Android only (a KMP variant exists but is younger and Android-centric); would tie the core to AndroidX. |
| **SQLDelight 2.x** | Common Kotlin, generated typed queries from `.sq` files, drivers for Android, JVM (sqlite-jdbc incl. aarch64) and native; schema verification and migration tests built in; Apache-2.0. |
| Realm Kotlin | Multiplatform, but the product line was deprecated in 2024. |
| Files (JSON Lines / CSV) as primary store | Simple, but no transactions, no indexed queries for sessions/statistics, fragile under crash. |
| Upload scheduling with WorkManager | Android only; the same problem must be solved on JVM anyway. |
| In-memory upload queue | Loses items on process death — exactly the defect being fixed. |

## Decision

Use **SQLDelight** as the single local database on all platforms with the
schema of TDS §5.1. Apply the **transactional outbox pattern**: the pipeline
writes measurement, receptions, gateway upserts and the outbox rows for the
eligible upload targets in one transaction; per-target workers consume the
outbox with retry/backoff and mark results. Delivery is at-least-once;
ttnmapper.org tolerates the rare duplicate that a crash between send and
mark can cause.

## Consequences

- One writer (the pipeline actor) per session; WAL mode; batched writes.
- Migrations are numbered and tested against archived databases of every
  release (TC-ST-02).
- Upload state is derivable and visible per session (REQ-F-051, F-082).
- Export reads stream through SQLDelight's cursor-based queries.
- Secrets are **not** in this database (secure store, TDS §5.4).

## Links

[03 FDS FDS-6, FDS-10, §5.3](../03-fds.md), [05 TDS §5](../05-tds.md).

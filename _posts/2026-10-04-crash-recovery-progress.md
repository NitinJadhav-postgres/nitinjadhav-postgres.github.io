---
author: Nitin Jadhav
title: How Far Has PostgreSQL Crash Recovery Progressed?
date: 2026-10-04
description: Learn how to monitor PostgreSQL crash-recovery progress using startup logs and WAL replay positions.
section: "blog"
pin: true
math: false
mermaid: false
---

A PostgreSQL server restarts after an unexpected shutdown. Before it can accept connections, the log reports:

```text
LOG:  database system was not properly shut down; automatic recovery in progress
LOG:  redo starts at 1/1000028
```

Crash recovery is underway—but how far has it progressed?

Crash recovery replays PostgreSQL's write-ahead log (WAL). PostgreSQL identifies each byte position in WAL with a log sequence number (LSN).

By comparing the reported LSNs over time, we can investigate four practical questions:

- **Movement:** Is WAL replay advancing?
- **Volume:** How much WAL has been replayed?
- **Health:** Is recovery slow or stalled?
- **Completion:** Can we calculate a completion percentage or estimate when the database will become available?

During a long-running startup, PostgreSQL can periodically report the current WAL replay position. This lets us measure recovery movement and replay throughput.

This is the first in a series exploring how to measure PostgreSQL recovery progress using information available today. The series covers crash recovery, archive recovery—including point-in-time recovery—and standby recovery using WAL available in `pg_wal`, restored from a WAL archive, or received through streaming replication. A later post will examine what can and cannot be estimated from these signals.

## What happens during crash recovery?

PostgreSQL uses WAL to ensure that data files can be restored to a consistent state after an unexpected shutdown.

A [checkpoint guarantees that changes before a certain point in WAL have been written to the data files](https://www.postgresql.org/docs/current/wal-configuration.html). During crash recovery, PostgreSQL reads the latest checkpoint information and begins redo from the checkpoint's redo location. It then scans forward through WAL and reapplies changes that might not yet be present in the data files.

A simplified view is:

```text
checkpoint redo LSN                      end of valid WAL
        |                                       |
        v                                       v
        +---------- WAL records to replay ------+
                         ^
                         |
                  current replay LSN
```

*Crash recovery starts at the checkpoint's redo LSN and scans forward toward the end of valid WAL.*

To calculate a recovery percentage, we would need three positions:

```text
recovery start
current replay position
recovery end
```

The theoretical calculation is:

```text
progress =
    (current_lsn - start_lsn)
    / (end_lsn - start_lsn)
```

PostgreSQL exposes the starting and current positions reasonably well. The difficult part is knowing the final valid end of WAL before recovery reaches it.

## PostgreSQL startup-progress messages

PostgreSQL 15 introduced progress messages for slow startup operations. The interval is controlled by [`log_startup_progress_interval`](https://www.postgresql.org/docs/current/runtime-config-logging.html#GUC-LOG-STARTUP-PROGRESS-INTERVAL).

The default value is 10 seconds. A value of zero disables the messages. The setting applies separately to each startup operation and can be configured in `postgresql.conf` or on the server command line.

For a test environment, we can use a shorter interval:

```conf
log_startup_progress_interval = '5s'
```

After an unclean shutdown, a long crash recovery can produce messages similar to:

```text
LOG:  database system was not properly shut down; automatic recovery in progress
LOG:  redo starts at 1/1000028
LOG:  redo in progress, elapsed time: 5.00 s, current LSN: 1/130A71B0
LOG:  redo in progress, elapsed time: 10.00 s, current LSN: 1/1705CE40
LOG:  redo in progress, elapsed time: 15.00 s, current LSN: 1/1C91B728
LOG:  redo done at 1/208A3958 system usage: ...
LOG:  database system is ready to accept connections
```

These messages provide:

- **Starting position:** The redo starting LSN.
- **Current position:** The current replay LSN.
- **Elapsed time:** How long redo has been running.
- **Final position:** The replay position after redo completes.

This is enough to determine whether WAL replay is advancing and how many WAL bytes have been processed.

## Calculating how much WAL has been replayed

An LSN is a byte position in PostgreSQL's WAL stream. You can [compare LSN values to calculate the volume of WAL between them](https://www.postgresql.org/docs/current/wal-internals.html), which makes them useful for measuring recovery progress.

If recovery starts at:

```text
1/1000028
```

and the latest progress message reports:

```text
1/1C91B728
```

then:

```text
WAL replayed = current LSN - redo start LSN
```

After the server becomes available, you can perform this calculation with PostgreSQL's `pg_lsn` type:

```sql
SELECT
    '1/1C91B728'::pg_lsn -
    '1/1000028'::pg_lsn AS bytes_replayed;
```

During crash recovery, however, the server is not yet accepting ordinary SQL connections. An external monitoring tool must parse the log and perform the calculation itself.

A small Python function can convert an LSN to an integer byte position:

```python
def lsn_to_bytes(lsn: str) -> int:
    high, low = lsn.split("/")
    return (int(high, 16) << 32) + int(low, 16)


start_lsn = lsn_to_bytes("1/1000028")
current_lsn = lsn_to_bytes("1/1C91B728")

bytes_replayed = current_lsn - start_lsn

print(f"Replayed: {bytes_replayed / 1024 / 1024:.2f} MiB")
```

This measures distance through WAL. It does not measure the number of WAL records processed, database pages changed, or amount of actual work performed.

## Calculating replay throughput

Successive progress messages can also be used to estimate the recent WAL replay rate.

Suppose the log reports:

```text
elapsed time:  60.00 s, current LSN: 1/3C000000
elapsed time: 120.00 s, current LSN: 1/8C000000
```

The replay rate over that interval is:

```text
replay rate =
    (second_lsn - first_lsn)
    / (second_time - first_time)
```

For example:

```python
first_lsn = lsn_to_bytes("1/3C000000")
second_lsn = lsn_to_bytes("1/8C000000")

elapsed_seconds = 120 - 60

rate = (second_lsn - first_lsn) / elapsed_seconds

print(f"Replay rate: {rate / 1024 / 1024:.2f} MiB/s")
```

It is usually better to measure the rate between recent samples instead of dividing all replayed WAL by the total elapsed time. Recovery throughput can change as the workload, cache state, and I/O behavior change.

A monitoring tool could report:

```text
Crash recovery
  Redo start:          1/1000028
  Current replay LSN:  1/8C000000
  Elapsed time:        120 seconds
  WAL replayed:        2.17 GiB
  Recent replay rate:  21.33 MiB/s
```

This is useful operational information even without a completion percentage or ETA.

## Why this is not a recovery percentage

To calculate a percentage, we also need to know the end LSN:

```text
percentage =
    replayed bytes
    / total bytes requiring replay
```

During crash recovery, PostgreSQL scans WAL forward until it reaches the end of valid WAL. That position is not directly reported at the beginning of recovery.

You might be tempted to use the highest WAL segment filename in `pg_wal` as the endpoint. Unfortunately, that is not reliable.

PostgreSQL can recycle old WAL files by renaming them as future WAL segments. Therefore, the presence and size of a segment file do not prove that the complete segment contains valid WAL requiring replay. The last active segment can also be only partially filled.

For example:

```text
pg_wal/
    000000010000000100000020
    000000010000000100000021
    000000010000000100000022
    000000010000000100000023
```

The existence of segment `23` does not necessarily mean that recovery must replay through the end of segment `23`.

At best, WAL-file inspection might provide an upper bound. Treating that upper bound as the exact endpoint could produce a misleading percentage.

## What can `pg_controldata` tell us?

[`pg_controldata` displays cluster-wide control information](https://www.postgresql.org/docs/current/app-pgcontroldata.html), including checkpoint and WAL information:

```bash
pg_controldata "$PGDATA"
```

Relevant fields can include:

```text
Database cluster state:               in production
Latest checkpoint location:           1/1000098
Latest checkpoint's REDO location:    1/1000028
Latest checkpoint's TimeLineID:       1
```

The checkpoint's redo location helps explain where recovery will begin.

However, `pg_controldata` gives us only the recovery starting point. It does not reveal the final valid WAL position, so it cannot tell us how much work remains.

## Is recovery progressing or stuck?

If you see the current LSN change between progress messages, you know that WAL replay is advancing:

```text
1/130A71B0
1/1705CE40
1/1C91B728
```

If the current LSN does not change for several intervals, further investigation is needed. However, an unchanged LSN does not immediately prove that recovery is stuck.

Recovery might be:

- processing an expensive WAL record;
- waiting for data-page I/O;
- waiting for WAL I/O;
- synchronizing files;
- resetting unlogged relations;
- performing end-of-recovery work;
- slowed by storage latency;
- competing with other activity on the host.

It is therefore useful to correlate PostgreSQL logs with operating-system signals such as:

- CPU utilization;
- disk throughput;
- disk latency;
- I/O queue depth;
- process state;
- filesystem or storage errors.

A practical interpretation might be:

| Observation | Possible interpretation |
|---|---|
| LSN advancing and rate stable | Recovery is actively progressing |
| LSN advancing but rate falling | Recovery is progressing but slowing |
| LSN unchanged and storage busy | Recovery might be waiting for data or WAL I/O |
| LSN unchanged and CPU busy | Recovery might be processing expensive redo work |
| LSN unchanged with no resource activity | Investigate waits, errors, or a possible stall |
| Redo complete but server unavailable | PostgreSQL might be performing another startup phase |

The last case is important:

> Redo completion is not necessarily the same as database availability.

After WAL replay finishes, PostgreSQL can still have startup work to complete before logging:

```text
LOG:  database system is ready to accept connections
```

A recovery monitor should therefore distinguish between:

```text
WAL redo completed
database became available
```

## Conclusion

PostgreSQL's startup-progress messages provide enough information to monitor crash recovery meaningfully. By collecting the redo start LSN, current replay LSN, and elapsed time, we can determine:

- how much WAL has been replayed;
- whether recovery is advancing;
- the recent WAL replay rate;
- when redo has completed; and
- when the database becomes available.

What we generally cannot calculate is an exact completion percentage. PostgreSQL discovers the end of valid WAL while processing it, and the WAL files present in `pg_wal` do not necessarily identify the exact recovery endpoint.

This distinction is important:

> We can measure how far crash recovery has moved without necessarily knowing how much work remains.

In the next post, I will examine archive recovery, including point-in-time recovery. We will compare cases where the endpoint is a known LSN with cases where PostgreSQL must discover the endpoint during WAL replay.

---
author: Nitin Jadhav
title: Report progress of startup operations that take a long time
date: 2021-10-25
commiturl: "https://github.com/postgres/postgres/commit/9ce346eabf350a130bba46be3f8c50ba28506969"
section: "authored"
pin: true
math: true
mermaid: true
---

Users sometimes get concerned whe they start the server and it
emits a few messages and then doesn't emit any more messages for
a long time. Generally, what's happening is either that the
system is taking a long time to apply WAL, or it's taking a
long time to reset unlogged relations, or it's taking a long
time to fsync the data directory, but it's not easy to tell
which is the case.

To fix that, add a new 'log_startup_progress_interval' setting,
by default 10s. When an operation that is known to be potentially
long-running takes more than this amount of time, we'll log a
status update each time this interval elapses.

To avoid undesirable log chatter, don't log anything about WAL
replay when in standby mode.

Nitin Jadhav and Robert Haas, reviewed by Amul Sul, Bharath
Rupireddy, Justin Pryzby, Michael Paquier, and Álvaro Herrera.

Discussion: https://postgr.es/m/CA+TgmoaHQrgDFOBwgY16XCoMtXxsrVGFB2jNCvb7-ubuEe1MGg@mail.gmail.com
Discussion: https://postgr.es/m/CAMm1aWaHF7VE69572_OLQ+MgpT5RUiUDgF1x5RrtkJBLdpRj3Q@mail.gmail.com
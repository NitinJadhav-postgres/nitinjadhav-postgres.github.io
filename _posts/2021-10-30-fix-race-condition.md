---
author: Nitin Jadhav
title: Fix race condition in startup progress reporting
date: 2021-10-30
commiturl: "https://github.com/postgres/postgres/commit/5ccceb2946d4104804f8dca67515b602f5e78cdd"
section: "reviewed"
pin: true
math: true
mermaid: true
---

Commit 9ce346e added startup
progress reporting, but begin_startup_progress_phase has a race
condition: the timeout for the previous phase might fire just
before we reschedule the interrupt for the next phase.

To avoid the race, disable the timeout, clear the flag, and then
re-enable the timeout.

Patch by me, reviewed by Nitin Jadhav.

Discussion: https://postgr.es/m/CA+TgmoYq38i6iAzfRLVxA6Cm+wMCf4WM8wC3o_a+X_JvWC8bJg@mail.gmail.com
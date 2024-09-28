---
author: Nitin Jadhav
title: Extend check_GUC_init() with checks on flag combinations when loading GUCs
date: 2023-02-06
commiturl: "https://github.com/postgres/postgres/commit/009f8d17146da72478fcb8f544b793c443fa254c"
section: "authored"
pin: true
math: true
mermaid: true
---

This extends the work begun by a73952b, with the addition of a GUC check
for flag combinations in check_GUC_init(), making sure that anything
defined with GUC_NO_SHOW_ALL also includes GUC_NOT_IN_SAMPLE, as first
step.  There has never been any GUCs of this kind in the core code, and
this combination makes little sense as a parameter marked as not fit for
SHOW ALL should not be hidden in postgresql.conf.sample.

Note that GUCs marked with GUC_NO_SHOW_ALL are not listed under
pg_settings or SHOW ALL (still they can be queried individually), making
them unfit for checks via SQL queries in the regression tests that do a
full scan of the parameters available.  The SQL tests are still a bit
incorrect about that, and will be cleaned up in a separate commit.  We
have also discussed the possibility to extend the SQL functions for GUCs
so as they could show more information about parameters defined with
GUC_NO_SHOW_ALL, though it has been concluded that this is not worth the
extra complication in the long run, an enforced policy at initialization
time being enough to do the same job.

Per discussion with Nitin Jadhav and Tom Lane.

Discussion: https://postgr.es/m/CAMm1aWaYe0muu3ABo7iSAgK+OWDS9yNe8GGRYnCyeEpScYKa+g@mail.gmail.com
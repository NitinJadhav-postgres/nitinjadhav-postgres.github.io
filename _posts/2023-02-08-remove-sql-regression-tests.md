---
author: Nitin Jadhav
title: Remove SQL regression tests for GUCs related to NO_SHOW_ALL
date: 2023-02-08
commiturl: "https://github.com/postgres/postgres/commit/b7e84c65d5b2d694a669ae1db8ab1d6c51ef8596/"
section: "authored"
pin: true
math: true
mermaid: true
---

No GUCs that use NO_SHOW_ALL are reported in pg_show_all_settings(),
hence trying to check combinations of flags related to it is pointless.

These queries have been introduced by d10e41d, so backpatch down to 15
to keep all the branches consistent.  Equivalent checks based on
NO_SHOW_ALL could be added in check_GUC_init() when a GUC is initially
loaded, but this can be done only on HEAD.

Author: Nitin Jadhav
Discussion: https://postgr.es/m/CAMm1aWaYe0muu3ABo7iSAgK+OWDS9yNe8GGRYnCyeEpScYKa+g@mail.gmail.com
Backpatch-through: 15
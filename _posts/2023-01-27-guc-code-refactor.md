---
author: Nitin Jadhav
title: Minor GUC code refactoring
date: 2023-01-27
commiturl: "https://github.com/postgres/postgres/commit/e4e89eb5bbfdae30349b38344e9c604411174f6b"
section: "authored"
pin: true
math: true
mermaid: true
---

Split out "ConfigOptionIsVisible" to perform the privilege
check for GUC_SUPERUSER_ONLY GUCs (which these days can also
be read by pg_read_all_settings role members), and move the
should-we-show-it checks from GetConfigOptionValues to its
sole caller.

This commit also removes get_explain_guc_options's check of
GUC_NO_SHOW_ALL, which seems to have got cargo-culted in there.
While there's no obvious use-case for marking a GUC both
GUC_EXPLAIN and GUC_NO_SHOW_ALL, if it were set up that way
one would expect EXPLAIN to show it --- if that's not what
you want, then don't set GUC_EXPLAIN.

In passing, simplify the loop logic in show_all_settings.

Nitin Jadhav, Bharath Rupireddy, Tom Lane

Discussion: https://postgr.es/m/CAMm1aWYgfekpRK-Jz5=pM_bV+Om=ktGq1vxTZ_dr1Z6MV-qokA@mail.gmail.com
---
author: Nitin Jadhav
title: Make use of in-core query id added by commit 5fd9dfa
date: 2021-04-07
commiturl: "https://github.com/postgres/postgres/commit/4f0b0966c866ae9f0e15d7cc73ccf7ce4e1af84b"
section: "reviewed"
pin: true
math: true
mermaid: true
---

Use the in-core query id computation for pg_stat_activity,
log_line_prefix, and EXPLAIN VERBOSE.

Similar to other fields in pg_stat_activity, only the queryid from the
top level statements are exposed, and if the backends status isn't
active then the queryid from the last executed statements is displayed.

Add a %Q placeholder to include the queryid in log_line_prefix, which
will also only expose top level statements.

For EXPLAIN VERBOSE, if a query identifier has been computed, either by
enabling compute_query_id or using a third-party module, display it.

Bump catalog version.

Discussion: https://postgr.es/m/20210407125726.tkvjdbw76hxnpwfi@nol

Author: Julien Rouhaud

Reviewed-by: Alvaro Herrera, Nitin Jadhav, Zhihong Yu
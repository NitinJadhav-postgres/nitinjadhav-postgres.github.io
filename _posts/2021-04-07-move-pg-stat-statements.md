---
author: Nitin Jadhav
title: Move pg_stat_statements query jumbling to core
date: 2021-04-07
commiturl: "https://github.com/postgres/postgres/commit/5fd9dfa5f50e4906c35133a414ebec5b6d518493"
section: "reviewed"
pin: true
math: true
mermaid: true
---

Add compute_query_id GUC to control whether a query identifier should be
computed by the core (off by default).  It's thefore now possible to
disable core queryid computation and use pg_stat_statements with a
different algorithm to compute the query identifier by using a
third-party module.

To ensure that a single source of query identifier can be used and is
well defined, modules that calculate a query identifier should throw an
error if compute_query_id specified to compute a query id and if a query
idenfitier was already calculated.

Discussion: https://postgr.es/m/20210407125726.tkvjdbw76hxnpwfi@nol

Author: Julien Rouhaud

Reviewed-by: Alvaro Herrera, Nitin Jadhav, Zhihong Yu
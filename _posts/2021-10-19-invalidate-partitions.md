---
author: Nitin Jadhav
title: Invalidate partitions of table being attached/detached
date: 2021-10-19
commiturl: "https://github.com/postgres/postgres/commit/d6f1e16c8fe27100e371a15aeeb498faa680ceed"
section: "reviewed"
pin: true
math: true
mermaid: true
---

Failing to do that, any direct inserts/updates of those partitions
would fail to enforce the correct constraint, that is, one that
considers the new partition constraint of their parent table.

Backpatch to 10.

Reported by: Hou Zhijie <houzj.fnst@fujitsu.com>
Author: Amit Langote <amitlangote09@gmail.com>
Author: Álvaro Herrera <alvherre@alvh.no-ip.org>
Reviewed-by: Nitin Jadhav <nitinjadhavpostgres@gmail.com>
Reviewed-by: Pavel Borisov <pashkin.elfe@gmail.com>

Discussion: https://postgr.es/m/OS3PR01MB5718DA1C4609A25186D1FBF194089%40OS3PR01MB5718.jpnprd01.prod.outlook.com
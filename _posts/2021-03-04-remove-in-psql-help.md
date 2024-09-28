---
author: Nitin Jadhav
title: Remove redundant getenv() for PGUSER, in psql help
date: 2021-03-04
commiturl: "https://github.com/postgres/postgres/commit/4a4241e15b246663fc44b5e5355bc6d19c6d2aa6"
section: "reviewed"
pin: true
math: true
mermaid: true
---

Previously psql obtained the value of PGUSER twice to display
a default user in its help message.

Author: Kota Miyake
Reviewed-by: Nitin Jadhav, Fujii Masao
Discussion: https://postgr.es/m/2a3c612babdd6ed63a9d877bb575d793@oss.nttdata.com
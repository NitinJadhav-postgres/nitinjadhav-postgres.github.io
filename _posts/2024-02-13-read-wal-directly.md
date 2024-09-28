---
author: Nitin Jadhav
title: Read WAL directly from WAL buffers
date: 2024-02-13
commiturl: "https://github.com/postgres/postgres/commit/91f2cae7a4e664e9c0472b364c7db29d755ab151"
section: "reviewed"
pin: true
math: true
mermaid: true
---

If available, read directly from WAL buffers, avoiding the need to go
through the filesystem. Only for physical replication for now, but can
be expanded to other callers.

In preparation for replicating unflushed WAL data.

Author: Bharath Rupireddy
Discussion: https://postgr.es/m/CALj2ACXKKK%3DwbiG5_t6dGao5GoecMwRkhr7GjVBM_jg54%2BNa%3DQ%40mail.gmail.com
Reviewed-by: Andres Freund, Alvaro Herrera, Nathan Bossart, Dilip Kumar, Nitin Jadhav, Melih Mutlu, Kyotaro Horiguchi
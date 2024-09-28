---
author: Nitin Jadhav
title: Use native path separators to pg_ctl in initdb
date: 2021-03-03
commiturl: "https://github.com/postgres/postgres/commit/75dbfe4ca70d5b7a83f61b0a66a0a14ad0b739ed"
section: "authored"
pin: true
math: true
mermaid: true
---

On Windows, CMD.EXE allegedly does not run a command that uses forward slashes,
so let's convert the path to use backslashes instead.

Backpatch to 10.

Author: Nitin Jadhav <nitinjadhavpostgres@gmail.com>
Reviewed-by: Juan José Santamaría Flecha <juanjo.santamaria@gmail.com>
Discussion: https://postgr.es/m/CAMm1aWaNDuaPYFYMAqDeJrZmPtNvLcJRS++CcZWY8LT6KcoBZw@mail.gmail.com
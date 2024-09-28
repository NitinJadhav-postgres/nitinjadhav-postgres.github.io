---
author: Nitin Jadhav
title: Add pg_buffercache_evict() function for testing
date: 2024-04-07
commiturl: "https://github.com/postgres/postgres/commit/13453eedd3f692f8dcf8e334396eee84f00fdde2"
section: "reviewed"
pin: true
math: true
mermaid: true
---

When testing buffer pool logic, it is useful to be able to evict
arbitrary blocks.  This function can be used in SQL queries over the
pg_buffercache view to set up a wide range of buffer pool states.  Of
course, buffer mappings might change concurrently so you might evict a
block other than the one you had in mind, and another session might
bring it back in at any time.  That's OK for the intended purpose of
setting up developer testing scenarios, and more complicated interlocking
schemes to give stronger guararantees about that would likely be less
flexible for actual testing work anyway.  Superuser-only.

Author: Palak Chaturvedi <chaturvedipalak1911@gmail.com>
Author: Thomas Munro <thomas.munro@gmail.com> (docs, small tweaks)
Reviewed-by: Nitin Jadhav <nitinjadhavpostgres@gmail.com>
Reviewed-by: Andres Freund <andres@anarazel.de>
Reviewed-by: Cary Huang <cary.huang@highgo.ca>
Reviewed-by: Cédric Villemain <cedric.villemain+pgsql@abcsql.com>
Reviewed-by: Jim Nasby <jim.nasby@gmail.com>
Reviewed-by: Maxim Orlov <orlovmg@gmail.com>
Reviewed-by: Thomas Munro <thomas.munro@gmail.com>
Reviewed-by: Melanie Plageman <melanieplageman@gmail.com>
Discussion: https://postgr.es/m/CALfch19pW48ZwWzUoRSpsaV9hqt0UPyaBPC4bOZ4W+c7FF566A@mail.gmail.com
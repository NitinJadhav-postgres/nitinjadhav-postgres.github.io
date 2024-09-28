---
author: Nitin Jadhav
title: Reduce the number of pallocs when building partition bounds
date: 2021-07-06
commiturl: "https://github.com/postgres/postgres/commit/53d86957e980efa06f15232b8cff430d4cc6dd64"
section: "authored"
pin: true
math: true
mermaid: true
---

In each of the create_*_bound() functions for LIST, RANGE and HASH
partitioning, there were a large number of palloc calls which could be
reduced down to a much smaller number.

In each of these functions, an array was built so that we could qsort it
before making the PartitionBoundInfo. For LIST and HASH partitioning, an
array of pointers was allocated then each element was allocated within
that array.  Since the number of items of each dimension is known
beforehand, we can just allocate a single chunk of memory for this.

Similarly, with all partition strategies, we're able to reduce the number
of allocations to build the ->datums field.  This is an array of Datum
pointers, but there's no need for the Datums that each element points to
to be singly allocated.  One big chunk will do.  For RANGE partitioning,
the PartitionBoundInfo->kind field can get the same treatment.

We can apply the same optimizations to partition_bounds_copy().  Doing
this might have a small effect on cache performance when searching for the
correct partition during partition pruning or DML on a partitioned table.
However, that's likely to be small and this is mostly about reducing
palloc overhead.

Author: Nitin Jadhav, Justin Pryzby, David Rowley
Reviewed-by: Justin Pryzby, Zhihong Yu
Discussion: https://postgr.es/m/flat/CAMm1aWYFTqEio3bURzZh47jveiHRwgQTiSDvBORczNEz2duZ1Q@mail.gmail.com
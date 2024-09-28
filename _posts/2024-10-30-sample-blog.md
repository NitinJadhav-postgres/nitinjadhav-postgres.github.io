---
author: Nitin Jadhav
title: A deep dive into the file format and internals of write-ahead logging of PostgreSQL
date: 2024-08-30
slidesurl: "https://www.pgday.org.il/#program"
scheduleurl: "https://www.pgday.org.il/#program"
videourl: "https://www.pgday.org.il/#program"
section: "blog"
pin: true
math: true
mermaid: true
---

Write-Ahead Logging (WAL) is a standard method for ensuring data integrity. A detailed description can be found in most (if not all) books about transaction processing. Briefly, WAL's central concept is that changes to data files (where tables and indexes reside) must be written only after those changes have been logged, that is, after WAL records describing the changes have been flushed to permanent storage. If we follow this procedure, we do not need to flush data pages to disk on every transaction commit, because we know that in the event of a crash we will be able to recover the database using the log: any changes that have not been applied to the data pages can be redone from the WAL records. (This is roll-forward recovery, also known as REDO.)

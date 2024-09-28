---
author: Nitin Jadhav
title: Support "of", "tzh" and "tzm" format codes
date: 2022-03-15
commiturl: "https://github.com/postgres/postgres/commit/9dde82899cdf48bd7b2f3d83e4f724ac9ae02c79"
section: "authored"
pin: true
math: true
mermaid: true
---

The upper case versions "OF", "TZH", and "TZM" are already supported,
and all other format codes that are supported in upper case are also
supported in lower case, so we should support these as well for
consistency.

Nitin Jadhav, with a tiny cosmetic change by me. Reviewed by Suraj
Kharage and David Zhang.

Discussion: http://postgr.es/m/CAMm1aWZ-oZyKd75+8D=VJ0sAoSwtdXWLP-MAWD4D8R1Dgandzw@mail.gmail.com
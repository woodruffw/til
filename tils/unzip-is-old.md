---
title: "'unzip' is old"
date: 2026-09-20
tags: [oss, security, shell]
---

Like a lot of people I've invoked the [`unzip`](https://linux.die.net/man/1/unzip)
tool thousands of times (enough that I have muscle memory for its options/flags).

But I've never really thought about where it comes from, or who maintains it.

The answers end up being interesting:

- `unzip` is Info-ZIP's UnZip 6.0, as originally open sourced in 2009. The [unzip package] in
  Debian, Ubuntu and similar is essentially just that original source dump from 2009,
  with various patches carried forward over time.
- As best I can tell, the "living" copy of UnZip is [madler/unzip] on GitHub, which also hasn't
  received any patches in several years (not that it necessarily needs to). Debian and others
  don't link to this repository; they link to the original [SourceForge website], which hasn't
  been updated since 2009.

Software can be and often is "done," so age is not in and of itself a concern. However, 17 years
is a _long_ time, and our understanding of security with respect to C codebases (and particularly
differential-prone formats like ZIP) has shifted significantly in that period.

[unzip package]: https://packages.debian.org/search?keywords=unzip

[madler/unzip]: https://github.com/madler/unzip

[SourceForge website]: https://infozip.sourceforge.net/UnZip.html

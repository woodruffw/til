---
title: "POSIX getdate(3) is odd"
date: 2026-09-21
tags: [posix, c]
---

I've used [`strptime(3)`](https://pubs.opengroup.org/onlinepubs/9799919799/functions/strptime.html)
many times, but POSIX also standardizes [`getdate(3)`](https://pubs.opengroup.org/onlinepubs/9799919799/functions/getdate.html).

The difference between these can be seen in their signatures:

```c
char *strptime(const char *restrict buf, const char *restrict format, struct tm *restrict tm);
struct tm *getdate(const char *string);
```

In English: both parse an input into a `struct tm`, but that's where the similiarities end:

- `strptime(3)` accepts a `struct tm*` to write into; `getdate(3)` is defined to return
  a `struct tm*` (which implies non-reentrancy, and POSIX.1-2024 formalizes this by saying
  it "need not be thread-safe").
- `strptime(3)` returns `NULL` on error, and (as of POSIX.1-2024) otherwise defines no
  error states; `getdate(3)` has its own special `getdate_err` error variable or macro
  that reflects the error state. POSIX.1-2024 does not define whether `errno` is affected
  by `getdate(3)`.
- Finally, `strptime(3)` takes a format string, which is used to parse the input. But
  `getdate(3)` only takes an input.

That last point is what originally made me curious about `getdate(3)`.

My intuition was that it parsed from some standard format[^iso], but no! It actually does something
_much_ more dynamic: it looks up the `DATEMSK` environment variable, reads the text file
referenced by that variable, and then uses the _first matching pattern_ in that file to
parse the input.

In other words, if you had:

```bash
DATEMSK=/tmp/foo
```

...and `/tmp/foo`:

```
at %A the %dst of %B in %Y
```

Then `getdate("at monday the 1st of december in 1986")` should correctly parse that entire string[^entire]
into a `struct tm`.

This is a pretty strange API overall: the non-rentrancy[^gnu] makes it unsuitable for a lot of modern
applications, the use of a custom global (`getdate_err`) is funky, and the ambiguity inherent in
selection a "matching" format string likely makes any non-trivial use of `DATEMSK` unappealing.

But unlike many of POSIX's other weird or other unwieldy APIs, it doesn't appear to have been deprecated,
much less removed.

[^iso]: Something like ISO 8601, but not necessarily that.

[^gnu]: glibc supplies a non-standard `getdate_r` version that is re-entrant. I have no idea how common it is.

[^entire]: Unlike `strptime(3)` there's no "return a pointer to the next unparsed character" contract anyways.

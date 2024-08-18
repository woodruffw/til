---
title: RFC 5280 implies 0.999... ≠ 1
tags: [cryptography]
date: 2024-03-18
approx: true
origin: "https://groups.google.com/a/mozilla.org/g/dev-security-policy/c/-BogZx_IJyk/m/gHm3l613AgAJ"
---

[RFC 5280] is the foundational document behind the Internet PKI, and the base
standard behind the [CA/Browser Forum] [Baseline Requirements].

RFC 5280 defines validity periods (i.e., the span of time within which
an X.509 certificate is valid) at the maximum resolution of seconds:

> For the purposes of this profile, GeneralizedTime values MUST be
> expressed in Greenwich Mean Time (Zulu) and MUST include seconds
> (i.e., times are YYYYMMDDHHMMSSZ), even where the number of seconds
> is zero.  GeneralizedTime values MUST NOT include fractional seconds.

Meanwhile, in the real world, time is a continuous interval.

This results in an incongruity: `notBefore` and `notAfter` specify
their times at the second boundaries, but system clock times are significantly
more precise (and in theory could approach arbitrary precision).

The (weak) consensus from Internet PKI experts is that this should be resolved
by performing validity comparisons at the granularity offered by the
certificate, i.e. seconds. This implies truncation with no rounding, since
the comparison should not be aware of fractional components whatsoever.

In practice, this means that system times (expressed as `HHMMSS.SS`) are
interpreted as follows:

```
# 11:59:59 with different fractional seconds all transform to 11:59:59,
# even when the fractional second approaches 1.0 i.e. 12:00:00.
115959.00         -> 115959
115959.123        -> 115959
115959.4999999... -> 115959
115959.9999999... -> 115959

# 12:00:00 with different fractional seconds all transform to 12:00:00,
# even when the fractional second approaches 1.0 i.e. 12:00:01.
120000.00         -> 120000
120000.123        -> 120000
120000.4999999... -> 120000
120000.9999999... -> 120000
```

...which, of course, requires the assumption `0.999... ≠ 1`, since assuming
otherwise would enable an inductive proof that all X.509 certificates
are simultaneously valid and invalid, regardless of their validity ranges.

This might seem like a pointless and pedantic fixture of the Internet PKI, but
it caused [an actual compliance incident within Lets Encrypt]!

I've added testcases to [x509-limbo] for this:
`rfc5280::validity::notbefore-fractional` and `rfc5280::validity::notafter-fractional`.

[RFC 5280]: https://www.rfc-editor.org/rfc/rfc5280

[CA/Browser Forum]: https://cabforum.org/

[Baseline Requirements]: https://cabforum.org/working-groups/server/baseline-requirements/documents/

[x509-limbo]: https://x509-limbo.com/

[an actual compliance incident within Lets Encrypt]: https://bugzilla.mozilla.org/show_bug.cgi?id=1715455

---
title: Go's encoding/asn1 can produce invalid DER encodings
date: 2024-10-05
tags: [go, cryptography]
origin: personal correspondence
---

My colleague [Alexis Challande] discovered this and shared it with me;
we've reported it to Go as [golang/go#69782].

One of the coolest things about Go as a language (and ecosystem)
is its incredibly high-quality, large-and-yet-tastefully curated
[standard library].

One of the fantastic things that Go's stdlib comes with is
[`encoding/asn1`], which provides [DER] encoding and decoding APIs
via Go's (again, very tasteful!) `Marshal` and `Unmarshal` interfaces,
which use [struct tags] to reflect over a struct's members and provide
ser/de in a manner similar to Rust's [serde].

For example, the following struct definition:

```go
type MyTime struct {
    Time time.Time `asn1:"generalized"`
}
```

gets interpreted by `Marshal` and `Unmarshal` as representing the following
ASN.1 definition, in DER encoding:

```asn1
MyTime ::= SEQUENCE {
    time GeneralizedTime,
}
```

But there's a hiccup: if you push a local timezone-aware `time.Time` through
`asn1/encoding`'s `Marshal`, you might get something like this:

```
GeneralizedTime 2024-10-04 11:04:31 UTC+02:00
```

This *is* a valid `GeneralizedTime` but it's **not** a valid DER
encoding of a `GeneralizedTime`, per [X.690 11.7.1]:

> The encoding shall terminate with a "Z", as described in the
> ITU-T Rec. X.680 | ISO/IEC 8824-1 clause on GeneralizedTime

In other words: the only `GeneralizedTime` encodings that are valid in DER are
UTC ones, where the UTC time is encoded with `Z` (and *not* `+00:00` or any
other equivalent encoding).

Despite this, `encoding/asn1` currently supports relative timezone offsets,
as can be seen in the [`appendTimeCommon` helper]:

```go
func appendTimeCommon(dst []byte, t time.Time) []byte {
	_, month, day := t.Date()

	dst = appendTwoDigits(dst, int(month))
	dst = appendTwoDigits(dst, day)

	hour, min, sec := t.Clock()

	dst = appendTwoDigits(dst, hour)
	dst = appendTwoDigits(dst, min)
	dst = appendTwoDigits(dst, sec)

	_, offset := t.Zone()

	switch {
	case offset/60 == 0:
		return append(dst, 'Z')
	case offset > 0:
		dst = append(dst, '+')
	case offset < 0:
		dst = append(dst, '-')
	}

	offsetMinutes := offset / 60
	if offsetMinutes < 0 {
		offsetMinutes = -offsetMinutes
	}

	dst = appendTwoDigits(dst, offsetMinutes/60)
	dst = appendTwoDigits(dst, offsetMinutes%60)

	return dst
}
```

The end result: unless users are careful to explicitly convert their
`time.Time`s into UTC first (e.g. via [`Time.UTC()`]), they may accidentally
produce [BER] instead of DER (which will then be rejected by strict DER
decoders, like [rust-asn1]).

[standard library]: https://pkg.go.dev/std

[`encoding/asn1`]: https://pkg.go.dev/encoding/asn1

[DER]: https://en.wikipedia.org/wiki/X.690#DER_encoding

[struct tags]: https://go.dev/ref/spec#Tag

[serde]: https://serde.rs/

[Alexis Challande]: https://darkamaul.github.io/

[golang/go#69782]: https://github.com/golang/go/issues/69782

[X.690 11.7.1]: https://www.itu.int/ITU-T/studygroups/com17/languages/X.690-0207.pdf

[`appendTimeCommon` helper]: https://cs.opensource.google/go/go/+/refs/tags/go1.23.2:src/encoding/asn1/marshal.go;l=416-448

[`Time.UTC()`]: https://pkg.go.dev/time#Time.UTC

[BER]: https://en.wikipedia.org/wiki/X.690#BER_encoding

[rust-asn1]: https://docs.rs/asn1/latest/asn1/

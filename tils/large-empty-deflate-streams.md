---
title: Large, empty DEFLATE streams
date: 2026-11-17
tags: [compression]
---

[DEFLATE] is a very widely used compression algorithm, standardized in
[RFC 1951].

If you're like me and have very little experience with compression, you
might have some misconceptions about how compression algorithms work.
In particular, you might think that compression algorithms are
"canonical" or "minimal" by construction[^param], since there's very little
value in a stream of compressed data that's bigger than the underlying data.

This of course is not true: real-world compression algorithms are a lot more
like automata than canonical encodings like [DER]. DEFLATE is no exception to
this.

One of DEFLATE's interesting properties is that, for `N` such that `N >= 2`,
there exists a valid DEFLATE stream of `N` bytes that, when decompressed,
yields zero bytes.

Here are some empty encodings for `N` of 2 through 6:

| Size | Hex |
| ----- | --- |
| 2 | `03 00` |
| 3 | `02 0c 00` |
| 4 | `02 08 30 00` |
| 5 | `02 08 20 c0 00` |
| 6 | `02 04 00 00 ff ff` |

(These encodings are not unique: there are many encodings for each `N` that, when decoded,
yield zero bytes.)

And then, for the inductive step, here's a 5-byte non-final stored block that emits nothing:

```
00 00 00 ff ff
```

This can be prepended repeatedly, giving us the ability to produce a "compressed" stream
that's arbitrarily large that ultimately decompresses to nothing.

Does this matter? It's unclear. But it results in an intersting situation where compressed files
(e.g. ZIP archives) can contain extremely large members that actually decompress to
nothing[^zip].

[DEFLATE]: https://en.wikipedia.org/wiki/Deflate

[RFC 1951]: https://www.rfc-editor.org/info/rfc1951/

[DER]: https://letsencrypt.org/docs/a-warm-welcome-to-asn1-and-der/

[^param]: For some set of parameters.

[^zip]: ZIP's APPNOTE forbids this, but it's widely ignored in practice.

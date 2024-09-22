---
title: Python's base64.encodebytes has surprising behavior
date: 2024-09-19
tags: [python]
origin: https://github.com/pydantic/pydantic/issues/9072
---

If you've ever had to munge binary data in Python, you've probably used these
two APIs before:

```python
base64.b64encode(s, altchars=None)
base64.b64decode(s, altchars=None, validate=False)
```

However, there are actually two *other* base64 encode/decode functions
in the `base64` module:

```python
base64.encodebytes(s)
base64.decodebytes(s)
```

If you go by names alone, you might think that these two functions are
simple (legacy?) aliases of `b64encode` and `b64decode`, without the keyword
arguments present on the latter. But you'd be wrong!

```python
>>> import base64
>>> base64.b64encode(b"hello") == base64.encodebytes(b"hello")
False
```

Under the hood, `encodebytes` adds a trailing newline:

```python
>>> base64.b64encode(b"hello")
b'aGVsbG8='
>>> base64.encodebytes(b"hello")
b'aGVsbG8=\n'
```

...and the plot thickens when the (encoded) input is longer than 76 bytes:

```python
>>> base64.encodebytes(b"0" * 58)
b'MDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAw\nMA==\n'
```

Observe the additional newline, inserted so that no one encoded line exceeds 77
bytes (76 plus the newline itself).

Why is any of this? Because `encodebytes` and `decodebytes`, despite their
names, are **not** generic base64 APIs. Per the [Python docs], they are
intended **explicitly** for [RFC 2045], i.e. MIME:

> Encode the bytes-like object s, which can contain arbitrary binary data, and
> return bytes containing the base64-encoded data, with newlines (`b'\n'`)
> inserted after every 76 bytes of output, and ensuring that there is a
> trailing newline, as per [RFC 2045] (MIME).

How much does this matter? Not much for most people, who are probably used
to using `b64encode` and `b64decode` directly.

But it *might* matter if you, like me, use [Pydantic] and its
`Base64Bytes`/`Base64Bytes` types! From [Pydantic's docs]:

> Under the hood, Base64Bytes use standard library `base64.encodebytes` and
> `base64.decodebytes` functions.
>
> As a result, attempting to decode url-safe base64 data using the `Base64Bytes`
> type may fail or produce an incorrect decoding.

This note is correct, but IMO incomplete because it
doesn't mention the difference in behavior even in the non-URL-safe case;
I've followed up on [Pydantic's issue tracker] with a request for improvement.

[Python docs]: https://docs.python.org/3/library/base64.html#base64.encodebytes

[RFC 2045]: https://www.ietf.org/rfc/rfc2045.txt

[Pydantic]: https://github.com/pydantic/pydantic

[Pydantic's docs]: https://docs.pydantic.dev/dev/api/types/#pydantic.types.Base64Bytes

[Pydantic's issue tracker]: https://github.com/pydantic/pydantic/issues/9072#issuecomment-2361185555

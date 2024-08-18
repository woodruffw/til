---
title: "BoringSSL includes Chrome's libpki"
tags: [cryptography]
date: 2024-08-02
origin: personal correspondence
---

[BoringSSL] is Google's fork of [OpenSSL], which they created after
the [Heartbleed] fiasco.

As an outsider, I always thought BoringSSL was "just" a cleaned-up fork
of OpenSSL, with exceptionally old cruft removed and some APIs cleaned up.

However, it's actually much more than that: BoringSSL *also* contains
an [entire copy of Google's `libpki`], which is how Chromium (Chrome, etc.)
does path validation when the platform's X.509 stack isn't used.

BoringSSL's `libpki` is explicitly considered experimental and not
yet public, but you can experiment with it by telling the CMake-based build
to build the `pki` target:

```bash
git clone https://boringssl.googlesource.com/boringssl
cmake -G Ninja -B build
ninja -C build pki
```

...which can then be used with the `<openssl/pki>` headers, like
[`<openssl/pki/verify.h>`].

[BoringSSL]: https://boringssl.googlesource.com/boringssl

[OpenSSL]: https://www.openssl.org/

[Heartbleed]: https://en.wikipedia.org/wiki/Heartbleed

[entire copy of Google's `libpki`]: https://boringssl.googlesource.com/boringssl/+/master/pki/

[`<openssl/pki/verify.h>`]: https://boringssl.googlesource.com/boringssl/+/master/include/openssl/pki/verify.h

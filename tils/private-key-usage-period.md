---
title: "X.509: Private Key Usage Period"
tags: [cryptography]
date: 2024-07-04
origin: https://github.com/pyca/cryptography/issues/11195
---

I thought I knew the "standard" X.509v3 extensions pretty well, but
today I learned a new one: `2.5.29.16`, also known as the
"Private Key Usage Period."

From [RFC 3280 4.2.1.4]:

> The private key usage period extension allows the certificate issuer
> to specify a different validity period for the private key than the
> certificate.  This extension is intended for use with digital
> signature keys.  This extension consists of two optional components,
> notBefore and notAfter.  The private key associated with the
> certificate SHOULD NOT be used to sign objects before or after the
> times specified by the two components, respectively.  CAs conforming
> to this profile MUST NOT generate certificates with private key usage
> period extensions unless at least one of the two components is
> present and the extension is non-critical.

...and also from RFC 3280:

> This extension SHOULD NOT be used within the Internet PKI.  CAs
> conforming to this profile MUST NOT generate certificates that
> include a critical private key usage period extension.

...which explains why I haven't heard of it. RFC 5280 similarly
mentions it only in the context of explicitly **not** including it
in the defined X.509 profile.

The removal of this extension makes a lot of sense: it's not clear how
a certificate consumer should handle/interpret this extension, and being
able to decouple a certificate's lifetime from that of its underlying keypair
seems like a significant security and operational footgun.

Regardless, here's the ASN.1 definition:

```asn1
id-ce-privateKeyUsagePeriod OBJECT IDENTIFIER ::=  { id-ce 16 }

PrivateKeyUsagePeriod ::= SEQUENCE {
     notBefore       [0]     GeneralizedTime OPTIONAL,
     notAfter        [1]     GeneralizedTime OPTIONAL }
```

[RFC 3280 4.2.1.4]: https://www.rfc-editor.org/rfc/rfc3280#section-4.2.1.4

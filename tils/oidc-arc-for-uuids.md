---
title: UUIDs can be OIDs
tags: [cryptography, ontology]
date: 2024-07-25
origin: personal correspondence
---

[OIDs] are used in cryptography and a bunch of other domains (like medicine
and networking) to uniquely identify object and categorize them into "arcs,"
enabling the categorization and taxonimization of concepts.

For example, here's an OID for SHA2-256:

```
2.16.840.1.101.3.4.2.1
```

corresponding to a taxonomy of:

```
joint-iso-itu-t(2)
  | country(16)
    | us(840)
      | organization(1)
        | gov(101)
          | csor(3)
            | nistAlgorithms(4)
              | hashAlgs(2)
                | sha256(1)
```

[UUIDs] are another unique object identifying scheme, with a polar opposite
philosophy: UUIDs are (typically, with "v4" being the most common) random
identifiers, with uniqueness being guaranteed by a negligible chance of
collision rather than strict taxonimization.

On face value, these two schemes don't interoperate, meaning that a system
that deals in both OIDs and UUIDs needs to manually translate between
the two.

But no! The OID scheme includes an entire arc dedicated just to
subsuming UUIDs: [`2.25`]. Under the `2.25` arc, OIDs are represented
by the compact decimal equivalent of their normal hexadecimal form.

For example, the UUID `00000000-0000-0000-0000-000000000000` becomes:

```
2.25.0
```

...and the UUID `ae9f447-d440-4535-8755-56d61e36446e` becomes:

```
2.25.312254110780540278782586910815974474862
```

The UUID arc is one of the easiest "proper" ways to obtain an OID, since it
has no central authority. However, the length of UUID OIDs means that not
all applications will accept them, either due to overall OID length restrictions
or restrictions on individual segment lengths within an OID.

Finally, as a fun fact: the ITU-T has an official
government health warning for the UUID arc:

> Government health warning: It is important to note identical values for a UUID
> might be used, although the probability of this occurring is very small.
> The probability is increased if UUIDs are generated from MD5 hash values or
> pseudo-random numbers, rather than from SHA-1 hash values and
> cryptographic-quality random numbers. This may cause confusion for the users
> of the OID, and could be the trigger of malicious use such as spoofing.

[OIDs]: https://en.wikipedia.org/wiki/Object_identifier

[UUIDs]: https://en.wikipedia.org/wiki/Universally_unique_identifier

[`2.25`]: https://www.itu.int/en/ITU-T/asn1/Pages/UUID/uuids.aspx

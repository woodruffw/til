---
title: Remote Self derivation with serde for custom invariants
date: 2024-11-07
tags: [rust]
---

[`serde`] is one of the best things about the Rust ecosystem: it's a supercharged,
format agnostic version of Go's [struct tags], allowing ordinary developers
like myself to painlessly serialize and deserialize to any format that
can be made compatible with a basic [data model].

One thing I frequently run into with `serde` is having a data model,
and needing to *refine* that data model slightly via an invariant
that's hard to express with `serde`'s normal [attributes].

For example, consider this structure:

```rust
/// Invariant: one or both of `foo` and `bar` MUST be present.
#[derive(Deserialize)]
struct Foo {
    bar: Option<String>,
    baz: Option<String>,
}
```

This invariant is somewhat annoying to encode by default:

1. We could turn `Foo` into an `enum` with all possible typestates for maximum
   correctness, but that wouldn't be very ergonomic to work with.
2. We could manually [implement `Deserialize`] for `Foo`, but this is tedious,
   error-prone (especially as fields change), and loses much of the benefit of
   `serde`'s rich `derive` APIs.

Neither of these is great. But there's a secret third way:
[remote derivation]!

The TL;DR of this is that `serde` supports `#[serde(remote = "OtherType")]`
as a type-level attribute. Normally, this is used to provide ser/de derivations
on remote types, bypassing the [orphan rule] that limits third-party
trait implementations on third-party types.

Under the hood, `serde(remote = "OtherType")` works by defining an associated
function on the target type (`OtherType`) that implements the *signatures*
of the `Serialize` and `Deserialize` machinery but not the traits themselves.

From here, the trick is that `OtherType` doesn't have to be a different type
at all -- it can be `Self`! For deserialization, this means that
`serde(remote = "Self")` results in the creation of a "basic"
`Self::deserialize` function which can then be specialized through a
much shorter custom `Deserialize` implementation.

This can be hard to follow, so to use the `Foo` example above:

```rust
#[derive(Deserialize)]
#[serde(remote = "Self")]
struct Foo {
    bar: Option<String>,
    baz: Option<String>,
}

impl<'de> Deserialize<'de> for Foo {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: serde::Deserializer<'de>,
    {
        let foo = Self::deserialize(deserializer)?;

        if bar.is_none() && baz.is_none() {
            return Err(de::Error::custom(
                "at least one of `bar` or `baz` must be present",
            ));
        }

        Ok(foo)
    }
}
```

This is still slightly more verbose than a fully automatic
`#[derive(Deserialize)]` would be, but it gets us the best of both worlds:
`remote` does the bulk of the deserialization work for us, *and* allows us to
write our custom invariant.

[`serde`]: https://serde.rs

[struct tags]: https://go.dev/ref/spec#Tag

[data model]: https://serde.rs/data-model.html

[attributes]: https://serde.rs/attributes.html

[implement `Deserialize`]: https://serde.rs/impl-deserialize.html

[remote derivation]: https://serde.rs/remote-derive.html

[orphan rule]: https://doc.rust-lang.org/book/ch10-02-traits.html#implementing-a-trait-on-a-type

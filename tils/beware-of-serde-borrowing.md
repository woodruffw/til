---
title: serde's borrowing can be treacherous
date: 2025-12-25
tags: [rust]
---

(This is not super surprising when you think about it, but it bit me
recently so I figured I'd write it up.)

TL;DR: Be careful when using `&'a str` or `&'a [u8]` with serde deserializers;
serde has no way to produce an appropriate compile-time error when zero-copy
deserialization isn't possible or just isn't supported. Instead, you'll get
a runtime error indefinitely later.

[serde] is Rust's de facto standard serialization/deserialization framework.
It's also arguably one of the "crown jewels" of the Rust ecosystem, insofar
as it's broadly used and solves a thorny ergonomics problem (corresponding
the serialized representation of a structure to its type) *without*
imposting format-specific constraints on the types themselves[^go].

serde also has a degree of support for _zero-copy_ deserialization, i.e.
where the deserializer returns borrowed references into the input data,
instead of an owned copy. This is typically achievable for formats like
JSON, where the encoded form of a string corresponds to an appropriate
borrowed form:

```json
{ "foo": "this is a string" }
          ^^^^^^^^^^^^^^^^
          |
          +-- representable as &str
```

To take advantage of this, we can pull `&str` out of a `Deserializer`
instead of `String`. This even works in the derive APIs:

```rust
use serde::Deserialize;

#[derive(Deserialize)]
struct Example<'a> {
    foo: &'a str,
}
```

([Playground example](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=43c3a0536d2a5431d666fa6beb779d29))

However, there's a significant caveat to this: what happens with
this kind of input?

```json
{ "foo": "this is a string\nwith two lines" }
          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
          |
          +-- representable as &str?
```

Instead of an `Example { ... }`, we get an error at runtime:

```rust
Err(Error("invalid type: string \"this is a string\\nwith two lines\", expected a borrowed string", line: 1, column: 42))
```

This error is a little confusing because we *gave* serde a borrowed string type,
and there's nothing obviously wrong with the string it errored on. But what
serde is saying is that it _wanted_ to borrow a `&str` from the input, but
it _couldn't_.

Why not? Because the string contains an escape (`\n` in this case), which
serde needs to decode into an actual newline character. That means mutation,
not just a reference to the input, which in turn means that there's no possible
way for serde to return a `&str` that points into the _original_ input data.

Unfortunately this only surfaces at runtime, since serde doesn't have any
way to know ahead of time whether a particular input can be deserialized
without copying. This is particularly true in human-readable formats like
JSON, where escapes are common.

(Also, there are serde deserializers like `serde-yaml`, which have very
limited support for borrowing despite happy cases similar to those of JSON.)

There are two workarounds to this:

1. Use fully owned types, e.g. `String` instead of `&str`.
2. Use `Cow<'a, str>` along with `#[serde(borrow)]` on the field.
   Both are needed, since by default serde will select `Cow::Owned`
   due to a blanket implementation. See [#1852] for some details.

[#1852]: https://github.com/serde-rs/serde/issues/1852

For the example above, this means:

```rust
use serde::Deserialize;
use std::borrow::Cow;

#[derive(Deserialize)]
struct Example<'a> {
    #[serde(borrow)]
    foo: Cow<'a, str>,
}
```

([Playground example](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=15cf283dd6a14564efb49cc4da7e78bd))

[serde]: https://serde.rs/

[^go]: In marked contrast to Go, where each encoding has its own tag-based DSL.

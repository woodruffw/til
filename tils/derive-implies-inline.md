---
title: Rust's derive often implies inline
date: 2026-10-03
tags: [rust]
---

In Rust, one of the most common ways to implement core traits (like
`Debug`, `Display`, and `Clone`) is to `#[derive(...)]` them, e.g.:

```rust
#[derive(Debug)]
struct Widgets {
  foo: u32,
  bar: usize,
}
```

What I _didn't_ know until recently is that Rust currently emits
`#[inline]` as part of these derivations. This is seemingly
not guaranteed, but is [implied by example in the reference]
and can also be seen if one expands the macros.

Using the example above, this is what you get when you expand
the `#[derive(Debug)]` [in the playground]:

```rust
struct Widgets {
    foo: u32,
    bar: usize,
}
#[automatically_derived]
impl ::core::fmt::Debug for Widgets {
    #[inline]
    fn fmt(&self, f: &mut ::core::fmt::Formatter) -> ::core::fmt::Result {
        ::core::fmt::Formatter::debug_struct_field2_finish(f, "Widgets",
            "foo", &self.foo, "bar", &&self.bar)
    }
}
```

This is _almost_ always what we want: `#[inline]` is just a hint,
and typical derived `Debug`, `Clone`, etc. implementations benefit
from being inlined (since they're often trivial).

But not always! Imagine an error hierarchy like this[^simple]:

```rust
#[derive(Debug)]
struct ErrorA {
  lots: String,
  of: String,
  chunky: String,
  fields: String,
  within: String,
  this: String,
  r#type: String,
}

#[derive(Debug)]
struct ErrorB {
  inner: ErrorA,
}

#[derive(Debug)]
struct ErrorC {
  inner: ErrorB,
}

#[derive(Debug)]
enum Errors {
    A(ErrorA),
    B(ErrorB),
    C(ErrorC),
}
```

produces:

```rust
struct ErrorA {
    lots: String,
    of: String,
    chunky: String,
    fields: String,
    within: String,
    this: String,
    r#type: String,
}
#[automatically_derived]
impl ::core::fmt::Debug for ErrorA {
    #[inline]
    fn fmt(&self, f: &mut ::core::fmt::Formatter) -> ::core::fmt::Result {
        let names: &'static _ =
            &["lots", "of", "chunky", "fields", "within", "this", "type"];
        let values: &[&dyn ::core::fmt::Debug] =
            &[&self.lots, &self.of, &self.chunky, &self.fields, &self.within,
                        &self.this, &&self.r#type];
        ::core::fmt::Formatter::debug_struct_fields_finish(f, "ErrorA", names,
            values)
    }
}
#[automatically_derived]
impl ::core::default::Default for ErrorA {
    #[inline]
    fn default() -> Self {
        Self {
            lots: ::core::default::Default::default(),
            of: ::core::default::Default::default(),
            chunky: ::core::default::Default::default(),
            fields: ::core::default::Default::default(),
            within: ::core::default::Default::default(),
            this: ::core::default::Default::default(),
            r#type: ::core::default::Default::default(),
        }
    }
}

struct ErrorB {
    inner: ErrorA,
}
#[automatically_derived]
impl ::core::fmt::Debug for ErrorB {
    #[inline]
    fn fmt(&self, f: &mut ::core::fmt::Formatter) -> ::core::fmt::Result {
        ::core::fmt::Formatter::debug_struct_field1_finish(f, "ErrorB",
            "inner", &&self.inner)
    }
}

struct ErrorC {
    inner: ErrorB,
}
#[automatically_derived]
impl ::core::fmt::Debug for ErrorC {
    #[inline]
    fn fmt(&self, f: &mut ::core::fmt::Formatter) -> ::core::fmt::Result {
        ::core::fmt::Formatter::debug_struct_field1_finish(f, "ErrorC",
            "inner", &&self.inner)
    }
}

enum Errors { A(ErrorA), B(ErrorB), C(ErrorC), }
#[automatically_derived]
impl ::core::fmt::Debug for Errors {
    #[inline]
    fn fmt(&self, f: &mut ::core::fmt::Formatter) -> ::core::fmt::Result {
        match self {
            Self::A(__self_0) =>
                ::core::fmt::Formatter::debug_tuple_field1_finish(f, "A",
                    &__self_0),
            Self::B(__self_0) =>
                ::core::fmt::Formatter::debug_tuple_field1_finish(f, "B",
                    &__self_0),
            Self::C(__self_0) =>
                ::core::fmt::Formatter::debug_tuple_field1_finish(f, "C",
                    &__self_0),
        }
    }
}
```

That's a lot of code that can get inlined for each invocation of the `Debug`
implementation of `Errors`, which can occur repeatedly in e.g. debug or trace logging.

In fact, it's so much code that it can turn out to be a non-trivial amount of a Rust
binary's total size: we found that we could shrink uv's binary size by approximately
160KB by preventing[^how] `rustc` from inlining a given `Debug` implementation.

This was surprising to me on two levels: the size cost added up fast, and `rustc`
(seemingly) did not apply a limit to the size or number of times a `Debug` implementation
was inlined. I suspect this is the right decision in many programs, however!

[implied by example in the reference]: https://doc.rust-lang.org/reference/attributes/derive.html

[in the playground]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=bf60dea24f7278ad716ebc2a43f99a5e

[^simple]: This hierarchy is _drastically_ simplified: real world Rust applications often have
           deeply nested error enumerations with nontrivial numbers of fields.

[^how]: We did this by adding our own-proc macro that behaves like `derive(Debug)`, but
        with `#[inline(never)]`.

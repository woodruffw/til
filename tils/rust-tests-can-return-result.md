---
title: Rust unit tests can return Result
date: 2024-11-01
tags: [rust, testing]
origin: https://github.com/woodruffw/zizmor/pull/97
---

I'm only 6 years late on this one: Rust's unit test support has allowed
`#[test]`-annotated units to return `Result<()>`
[since the 2018 edition of Rust], meaning that they can use the `?` syntax like
every other function.

The old way to do things is to explicitly `unwrap`/`expect` everywhere,
since the test could only return `()` (i.e. unit):

```rust
#[cfg(test)]
mod tests {
    use super::something_fallible;

    #[test]
    fn test_thing() {
        assert_eq!(something_fallible().unwrap(), 42);
    }
}
```

Whereas the new way:

```rust
#[cfg(test)]
mod tests {
    use anyhow::Result;
    use super::something_fallible;

    #[test]
    fn test_thing() -> Result<()> {
        assert_eq!(something_fallible()?, 42);
    }
}
```

This is only a small improvement in the contrived example above, but it
makes larger tests with lots of result unwrapping much more legible!

[since the 2018 edition of Rust]: https://doc.rust-lang.org/rust-by-example/testing/unit_testing.html#tests-and-

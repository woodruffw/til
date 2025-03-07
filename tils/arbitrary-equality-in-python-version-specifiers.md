---
title: Arbitrary equality in Python version specifiers
date: 2025-03-07
tags: [python]
---

Python's version specifiers are specified in the
[Version Specifiers Specification] (say that 5 times fast).

Most people are familiar with the common specifiers: `foo == 1.0` means
"exactly version 1.0", `foo >= 1.0` means "version 1.0 or greater,"
`foo ~= 1.1` means "version 1.1 or greater, but less than 2.0,"
and so on. These are all commonplace in `requirements.txt` files,
`pyproject.toml`s, &amp;c.

Like most people, I thought that `==` was the only equality
operator in the version specification. But that's wrong!

Under ["Arbitrary equality"]:

> Arbitrary equality comparisons are simple string equality operations which do not
> take into account any of the semantic information such as zero padding or local
> versions. This operator also does not support prefix matching as the `==` operator does.
>
> The primary use case for arbitrary equality is to allow for specifying a
> version which cannot otherwise be represented by this specification. This
> operator is special and acts as an escape hatch to allow someone using a tool
> which implements this specification to still install a legacy version which is
> otherwise incompatible with this specification.
>
> An example would be `===foobar` which would match a version of `foobar`.
>
> This operator may also be used to explicitly require an unpatched version of a
> project such as `===1.0` which would not match for a version `1.0+downstream1`.
>
> Use of this operator is heavily discouraged and tooling MAY display a warning when it is used.

In short, you can also write `foo===1.0`, which means "exactly the literal
string `1.0`, and not `1.0.0` or any other equivalent parsed version."

This has no practical use in the modern packaging ecosystem, since
most tools and build backends expect [well-formed versions].

[Version Specifiers Specification]: https://packaging.python.org/en/latest/specifications/version-specifiers/#version-specifiers

["Arbitrary equality"]: https://packaging.python.org/en/latest/specifications/version-specifiers/#arbitrary-equality

[well-formed versions]: https://packaging.python.org/en/latest/specifications/version-specifiers/#version-scheme

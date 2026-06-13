---
title: Wrapping rustc for just the current workspace
date: 2026-06-13
tags: [rust]
---

I've known about `RUSTC_WRAPPER` for a while: if set, `cargo` will invoke
it instead of `rustc` directly. The wrapper in turn receives the "real"
`rustc` as its first argument, meaning that the following works
as a trivial echoing wrapper:

```shell
#!/bin/sh

next="${1}"
shift

echo "wrapped: ${@}"
"${next}" "${@}"
```

...then, to use it:

```shell
chmod +x wrapper.sh
export RUSTC_WRAPPER=$(readlink -f wrapper.sh)
```

This is useful for instrumenting every single `rustc` invocation that
`cargo` might make, but it can also be incredibly noisy:
often times what you _really_ want is to instrument only the calls that
are relevant to the crate (or crates) in your own workspace.
Doing this manually with `RUST_WRAPPER` is tedious and brittle.

Fortunately, `cargo` provides does it for us! `RUSTC_WORKSPACE_WRAPPER`
behaves exactly like `RUSTC_WRAPPER`, except that it only
gets invoked for workspace members.

This makes it ideal when e.g. instrumenting a build for coverage:
we only care about coverage within the workspace, so being able
to limit `-C instrument-coverage` to just those workspace members
makes both the build and any instrumented runs significantly faster.

`RUSTC_WORKSPACE_WRAPPER` is [documented here] in the Cargo book.
It's been available [since 2019].

[documented here]: https://doc.rust-lang.org/cargo/reference/config.html#buildrustc-workspace-wrapper

[since 2019]: https://github.com/rust-lang/cargo/pull/7533

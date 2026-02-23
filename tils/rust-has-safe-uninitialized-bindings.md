---
title: Rust has safe uninitialized bindings
date: 2026-02-22
tags: [rust]
---

I've been writing Rust for at least 6 or 7 years at this point, but recently
came across a construct I hadn't seen before:

```rust
let workspace;
let target = if let Some(script) = script.as_ref() {
    LockTarget::Script(script)
} else {
    workspace =
        Workspace::discover(project_dir, &DiscoveryOptions::default(), &workspace_cache)
            .await?;
    LockTarget::Workspace(&workspace)
};
```

(This is in [uv]'s codebase, e.g. [here]).

[uv]: https://github.com/astral-sh/uv

[here]: https://github.com/astral-sh/uv/blob/563c44984cbc1ff7e03c8488976ae0b4ed508ab7/crates/uv/src/commands/project/tree.rs#L68-L76

I hadn't seen the `let binding;` syntax before; I had only ever seen `let binding = value;`
(and correspondingly had been under the impression that bindings *must* be initialized
at their point of introduction).

But this was wrong (as evidenced by the above working), and Rust covers this
under the ["Declare first"] section in Rust by Example!

So, why would you want to do this? In the case above, `LockTarget` is defined as follows:

```rust
#[derive(Debug, Copy, Clone)]
pub(crate) enum LockTarget<'lock> {
    Workspace(&'lock Workspace),
    Script(&'lock Pep723Script),
}
```

In plain English, `LockTarget` holds a reference (of either type) of lifetime `'lock`. Therefore,
the value borrowed for `'lock` must live _at least_ as long as the `LockTarget` itself.

Thus, if we were to write the above code more naively:

```rust
let target = if let Some(script) = script.as_ref() {
    LockTarget::Script(script)
} else {
    let workspace =
        Workspace::discover(project_dir, &DiscoveryOptions::default(), &workspace_cache)
            .await?;
    LockTarget::Workspace(&workspace)
};
```

...we'd get a lifetime error, since `else { ... }` introduces a new scope and
`workspace` is dropped at the end of that scope, whereas `target` outlives the scope. By declaring `workspace` first,
we take advantage of Rust's ability to enforce initialization *and* communicate the correct
lifetime in the process.

([Here's a minimal example of the above](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=c7c647728838b960c2ec74a34adc1758).)

I revealed my ignorance on this matter to [Gankra], who showed me an even more interesting
example:

```rust
let x;
loop {
  // ...
  if cond {
    x = vec![];
    break;
  }
}

frob(x);
```

This works because Rust can see that there's no possible uninitialized load of `x`:
either the loop never terminates and `x` is never read, or the loop terminates
(via the `break`) after `x` has been initialized.

[Gankra]: https://github.com/gankra

["Declare first"]: https://doc.rust-lang.org/rust-by-example/variable_bindings/declare.html

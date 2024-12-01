---
title: Snapshot testing with insta
date: 2024-11-29
tags: [rust, testing]
---

Snapshot testing (also known as "golden testing" or
"known-answer testing"[^crypto]) is one of my favorite testing techniques:
when used properly and in the right context, it dramatically reduces the
overhead of integration testing by measuring *deviance* from a previous
result instead of incongruity with an inflexible expected result. LLVM
(via [lit]) is a good example of the extreme efficacy of golden testing
in the real world.

In practice, one of the biggest drawbacks of snapshot testing is *poor UX*:
it's relatively easy to *record* snapshots, but much harder to build
developer-friendly tooling for *comparing* and accepting/rejecting snapshot
changes between runs.

Today I discovered Armin Ronacher's [insta] ([docs]), which appears to win at
providing an extremely smooth, intuitive snapshot testing flow.

To start, I added `insta` to [zizmor]'s dev-deps:

```bash
cargo add --dev insta
```

Then, I installed `cargo-insta` as `insta`'s snapshot management tool. This
tool is strictly optional, but I found that it made things very smooth:

```bash
cargo install --locked cargo-insta
```

Then, I wrote some basic snapshot tests (you can find these [here]):

```rust
#[test]
fn self_hosted() -> Result<()> {
    insta::assert_snapshot!(zizmor()
        .workflow(workflow_under_test("self-hosted.yml"))
        .args(["--pedantic"])
        .run()?);

    insta::assert_snapshot!(zizmor()
        .workflow(workflow_under_test("self-hosted.yml"))
        .run()?);

    Ok(())
}
```

These tests work seamlessly with `cargo test`, except that they fail by default.
This makes sense, since I hadn't *accepted* the snapshot results yet!

To accept the snapshots, I used `cargo insta`:

```bash
cargo insta test --review
```

That popped up an interactive prompt showing me each new snapshot (or a
nicely formatted diff, when reviewing snapshot changes), allowing me to
review each.

Once my snapshots were accepted, I ran `cargo test` again and
confirmed that my tests pass, as the outputs now match my accepted
snapshots:

```console
$ cargo test --test snapshot
   Compiling zizmor v0.6.0 (/path/to/devel/zizmor)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.85s
     Running tests/snapshot.rs (target/debug/deps/snapshot-7207efbcffc6c7ac)

running 2 tests
test unpinned_uses ... ok
test self_hosted ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.72s
```

Success! This is by far the smoothest snapshot/golden testing workflow I've
ever used.

Other bits: there's [insta-cmd] for CLI snapshot testing, which I should
probably be using. Maybe soon.

[lit]: https://llvm.org/docs/CommandGuide/lit.html

[zizmor]: https://github.com/woodruffw/zizmor

[insta]: https://github.com/mitsuhiko/insta

[docs]: https://insta.rs/docs/

[here]: https://github.com/woodruffw/zizmor/blob/main/tests/snapshot.rs

[insta-cmd]: https://github.com/mitsuhiko/insta-cmd

[^crypto]: This term seems to mostly be used in the context of cryptography.

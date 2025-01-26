---
title: GitHub Actions is surprisingly case-insensitive
date: 2025-01-26
tags: [github, github-actions, security]
---

GitHub Actions is case-insensitive in a few places that are surprising,
at least to me:

- Expressions ignore case when comparing strings. For example, the following
  evaluates to `true` even if the branch is named `MAIN` or `mAiN`:

    ```
    ${{ github.ref == 'refs/heads/main' }}
    ```

    Example: [woodruffw-experiments/actions-experiments#2](https://github.com/woodruffw-experiments/actions-experiments/pull/2).

- Some (maybe not all?) context accesses appear to be case-insensitive.
  For example, `secrets` can be accessed via `SECRETS` and other casings.
  Secret names themselves are also case-insensitive.

    ```yaml
    - run: echo ${foo} ${bar} ${baz}
      env:
        foo: ${{ SECRETS.foo }}
        bar: ${{ Secrets.BAR }}
        baz: ${{ SeCRetS.Baz }}
    ```

- Expression functions are case-insensitive. For example, `toJSON`
  can be written as `tojson` or `TOJSON`.

Of these, the [first] and [second] are documented. The third appears to be
publicly undocumented, but is mentioned in a comment in
[actions/languageservices].

Whether or not this insensitivity is a security risk is difficult to assert
generally: `git` itself is mostly case sensitive, so any disconnect
between the user's *intent* and what GitHub Actions *does* is potentially
exploitable.

In particular, case insensitivity during string comparisons
strikes me as particularly risky, since a lot of projects have workflows
that treat branch names as unique for security/isolation purposes. The fact
that a branch named `RELEASE` can trigger a workflow that expects `release` is
potentially dangerous.

[first]: https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/evaluate-expressions-in-workflows-and-actions#operators

[second]: https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions#naming-your-secrets

[actions/languageservices]: https://github.com/actions/languageservices/blob/3a8c29c2df26d94c6fd07c7a918a0dfabedba685/expressions/src/funcs.ts#L28-L33

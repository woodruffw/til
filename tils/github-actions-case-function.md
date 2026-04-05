---
title: GitHub Actions has a new 'case' function
date: 2026-04-04
tags: [github-actions]
approx: true
---

I actually found out about this a few weeks ago, but forgot to write it up.

It seems like GitHub added a `case` function to GitHub Actions' expression
mini-language [at the end of January 2026], completing the functions
we all already know and ~love so well~ use so frequently (like `toJSON`, `constains`, etc.).
They announced it in that blog post (and updated [the docs]) too, but it
doesn't seem to have otherwise received much attention (or fanfare).

This is unfortunate, because it's a great change! Previously one would write
something messy like this:

```yaml
run: |
  ./some-incredibly-important-task.sh
env:
  ENVIRONMENT: ${{ github.ref_name == 'main' && 'production' || github.ref_name == 'staging' && 'staging' || 'development' }}
```

...but now we can write:

```yaml
run: |
  ./some-incredibly-important-task.sh
env:
  ENVIRONMENT: ${{ case(github.ref_name == 'main', 'production', github.ref_name == 'staging', 'staging', 'development') }}
```

That's a bit clearer (in my opinion) and doesn't rely on any precedence rules or short-circuiting
to evaluate correctly. This is particularly beneficial in the presence of this kind of footgun:

```yaml
env:
  # the user meant to evaluate to '' when `condition` holds, but '' is falsey so
  # the expression evaluates to 'value' instead
  VARIABLE: ${{ condition && '' || 'value' }}
```

([zizmor] will have [dedicated support] for analyzing `case` calls in the next release).

[zizmor]: https://docs.zizmor.sh/

[dedicated support]: https://github.com/zizmorcore/zizmor/pull/1828

[at the end of January 2026]: https://github.blog/changelog/2026-01-29-github-actions-smarter-editing-clearer-debugging-and-a-new-case-function/

[the docs]: https://docs.github.com/en/actions/learn-github-actions/expressions#case

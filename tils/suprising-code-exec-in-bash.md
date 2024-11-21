---
title: Some surprising code execution sources in bash
date: 2024-11-15
tags: [shell, security]
---

I ran across two surprising sources of code execution in `bash` (and probably
other shells) recently.

In a historic context these probably weren't *too*
serious of a problem, but in the context of CI systems where everything is
a rats' nest of shell and YAML they could be useful execution primitives.

## Source 1: arithmetic expressions (a.k.a. "white-collar eval")

Leading question aside, do you think this snippet of `bash`[^source] can run
arbitrary code?

```bash
function guess() {
  num="${1}"
  if [[ "${num}" -eq 42 ]]
  then
    echo "Correct"
  else
    echo "Wrong"
  fi
}
```

Most people (including experienced shell programmers[^poll]) say "no": they
recognize that there *could* have been a splatting bug if it was `$num` instead
of `"${num}"`, but the double quoting should firmly prevent any evaluation
of the `num` variable itself.

But nope: because of `-eq`, `num` is treated with `bash`'s arithmetic evaluation
rules, meaning that this works:

```bash
$ guess 'a[$(cat /etc/passwd > /tmp/pwned; true)] + 42'
Correct
$ cat /tmp/pwned
```

Note the single quotes: `$(cat /etc/passwd > ~/pwned)` is **not** executed
eagerly as a parameter to `guess`, but as part of the evaluation of `-eq`
within `[[`.

This *also* works with `[` and `test`, so long as their builtin variants
are used instead of their external binary variants. `/usr/bin/[` and
`/usr/bin/test` will of course not work, since they have no access to the
context of the shell that spawned them.

## Source 2: `test -v`

The same surprising code execution source exists with `test -v var`, under
the same conditions as arithmetic expressions (needs to use the builtin,
not the standard binary):

```bash
$ [[ -v 'x[$(cat /etc/passwd > /tmp/pwned)]' ]]
$ cat /tmp/pwned
```

I'm not 100% sure *why* this is the case, since `-v var` is documented as
testing whether `var` is set and it shouldn't be necessary to evaluate
a subscript to determine that.

[^source]: Minimized and tweaked from [Vidar Holen's blog](https://www.vidarholen.net/contents/blog/?p=716), where I learned about this!

[^poll]: I polled my coworkers: of 17 respondents, 16 through this snippet was fine and 1 thought it contained a potential vulnerability.

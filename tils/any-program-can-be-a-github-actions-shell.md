---
title: Any program can be a GitHub Actions shell
date: 2025-04-07
tags: [github-actions, security]
---

In GitHub Actions, you can use the `shell` keyword to specify the shell
that runs a given `run:` block. This keyword is optional for workflows
but mandatory for action definitions.

The shell normally defaults to something sensible for your runner,
e.g. `bash` on Linux and macOS, and `pwsh` on Windows. But it can also be
specified, and [GitHub documents] that specifying it explicitly also
implies some flags of their choosing:

[GitHub documents]: https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#defaultsrunshell

```yaml
# setting bash explicitly implies bash --noprofile --norc -eo pipefail
- shell: bash
  run: |
    echo "Hello, world!"
```

Based on that, you might think that there's a fixed number of valid
`shell` values[^toolcache], which GitHub tracks and adds their special flags to.
But you'd be wrong!

As it turns out, you can set `shell` to any executable on the `$PATH`,
and GitHub will happily use it to execute the `run` block. If the command
doesn't already take a single file as input, you need to pass it
the special `{0}` argument, which GitHub replaces with the temporary file
that it generates the template-expanded `run` block into.

Thanks to this, we can do all kinds of cursed things.

Using C as our step runner [works fine]:

```yaml
- run: sudo apt install -y tcc
- shell: tcc -run {0}
  run: |
    #include <stdio.h>
    int main() {
      printf("Hello, world!\n");
      return 0;
    }
```

...and so does [dynamically modifying] the `$PATH` via `$GITHUB_PATH`:

```yaml
- run: |
    touch ./bash
    chmod +x ./bash
    echo '#!/bin/sh' > ./bash
    echo 'echo hello from fake bash' >> ./bash
    echo "${PWD}" >> "${GITHUB_PATH}"
- run: |
    echo "this doesn't do what you expect"
  shell: bash
```

[works fine]: https://github.com/woodruffw-experiments/actions-experiments/pull/3

[dynamically modifying]: https://github.com/woodruffw-experiments/actions-experiments/pull/5

Does this matter from a security perspective? It's hard to say &mdash; there
are plenty of other ways in which file writes imply execution in GitHub Actions,
including `GITHUB_ENV` itself.

On the other hand, it's surprising (IMO) that GitHub does `$PATH` lookups even
for their "well-known" shell values; I would expect those to be fixed to
specific values (e.g. `bash` to `/bin/bash`), especially since they inject
flags into the `run`'s command line based on those well-known values.

[^toolcache]: Or, more generally, that GitHub will accept any pre-registered tool in its toolcache. That, for example, is how I thought `shell: python` worked, before I noticed this.

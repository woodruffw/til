---
title: "POSIX wordexp(3) is dangerous"
tags: [c, posix, security]
date: 2026-10-25
---

[`wordexp(3)`] is a POSIX API for performing shell-style expansions from C.

It's often cited/recommended as a way to do tilde expansion (e.g. `~/foo` to `/home/me/foo`):

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <wordexp.h>

int expand_tilde(const char *input, char **output) {
    wordexp_t p;
    int ret = wordexp(input, &p, 0);
    if (ret == 0) {
        *output = strdup(p.we_wordv[0]);
        wordfree(&p);
    }
    return ret;
}

int main(void) {
    char *expanded = NULL;
    expand_tilde("~/foo", &expanded);
    printf("expanded: %s\n", expanded);
    return 0;
}
```

However, `wordexp(3)` is also dangerous! That last `0` argument is for the _flags_ parameter.
By default, all expansion features are enabled, **including command expansion**:

```c
expand_tilde("$(whoami)/foo", &expanded);
```

or:

```c
expand_tilde("$(cat /etc/passwd)/foo", &expanded);
```

Disabling this requires passing [`WRDE_NOCMD`] as a flag. Unfortunately, this
palliative at best in some libc implementations:

- [musl implements `wordexp(3)`] by delegating to the system shell. It adds some
  perfunctory scanning for command substitutions before delegating, but
  fails to match some patterns that the underlying shell (like GNU Bash) can treat
  as valid command substitutions.

- [macOS's libc implements `wordexp(3)`] in a similar manner, delegating to
  `/usr/lib/system/wordexp-helper`[^perl]. According to [Julian Squires]
  this had the same bypass as musl [as recently as 2023], although I didn't bother
  testing whether it's still effective now that the default macOS shell is `zsh`
  instead of `bash`.

Bottom line: don't use `wordexp(3)` on untrusted inputs, even with `WRDE_NOCMD`.

[`wordexp(3)`]: https://pubs.opengroup.org/onlinepubs/9799919799/functions/wordexp.html

[`WRDE_NOCMD`]: https://pubs.opengroup.org/onlinepubs/9799919799/functions/wordexp.html#:~:text=WRDE_NOCMD,Substitution%2C%20is%20requested.

[musl implements `wordexp(3)`]: https://git.musl-libc.org/cgit/musl/tree/src/misc/wordexp.c

[macOS's libc implements `wordexp(3)`]: https://github.com/apple-oss-distributions/Libc/blob/71bbe350ab79eef58113991d817ccc6165061a64/gen/FreeBSD/wordexp.c#L201

[Julian Squires]: https://www.cipht.net

[as recently as 2023]: https://www.cipht.net/2023/11/21/worried-by-wordexp.html

[^perl]: Previously, macOS's `wordexp(3)` would shell out to Perl. `wordexp-helper` appears to be the "modern" solution.

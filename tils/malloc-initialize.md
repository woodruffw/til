---
title: Returning only initialized memory with malloc
date: 2024-08-13
tags: [c, security]
origin: personal correspondence
---

This is technically a "today I re-learned," since I knew this years ago
and had to go digging for it.

On most Operating Systems, the `malloc(3)` API returns *uninitialized* memory:
the program is expected to immediately overwrite the memory anyways, so
(in theory) there's no value to the system initializing it before returning it.

In practice, however, this is a consistent source of security bugs:
when combined with length validation bugs (or an attacker's ability to
induce memory reads e.g. over the network), uninitialized memory can result in
cryptographic material disclosure (see [Heartbleed]) or disclosure of machine
state (e.g. heap addresses) that can make [ASLR] and other mitigation bypasses
simpler.

Some OSes (e.g. [recent versions of XNU]) partially mitigate this by
zeroing allocations on `free(3)`, but this is only an improvement in the UAF
case, not the "uses and/or leaks uninitialized memory from the start" case.

But that doesn't mean initialized `malloc(3)` doesn't exist! Many of the major
`libc` implementations *do* provide it, it's just hidden away.

On macOS (and presumably iOS), among [other settings]:

```bash
# on malloc, set every byte to 0xAA
MallocPreScribble=1
```

and with `glibc`:

```bash
# on malloc, set every byte to 0x7F
MALLOC_PERTURB_=127
```

`glibc`'s setting can also be performed via [`mallopt(3)`] or the more generic
[tunables] environment setting.

These should really be the default, especially when any larger set of
runtime hardening options is enabled. Unfortunately, they don't seem to be.

[Heartbleed]: https://heartbleed.com/

[ASLR]: https://en.wikipedia.org/wiki/Address_space_layout_randomization

[recent versions of XNU]: https://security.apple.com/blog/towards-the-next-generation-of-xnu-memory-safety/

[other settings]: https://developer.apple.com/library/archive/documentation/Performance/Conceptual/ManagingMemory/Articles/MallocDebug.html

[`mallopt(3)`]: https://linux.die.net/man/3/mallopt

[tunables]: https://www.gnu.org/software/libc/manual/html_node/Tunables.html

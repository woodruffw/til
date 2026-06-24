---
title: Python has a standard ChainMap API
tags: [python]
date: 2026-06-23
---

I thought I knew all of the stdlib
[`collections`](https://docs.python.org/3/library/collections.html) APIs,
but I learned about a new (very old) one today:
[`collections.ChainMap`](https://docs.python.org/3/library/collections.html#collections.ChainMap).

`ChainMap` does exactly what it sounds like: it's a combinator that chains
mappings together into a single (and even updateable) view. Reads search
the entire chain in order; writes modify the first mapping in the chain.

By example:

```python
from collections import ChainMap

xs = {"a": 1, "b": 2}
ys = {"b": "dupe", "c": 3}
zs = ChainMap(xs, ys)

print(zs["b"]) # => prints 2, not "dupe"

zs["d"] = 4
assert "d" in xs
assert "d" not in ys
```

This is pretty convenient (more so for the read path than the write path).
I've certainly open-coded a version of this more than dozen times over the years,
not knowing that it was in the standard library this whole time.

---
title: Python 3.15's new buffer size
tags: [python, performance]
date: 2025-07-16
origin: personal correspondence
---

Python's default buffer size when doing buffered I/O has been 8 KB for a
very long time (at least 16 years, according to the issue linked below):

```python
>>> import io
>>> io.DEFAULT_BUFFER_SIZE
8192
```

Consequently, I've put some version of `_BUFFER_SIZE = 128 * 1024` in
countless programs to get a free performance boost.

But not for much longer! Python 3.15 will change `io.DEFAULT_BUFFER_SIZE`
to 128 KB, per [gh#117151](https://github.com/python/cpython/issues/117151).

More precisely, platforms that provide `st_blksize` via `stat(2)` will use
`max(min(st_blksize, 8 MiB), io.DEFAULT_BUFFER_SIZE)` for `open()`. That means
all buffered I/O will use 128 KB buffers at the absolute minimum on Linux,
even if the filesystem reports a tiny block size like 4 KB.

Many thanks to [Romain Morotti](https://github.com/morotti) for implementing
this change.

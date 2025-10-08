---
title: Python's splitlines does a lot more than just newlines
date: 2025-10-08
tags: [python]
---

(With thanks to [Seth Larson] for taking me down this rabbit hole.)

I always assumed that Python's [`str.splitlines()`] split strings by
"[universal newlines]", i.e., `\n`, `\r`, and `\r\n`.

But it turns out it does a lot more than that. From the docs:

> This method splits on the following line boundaries. In particular, the
> boundaries are a superset of [universal newlines].
>
> | Representation | Description                 |
> | -------------- | --------------------------- |
> | `\n`           | Line Feed                   |
> | `\r`           | Carriage Return             |
> | `\r\n`         | Carriage Return + Line Feed |
> | `\v` or `\x0b` | Line Tabulation             |
> | `\f` or `\x0c` | Form Feed                   |
> | `\x1c`         | File Separator              |
> | `\x1d`         | Group Separator             |
> | `\x1e`         | Record Separator            |
> | `\x85`         | Next Line (C1 Control Code) |
> | `\u2028`       | Line Separator              |
> | `\u2029`       | Paragraph Separator         |

This results in some surprising (to me) splitting behavior:

```python
>>> s = "line1\nline2\rline3\r\nline4\vline5\x1dhello"
>>> s.splitlines()
['line1', 'line2', 'line3', 'line4', 'line5', 'hello']
```

Whereas I would have expected:

```python
["line1", "line2", "line3", "line4\vline5\x1dhello"]
```

This was a good periodic reminder that Unicode does **not** mean
"printable," and that there are still plenty of ecosystems that
assign semantics to [C0 and C1 control codes].

[`str.splitlines()`]: https://docs.python.org/3/library/stdtypes.html#str.splitlines
[universal newlines]: https://docs.python.org/3/glossary.html#term-universal-newlines
[C0 and C1 control codes]: https://en.wikipedia.org/wiki/C0_and_C1_control_codes
[Seth Larson]: https://sethmlarson.dev

---
title: "Some surprising numeric conversions in Python"
tags: [python]
date: 2026-06-29
---

While researching for a full-length blog post, I came across this behavior:

```python
>>> int("٢")
2
>>> int("١٢")
12
```

(٢ and ١٢ are the Eastern Arabic numerals for 2 and 12, respectively.)

This also works for the Persian numerals, or Sundanese numerals:

```python
# Persian
>>> int("۴۴")
44

# Sundanese
>>> int("᮲᮲")
22
```

But _not_ Chinese/Japanese or (Sino-)Korean numerals:

```python
# Chinese/Japanese
>>> int("二")
ValueError: invalid literal for int() with base 10: '二'

# Sino-Korean
>>> int("이")
ValueError: invalid literal for int() with base 10: '이'
```

This was surprising to me as someone who doesn't read these languages
(and who doesn't think much about Unicode), but there _is_ a
consistency to it: Arabic, Persian, Sundanese (and Thai, etc.)
numerals are all `Numeric_Type=Decimal` in Unicode, whereas Chinese/Japanese
numerals are `Numeric_Type=Numeric`.

Korean "numerals" are even further removed, and don't have a numeric
category at all; they're in `General_Category=Lo` (Letter, Other), with
no numeric type[^1].

As a table:

| Language | Example | Unicode category | `int()` conversion | `isdecimal()` | `isdigit()` | `isnumeric()` |
|----------|---------|-----------------|------------------|---------------|-------------|----------------|
| Western Arabic | 2 | De | ✅ | ✅ | ✅ | ✅ |
| Eastern Arabic | ٢ | De | ✅ | ✅ | ✅ | ✅ |
| Persian | ۴ | De | ✅ | ✅ | ✅ | ✅ |
| Sundanese | ᮲ | De | ✅ | ✅ | ✅ | ✅ |
| Chinese/Japanese | 二 | Lo | ❌ | ❌ | ❌ | ✅ |
| Sino-Korean | 이 | Lo | ❌ | ❌ | ❌ | ❌ |

Python, as usual, _does_ document this, you just have to not glaze over it:

> A base-n integer string contains digits, each representing a value from 0 to n-1.
> The values 0–9 can be represented by any **Unicode decimal digit**.

([Source], emphasis mine).

[Source]: https://docs.python.org/3/library/functions.html#int

Why do I care about this? Because a lot of parsers do `int(x)` for some `x`,
assuming that `int(...)` only accepts Western Arabic numerals (and therefore,
that there's a direct correspondence between the byte length of the input
and its parsed value). This is not true, and can lead to surprising
parser misalignments/differentials. More in a forthcoming
full-length blog post.

[^1]: To my understanding, this is because Korean numerals (of both variants) are a counting scheme
      that uses letters as numbers, similar to how Hebrew uses aleph, bet, &amp;c.

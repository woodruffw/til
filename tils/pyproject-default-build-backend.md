---
title: pyproject.toml supports a default backend
tags: [python]
date: 2024-08-07
origin: "https://github.com/di/pip-api/pull/156#discussion_r1707285273"
---

[`pyproject.toml`] is the modern way to define Python project (i.e. library
or application) metadata.

At the core of `pyproject.toml` is the concept of a "build backend," which
is the toolchain responsible for interpreting the contents of a
`pyproject.toml` and performing ordinary development operations with them
(such as building a locally editable install, or a [source] or [wheel]
distribution).

I've always thought that the build backend needed to be specified explicitly,
e.g. to use [`flit`]:

```toml
[build-system]
requires = ["flit_core >=3.2,<4"]
build-backend = "flit_core.buildapi"
```

or [`setuptools`]:

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
```

However, this isn't actually required! You can *completely omit*
the `[build-system]` table from your `pyproject.toml`, and your build tool
(e.g. [pypa/build]) **should** default to using the `setuptools` backend.

This is really nice in the 99% case (pure Python packaging), since there's
effectively no difference between build backends when only handling pure Python
*and* it leaves one less version range/dependency to manage.

pypa/build's `setuptools` default can be seen [here (permalink)]:

```python
_DEFAULT_BACKEND = {
    'build-backend': 'setuptools.build_meta:__legacy__',
    'requires': ['setuptools >= 40.8.0'],
}
```

[`pyproject.toml`]: https://packaging.python.org/en/latest/specifications/pyproject-toml/

[source]: https://packaging.python.org/en/latest/specifications/source-distribution-format/

[wheel]: https://packaging.python.org/en/latest/specifications/binary-distribution-format/#binary-distribution-format

[`flit`]: https://flit.pypa.io/en/stable/

[`setuptools`]: https://setuptools.pypa.io/en/latest/

[pypa/build]: https://github.com/pypa/build

[here (permalink)]: https://github.com/pypa/build/blob/562907e605c3becb135ac52b6eb2aa939e84bdda/src/build/_builder.py#L33-L36

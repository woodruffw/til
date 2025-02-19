---
title: Python package metadata has at least 4 different representations
date: 2025-02-18
tags: [python]
---

Python packages come with _metadata_ that indices (like [PyPI])
and installers (like [pip] and [uv]) use to present and resolve
the package.

This metadata is standardized in the [Core metadata specifications],
which describes both the names and semantics of fields ("`Version` contains a
[valid version]") _and_ how they're parsed (as email-style headers, roughly).

What the standard _doesn't_ say is that there are really
(at least) _four_ different metadata representations, each of which appears in
a different context:

1. The standard or "email" representation, which is what appears
   in `PKG-INFO` for source distributions and `METADATA` for wheels.
   For example, [here's the METADATA for sampleproject==4.0.0].

2. The `pyproject.toml` representation, which what users actually
   type out (and what build backends consume). This is defined in a
   [separate PyPA standard].

   This representation is "rich" in the sense that it takes advantages
   of TOML's types, whereas the "email" representation imposes various
   conventions to express lists and mappings. For example,
   `Project-URL` is a multi-use field with a `label, url` format in the
    "email" representation, [but a mapping in the pyproject.toml]
    representation (as `[project.urls]`).

3. The "internal" representation, as expressed by the
   [`RawMetadata` API](https://github.com/pypa/packaging/blob/9b4922dd3c26c8522d716bec79d7e0ed408631c1/src/packaging/metadata.py#L64).
   This representation gets used throughout the packaging ecosystem,
   including within PyPI and `twine`.

   It closely resembles the
   `pyproject.toml` representation in that it uses types instead
   of multi-use fields, but with underscores instead of hyphens.
   For example, `Project-URL` becomes `project_urls: dict[str, str]`.

4. The "form" representation, which is what PyPI sees and interprets
   when users upload packages. This representation occurs in
   the form body of the `HTTP POST` sent to PyPI, and is
   encoded as `multipart/form-data`.

   This representation is a bit of a hybrid between the "email"
   and "internal" representations: fields go back to being multi-use
   or otherwise formatted to represent types (like the "email" representation)
   _while also_ using underscores instead of hyphens (like the "internal"
   representation).

   However, the names aren't completely the same: most multi-use fields
   are pluralized (e.g. `Project-URL` becomes `project_urls`), but
   three remain singular: `platform` (for `Platform`), `supported_platform`
   (for `Supported-Platform`), and `license_file` (for `License-File`).


To summarize, here's a table of the different representations:

| Representation | Example field | Example value |
|----------------|---------------|---------------|
| Standard       | `License-File` | `LICENSE.txt` |
| `pyproject.toml` | `license-files` | `["LICENSE.txt"]` |
| Internal       | `license_files` | `["LICENSE.txt"]` |
| Form           | `license_file` | `LICENSE.txt` |

There are of course even more representations, like the Python object
representation in a `setup.py` file. But these four represent what occurs
in current "idiomatic" Python packaging workflows.

[PyPI]: https://pypi.org/

[pip]: https://pip.pypa.io/

[uv]: https://github.com/astral-sh/uv

[Core metadata specifications]: https://packaging.python.org/en/latest/specifications/core-metadata/

[valid version]: https://packaging.python.org/en/latest/specifications/version-specifiers/

[here's the METADATA for sampleproject==4.0.0]: https://inspector.pypi.io/project/sampleproject/4.0.0/packages/d7/73/c16e5f3f0d37c60947e70865c255a58dc408780a6474de0523afd0ec553a/sampleproject-4.0.0-py3-none-any.whl/sampleproject-4.0.0.dist-info/METADATA

[separate PyPA standard]: https://packaging.python.org/en/latest/specifications/pyproject-toml/

[but a mapping in the pyproject.toml]: https://github.com/pypa/sampleproject/blob/621e4974ca25ce531773def586ba3ed8e736b3fc/pyproject.toml#L144-L149


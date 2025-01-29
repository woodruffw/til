---
title: tox supports labels as aliases for environments
date: 2025-01-28
tags: [tox, python]
origin: "https://github.com/di/pip-api/pull/240#discussion_r1932568984"
---

[`tox`](https://tox.wiki/) is a great automation tool for Python projects.
I don't use it a ton on my own projects (probably to my detriment), but
contributed to plenty of projects that do use it.

At the center of `tox` is its ability to define and manage multiple
virtual environments, including across multiple Python versions.

[`pip-api`](https://github.com/di/pip-api) takes this to its logical
extreme, generating a matrix of hundreds of environments to test
all supported permutations of (`$PYTHON_VERSION`, `$PIP_VERSION`).

However, sometimes you just need to test a single environment,
one that you want to label *conceptually*, e.g. "latest"
for the latest combination of Python and `pip`.

Fortunately, `tox` supports this with `labels` in `tox.ini`:

```ini
[tox]
envlist = # your environment expansions here
labels =
    latest = py313-pip250
```

([`tox`'s TOML config also works](https://tox.wiki/en/4.24.1/config.html#labels)).

From here, the `latest` label can be used with `tox -m` (unlike environments,
which are used with `tox -e`):

```bash
tox -m latest
```

Multiple environments can also be assigned to a single label, in which case
`tox -m somelabel` will run all of them.

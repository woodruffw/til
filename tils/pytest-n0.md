---
title: "Disabling pytest-xdist without removing it"
tags: [python]
date: 2024-07-12
origin: https://github.com/pypi/warehouse/pull/16206
---

I regularly use [pytest] both for my personal and professional projects.

One of the easiest ways to speed up a large pytest-based test suite is
to use the [pytest-xdist] plugin, which distributes your tests across
multiple cores (configurable with `-n NUM`).

This works great by default, *except* when you just want to run a single
test (or subset of tests) *and* your test initialization is time intensive:
this can result in `pytest-xdist` taking longer to collect and configure
its workers than the actual test case(s) would take when run serially!

I've historically just lived with this or set `-n 1` to reduce
the number of workers down to just 1 (which sometimes speeds things
up slightly), but there's a better way: `-n 0` disables worker distribution
entirely, meaning that `pytest` runs as if `pytest-xdist` isn't even installed.

[From the docs], it looks like `--dist no` might also work just as well.

[pytest]: https://docs.pytest.org/en/stable/

[pytest-xdist]: https://pytest-xdist.readthedocs.io/en/stable/

[From the docs]: https://pytest-xdist.readthedocs.io/en/stable/distribution.html#running-tests-across-multiple-cpus

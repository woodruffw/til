---
title: Overriding pytest fixtures
tags: [python, testing]
date: 2025-07-12
origin: personal correspondence
---

[pytest] is my favorite way to write tests in Python.

It supports "fixtures," which get dependency-injected into test functions
via matching parameters. For example:

```python
import pytest

@pytest.fixture
def db():
    # pretend there's some database setup here
    return object()

def test_something(db):
    # do some db operations here
```

Fixtures can also depend on each other:

```python
@pytest.fixture
def environment():
    return "development"

@pytest.fixture
def db(environment):
    # pretend there's some database setup here
    return object()

def test_something(db):
    # do some db operations here
```

This all works very nicely when `environment` is invariant: we can
stuff the `environment` and `db` fixtures into a `conftest.py` file
and reuse them throughout the test suite.

However, we run into a problem if we want the `environment` fixture to be
something new. For example, we might want a production fixture like this:

```python
@pytest.fixture
def production_environment():
    return "production"
```

However, that fixture is `production_environment`, not `environment`, so
`db` won't use it even when we want it to. The naive fix would be to add a
corresponding `production_db` fixture that depends on `production_environment`,
but this results in a lot of duplicated code.

To fix this, we can rely on the fact that `pytest` always loads the _most local_
matching fixture. In other words, if we define a fixture with the same name
as an existing one _but in a more local scope_, it will override the existing one.

For example, imagine the following test layout:

```
tests/
    conftest.py        <-- contains the `environment` and `db` fixtures
    test_production.py
```

Then, in `test_production.py`, we can override the `environment` fixture
like this:

```python
import pytest

@pytest.fixture
def environment():
    return "production"

def test_something(db):
    # uses db with the 'production' environment, even though 'db'
    # is defined in conftest.py
    pass
```

This example is of course simple and contrived, but the general form
is extremely useful in test suites with "generic" fixtures that need
to be tweaked slightly in different testing contexts.

[pytest]: https://docs.pytest.org/en/stable/

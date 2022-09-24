[![tests](https://github.com/robert-lieck/integration/actions/workflows/tests.yml/badge.svg)](https://github.com/robert-lieck/integration/actions/workflows/tests.yml) [![test_dev](https://github.com/robert-lieck/integration/actions/workflows/test_dev.yml/badge.svg)](https://github.com/robert-lieck/integration/actions/workflows/test_dev.yml)

# Integration

Integration tests for various other libraries

## Running the tests

In the repo root directory call (you will have to have the `unittest` or `pytest` module installed)

```
python -m unittest
```

or

```
pytest tests
```

This will discover and run all tests, including those from submodules added via symlinks (see below).

## Adding a new library

To add a new library for testing you have to take the following steps:

- Add the corresponding repo as a **new submodule** inside the `submodules` folder. See [here](https://gist.github.com/gitaarik/8735255) for some helpful tips on handling git submodules.

```
git submodule add git@github.com:url_to/new_submodule.git submodules/new_submodule_name
```

- Add a **relative symlink** inside the `tests` folder to point to the submodule. That is, from within the repo root directory do (the first part makes sure the link points *locally* to the submodule from within the integration test folder):

```
ln -sv ../submodules/new_submodule_name ./tests/new_submodule_name
```

- Make sure the tests use the **local version** of your submodule by adding the following inside `tests/__init__.py` (this makes sure your local version is first in `path` and therefore found before any preinstalled versions):

```python
import sys  # this is already there
sys.path.insert(0, "submodules/new_submodule_name")  # <-- add this
```

## Fixing tests in submodules locally

Fixing tests in the submodules can be tricky to do locally. This is because changes would need to be committed and updated in the main repo.  A quick fix to get a fast local development cycle is the following:

- rename the submodule folder, e.g., by appending an underscore
- create a symlink with the same *name* as the submodule folder (before renaming) that points to a *local version* of that submodule
- do your testing and development and revert (delete symlink, rename submodule) once you are done

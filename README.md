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

To add a new library `newlibrary` for testing you have to take the following steps.

### Add submodule

Add the corresponding repo as a **new submodule** inside the `submodules` folder. See [here](https://gist.github.com/gitaarik/8735255) for some helpful tips on handling git submodules.

```
git submodule add git@github.com:url_to/newlibrary.git submodules/newlibrary
```

### Symlink tests

Create symlinks to the relevant tests as follows:

- inside the `tests` folder create a subfolder `newlibrary`

- inside this folder, create a **relative symlink** named `newlibrary_tests` that points to the library's test folder. For instance, from the repo root directory call:
  ```
  ln -sv ../../submodules/newlibrary/tests ./tests/newlibrary/newlibrary_tests
  ```
  **Note:** The first part makes sure the link points to the test folder relative to its location (i.e. from inside `./tests/newlibrary/`). In the second part, it is not really relevant what the link is named, as long as it is a unique name (using the library's name just seems the obvious way to do that). Otherwise, there will be an error during test collection (tests are imported as python modules, which are named as the folder; this produces name clashes if the folder/modules have generic names such as `tests`, which they typically have in a standard python module).

- **Optional:** If the tests make use of any other files from `newlibrary` also add symlinks as needed. In particular, if there is an `examples` folder (containing examples to be included in the Sphinx documentation) and the tests also go through these examples, this folder has to be symlinked as well:
  
  ```
  ln -sv ../../submodules/newlibrary/examples ./tests/newlibrary/examples
  ```
  This symlink does *not* have to have a unique name; on the contrary, if the tests use a fixed relative link to the examples, it *has* to be named exactly as the folder it is pointing to.

### Use local version

- Make sure the tests use the **local version** of your library by adding the following inside `tests/__init__.py` (this makes sure your local version is first in `path` and therefore found before any preinstalled versions):
  ```python
  import sys  # this should be already there
  sys.path.insert(0, "submodules/newlibrary")  # <-- add this
  ```

## Multiple copies of submodules

If you use integration tests with a library you are also developing yourself, chances are that you have a main working copy of that library somewhere else. After including them as submodules in the integration tests, you now have two or more copies (at least a main working copy and the one in the submodule), which can cause confusion (e.g. changes in the main working copy are not reflected in the integration test until pushed upstream and pulled into the submodule).

An easy workaround is to create a `submodules_` folder (note the trailing underscore) that replicates the structure of the `submodules` folder but instead of containing submodules, it contains symlinks to the main working copies of the respective libraries. You can now easily switch to using your working copies (instead of the version in the submodule) by temporarily renaming the folders (`submodules --> _submodules` and `submodules_ --> submodules`). This allows you to do development, testing, debugging etc. as usual before committing the changes and pulling them into the submodules.

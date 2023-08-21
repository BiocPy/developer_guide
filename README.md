# Developer Guide

So you want to contribute or add a new package to [BiocPy](https://github.com/biocpy)? We'll use this repository to document our development setup, to maintain consistency across projects.

Following the zen of Python, we should be consistent within the package wrt documentation and code style, and consistent across packages for testing and publishing.

- Follow Google style guide - https://google.github.io/styleguide/pyguide.html
- For naming conventions, checkout https://google.github.io/styleguide/pyguide.html#3164-guidelines-derived-from-guidos-recommendations

## Setup

We use [pyscaffold](https://pyscaffold.org/en/stable/) to boostrap new packages. Optionally you can install the [markdown](https://github.com/pyscaffold/pyscaffoldext-markdown) and [pre-commit](https://pre-commit.com/) extensions.

```bash
pip install -U pyscaffold pyscaffoldext-markdown pre-commit
```

To scaffold a new package, it's as simple as,

```bash
putup <NEW_PACKAGE_NAME> --markdown --pre-commit
```

You should now have the basic package setup for you.

pyscaffold uses `tox`, to create isolated environments for tests, documentation and to publish the packages. This might be a good time to familiarize yourself with available [tox commands](https://pyscaffold.org/en/stable/features.html#pre-commit-hooks). **_you rarely would want to add or modify the default tox.ini file._**

### Update linter, styler configuration

I use line length of 100, but thats a personal choice if you want to stick to the default 88.

#### ruff

I highly recommend using [ruff](https://beta.ruff.rs/docs/).

Add this to the end of the `pyproject.toml` file.

```toml
[tool.ruff]
line-length = 100
src = ["src"]

[tool.ruff.pydocstyle]
convention = "google"

[tool.ruff.per-file-ignores]
"__init__.py" = ["E402", "F401"]
```

#### flake8

If you are using both flake and ruff, make these changes in the setup.cfg: Update `max_line_length` & add `per_file_ignores`.

```toml
[flake8]
# Some sane defaults for the code style checker flake8
max_line_length = 100
extend_ignore = E203, W503
# ^  Black-compatible
#    E203 and W503 have edge cases handled by black
exclude =
    .tox
    build
    dist
    .eggs
    docs/conf.py
per-file-ignores = __init__.py:F401
```

#### isort and black

Add this to the end of the `pyproject.toml` file.

```toml
[tool.isort]
profile = "black"
known_first_party = "biocframe"
skip = ["__init__.py"]

[tool.black]
force-exclude = "__init__.py"
```

#### pre-commit

If you are using pre-commits, you might need to run an extra step document [here](https://pyscaffold.org/en/stable/features.html#pre-commit-hooks). Checkout my [pre-commit config](./pre-commit-template.yml)

## Sphinx

I use the [furo](https://github.com/pradyunsg/furo) theme across all my packages.

Add furo to both `docs/requirements.txt` and update html theme to use furo (in `docs/conf.py`).

### Additional changes

- Use intersphinx to link to objects from other packages. This should work out of the box, e.g. to link to a pandas DataFrame object, **:py:class:\`pandas.DataFrame\`**. make sure the proper libraries are added to `intersphinx_mapping` in `docs/conf.py`.
- If classes implement dunder methods, these are not automatically documented. These are sensible defaults but modify accordingly in your `docs/conf.py` .

  ```py
  autodoc_default_options = {
      'special-members': True,
      'undoc-members': False,
      'exclude-members': '__weakref__, __dict__, __str__, __module__, __init__'
  }

  autosummary_generate = True
  autosummary_imported_members = True
  ```

## Publish package to PyPI

For most packages, the [github workflows](./workflows/) included in this repo should suffice. Feel free to copy them to the .github directory of your package. you might have to setup twine's username and token to publish the package to PyPI.

If you are developing packages that interface with C/C++ libraries and need to build many wheels, checkout the github workflows in [scranpy](htps://github.com/biocpy/scranpy). We use [cibuildwheel](https://cibuildwheel.readthedocs.io/en/stable/), but always looking to improve that workflow.

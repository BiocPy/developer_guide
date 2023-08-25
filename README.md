# Developer Guide: Contributing and Adding Packages to BiocPy

Welcome to the Developer Guide for contributing to BiocPy and/or adding new packages. This repository provides guidance on the developer tools we employ to ensure code quality and consistency across all our projects.

**Maintaining Consistency:**
When working within a package, it's essential to maintain consistency in both documentation and code style. This document outlines key practices.

**Embracing Google's Python Style Guide:**
We highly recommend adhering to Google's Python style guide for consistency. You can find detailed [naming conventions](https://google.github.io/styleguide/pyguide.html#3164-guidelines-derived-from-guidos-recommendations).

## Getting Started
We use [pyscaffold](pyscaffold.org/) to streamline the process of creating a new package. Optionally, you can enhance your workflow by installing the ***markdown*** and ***pre-commit*** extensions:

```bash
pip install -U pyscaffold pyscaffoldext-markdown pre-commit
```

Creating a new package is as simple as running the following command:

```bash
putup <NEW_PACKAGE_NAME> --markdown --pre-commit
```

This command sets up the basic package structure.

## Code Quality

### Isolated Environments and Testing
pyscaffold employs tox to create isolated environments for testing, documentation, and package publication. Familiarize yourself with [available tox commands](https://pyscaffold.org/en/stable/features.html). You'll rarely need to modify the default `tox.ini` file.

### Updating Linter and Styler Configuration
We recommend a line length of 120, but feel free to modify to the default 88.

We highly recommend using [ruff](https://beta.ruff.rs/docs/) for linting. Add the following to the end of your `pyproject.toml` file:

```toml
[tool.ruff]
line-length = 120
src = ["src"]
exclude = ["tests"]
extend-ignore = ["F821"]

[tool.ruff.pydocstyle]
convention = "google"

[tool.ruff.per-file-ignores]
"__init__.py" = ["E402", "F401"]

[tool.black]
force-exclude = "__init__.py"
```

### Pre-commit Setup
If you're utilizing [pre-commits](https://pre-commit.com/), an additional step might be needed. Check out our [pre-commit configuration](./pre-commit-template.yml). We have enabled [Pre-commit bot](https://pre-commit.ci/) across all BiocPy packages to ensure consistent code quality and documentation.

## Sphinx for Documentation
We use the [furo theme](https://github.com/pradyunsg/furo) across all packages for a unified look. Add furo to both `docs/requirements.txt` and update the HTML theme to use furo (in `docs/conf.py`).

### Additional Considerations
Utilize [intersphinx](https://www.sphinx-doc.org/en/master/usage/extensions/intersphinx.html) to link objects from other packages. For instance, to link to a pandas DataFrame object, use :py:class:\`~pandas.DataFrame\`. Make sure to update the intersphinx_mapping in `docs/conf.py`.

Note that dunder methods of classes aren't automatically documented. Modify these defaults as needed in your `docs/conf.py``:

```python
autodoc_default_options = {
    'special-members': True,
    'undoc-members': False,
    'exclude-members': '__weakref__, __dict__, __str__, __module__, __init__'
}

autosummary_generate = True
autosummary_imported_members = True
```

## Publishing on PyPI
For most packages, the provided [GitHub workflows](./workflows/) should suffice. You can copy them to your package's .github directory. You might need to set up Twine's username and token to publish packages to PyPI.

If you're developing packages interfacing with C/C++ libraries and require building multiple wheels, refer to the GitHub workflows in [scranpy](https://github.com/BiocPy/scranpy). While we use cibuildwheel, we're continuously improving this workflow.

# Have fun!
Your contributions and packages are valuable to BiocPy, and this guide helps to meet our standards. Thank you for being a part of our developer community and more importantly Have fun!
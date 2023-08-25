# Developer Guide: Contributing and Adding Packages to BiocPy

Welcome to the Developer Guide for contributing to BiocPy and/or adding new packages. This repository provides guidance on the developer tools we employ to ensure code quality and consistency within and across all the packages.

**Maintaining Consistency:**
When developing a package, it's essential to maintain consistency in documentation and code style. When working across packages, guidelines are designed to establish a reliable framework for testing and publishing packages, ensuring a seamless development process.

**Embracing Google's Python Style Guide:**
We highly recommend adhering to [Google's Python style guide](https://google.github.io/styleguide/pyguide.html) for consistency. You can find detailed information on [naming conventions](https://google.github.io/styleguide/pyguide.html#3164-guidelines-derived-from-guidos-recommendations), [type annotations](https://google.github.io/styleguide/pyguide.html#319-type-annotations) and more.

## Getting Started
We use [pyscaffold](pyscaffold.org/) to streamline the process of creating a new package. Optionally, you can enhance your workflow by installing the ***markdown*** (unless you really want to use [restructuredtext](https://docutils.sourceforge.io/rst.html)) and ***pre-commit*** extensions:

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
pyscaffold uses tox to create isolated environments for testing, documentation, and publishing packages. Familiarize yourself with [available tox commands](https://pyscaffold.org/en/stable/features.html). You'll rarely need to modify the default `tox.ini` file.

### Linter and Styler Configuration
We recommend a line length of 120 as it seems to work well in most scenarios, but feel free to modify this to your preference. We highly recommend using [ruff](https://beta.ruff.rs/docs/) for linting. 

To use this configuration, add the following to the end of  your `pyproject.toml` file:

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
An additional step might be needed if you are utilizing [pre-commits](https://pre-commit.com/). Our recommended [pre-commit configuration](./pre-commit-template.yml) is included in this repository. 

We also enabled [pre-commit bot](https://pre-commit.ci/) across all BiocPy packages to automate and auto-fix code and documentation.

## Sphinx for Documentation
We use the [furo theme](https://github.com/pradyunsg/furo) across all packages for a unified look. Add furo to both `docs/requirements.txt` and update the HTML theme to use furo (in `docs/conf.py`).

***Something on our todo list is to explore [quartodoc](https://github.com/machow/quartodoc) and/or [Jupyter Book](https://jupyterbook.org/en/stable/intro.html) for documentation and tutorials (vignette-like reproducible tutorials).***

### Additional Considerations
Utilize [intersphinx](https://www.sphinx-doc.org/en/master/usage/extensions/intersphinx.html) to link objects from other packages. For instance, to link to a pandas DataFrame object, one might simply specify **:py:class:\`~pandas.DataFrame\`** within their docstrings. While this is sometimes annoying, its helps developers and users recognizing and identifying the underlying data model or objects the documentation refers to.

Make sure to update the `"intersphinx_mapping"` to external packages in `docs/conf.py`.

Note that [dunder methods of classes](https://docs.python.org/3/reference/datamodel.html#) aren't automatically documented. Modify these defaults as needed in your `docs/conf.py`:

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
For most packages, the included [GitHub workflows](./workflows/) should suffice for most scenarios as long as you follow the instructions in this document. You might need to set up [twine](https://twine.readthedocs.io/en/stable/) to publish packages to PyPI.

If you are developing packages interfacing with C/C++ libraries and require building multiple wheels, refer to our GitHub workflows in [scranpy](https://github.com/BiocPy/scranpy). While we currently use cibuildwheel (and its very slow), we are trying to speed up this workflow.

# Have fun!
Your contributions and packages are valuable to BiocPy, and we hope this guide helps set guidelines and standards. Thank you for being a part of our developer community and more importantly have fun!
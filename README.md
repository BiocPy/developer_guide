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

### Sphinx for Documentation
We use the [furo theme](https://github.com/pradyunsg/furo) across all packages for a unified look. Add furo to both `docs/requirements.txt` and update the HTML theme to use furo (in `docs/conf.py`).

In addition, we use [sphinx-autodoc-typehints](https://github.com/tox-dev/sphinx-autodoc-typehints) for a cleaner api documentation. Include this package in `docs/requirements.txt` and add it as an extension in `docs/conf.py`

```python
extensions = [
    "sphinx.ext.autodoc",
    "sphinx.ext.intersphinx",
    "sphinx.ext.todo",
    "sphinx.ext.autosummary",
    "sphinx.ext.viewcode",
    "sphinx.ext.coverage",
    "sphinx.ext.doctest",
    "sphinx.ext.ifconfig",
    "sphinx.ext.mathjax",
    "sphinx.ext.napoleon",
    "sphinx_autodoc_typehints",
]
```

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

### Type hints

As the term suggests, these are "hints", only used to enhance the developer experience; they should not dictate how we write our code. For this reason, we prefer simple types in these hints, usually corresponding to base Python types with minimal nesting. For example, if a function is expected to operate on any arbitrary list, the basic list type hint should suffice. 

```python
def find_element(arr: list, query: int)
    pass
```

If your function expects a list of strings, 

```py
from typing import List

def find_element(arr: List[str], query: str):
    pass
```

If your function accepts multiple types as inputs, 

```py
from typing import Union

def find_element(arr: List[str], query: Union[int, str, slice]):
    pass
```

There is no need to waste time constructing the most perfectly descriptive type for your arguments or return values; just use a simple hint with minimal nesting and put the details in the docstring instead. 

## Class design

### General comments
For each Bioconductor class, we aim to provide the same user experience in Python.
In most cases, this is done by just directly re-implementing the class and its associated methods in Python.
Occasionally, the Bioconductor implementation has some historical baggage (e.g., the storage of `rowData` in a `RangedSummarizedExperiment`, `MultiAssayExperiment` harmonization);
developers should use their own discretion to decide whether that really needs to be replicated in Python.

### Use functional discipline
The existence of mutable types in Python means that it can be dangerous to modify complex objects.
If a mutable object has a user-visible reference and is also a member of a larger Bioconductor object,
a user-specified modification to that object may violate the constraints of the parent object.

To mitigate these issues, we enforce a functional programming discipline in all class methods.
By default, all methods should avoid side effects that mutate the object.
This simplifies reasoning around the effect of methods in large complex objects.

The most obvious application of this philosophy is in setter methods.
Rather than mutating the object directly, they should return a new copy of the object with the desired modification.
The "depth" of the copy depends on the nature of the field being set; the aim should be to avoid any modification of the contents in `self`.
Implementations may offer an `in_place=` option to apply the modification to the original object, but this should be `False` by default.

To avoid performance issues, getter methods may return mutable objects without copying 
This assumes that their return values are read-only and will not be directly mutated.
(Setter methods that operate via a copy are allowed.)
In some cases; the return value of a getter method may be directly mutated, e.g., because a copy was already created in the getter;
this should be clearly stated in the documentation but should not be treated as the default.

Direct access to class members (via properties or `@property`) should generally be avoided,
as it is too easy to perform modifications via one liners with the `class.property` on the left-hand-side of an assignment.

### Nomenclature
Classes should use `PascalCase` and should follow Bioconductor's class names.

Methods should use `snake_case` and should take the form of `<verb>[_<details>]`.
For example, `get_start()`, `set_names()` and so on.

Method arguments should also use `snake_case`.

## Publishing on PyPI
For most packages, the included [GitHub workflows](./workflows/) should suffice for most scenarios as long as you follow the instructions in this document. You might need to set up [twine](https://twine.readthedocs.io/en/stable/) to publish packages to PyPI.

If you are developing packages interfacing with C/C++ libraries and require building multiple wheels, refer to our GitHub workflows in [scranpy](https://github.com/BiocPy/scranpy). While we currently use cibuildwheel (and its very slow), we are trying to speed up this workflow.

# Have fun!
Your contributions and packages are valuable to BiocPy, and we hope this guide helps set guidelines and standards. Thank you for being a part of our developer community and more importantly have fun!

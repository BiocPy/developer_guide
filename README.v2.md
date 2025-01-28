# Developer Guide (V2): Contributing and Adding Packages to BiocPy

Welcome to the Developer Guide for contributing to BiocPy and/or adding new packages. This repository provides guidance on the developer tools we employ to ensure code quality and consistency within and across all the packages.

**Maintaining Consistency:**
When developing a package, it's essential to maintain consistency in documentation and code style. When working across packages, guidelines are designed to establish a reliable framework for testing and publishing packages, ensuring a seamless development process.

## Getting Started

We created [BiocSetup](https://github.com/BiocPy/BiocSetup) to automate the scaffolding of new Python packages with additional configurations specific to BiocPy projects, including documentation setup, GitHub Actions for testing and publishing, and code quality tools.

```sh
# Install biocsetup
pip install biocsetup

# Start a new package
biocsetup my-new-package --description "Description of my package" --license MIT
```

> **Note**: If you prefer to use `pyscaffold`: check out the V1 of this README [here](./README.v1.md).

## Class design

### Nomenclature

**Embracing Google's Python Style Guide:**
We highly recommend adhering to [Google's Python style guide](https://google.github.io/styleguide/pyguide.html) for consistency. You can find detailed information on [naming conventions](https://google.github.io/styleguide/pyguide.html#3164-guidelines-derived-from-guidos-recommendations), [type annotations](https://google.github.io/styleguide/pyguide.html#319-type-annotations) and more.

#### tldr

- Classes should use `PascalCase` and should follow Bioconductor's class names.
- Methods should use `snake_case` and should take the form of `<verb>[_<details>]`.
For example, `get_start()`, `set_names()` and so on.
    - Method arguments should also use `snake_case`.

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

#### Property based accessors and setters
Direct access to class members (via properties or `@property`) should generally be avoided,
as it is too easy to perform modifications via one liners with the `class.property` on the left-hand-side of an assignment.

***The default assumption is property based setters will mutate the object in-place.***

## Code Quality

A number of changes with environment isolation, linter, sphinx configuration and publishing to PyPI is included when new packages are created using [BiocSetup](https://github.com/BiocPy/BiocSetup). If you prefer to manually configure these settings, check out the V1 of this README [here](./README.v1.md).

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

There is no need to waste time constructing the most perfectly descriptive type for your arguments or return values; just use a simple hint with minimal nesting and put the details in the docstring instead. This [reddit post](https://www.reddit.com/r/Python/comments/10zdidm/why_type_hinting_sucks/) perfectly sums up some of these ideas.

# Have fun!

Your contributions and packages are valuable to BiocPy, and we hope this guide helps set guidelines and standards. Thank you for being a part of our developer community and more importantly have fun!

# Setuptools

Setuptools is a collection of enhancements to the Python `distutils` that allow developers to more easily build and distribute Python packages, especially ones that have dependencies on other packages.

<!-- markdownlint-disable MD024 -->
<!-- /* spell-checker:words distutils */ -->

## Table of Contents <!-- omit in toc -->

- [Installation](#installation)
- [Configurations](#configurations)
- [Creating and Configuring a Project Using pyproject.toml](#creating-and-configuring-a-project-using-pyprojecttoml)
  - [pyproject.toml](#pyprojecttoml)
  - [How the Project Was Initialized](#how-the-project-was-initialized)
  - [Build Package](#build-package)
  - [Managing Package Version](#managing-package-version)
  - [Managing Dependency Packages](#managing-dependency-packages)
  - [Provide Console Scripts](#provide-console-scripts)
- [Creating and Configuring a Project Using setup.cfg](#creating-and-configuring-a-project-using-setupcfg)
  - [Controlling Files in the Distribution](#controlling-files-in-the-distribution)
  - [How the Project Was Initialized](#how-the-project-was-initialized-1)
  - [Build Package](#build-package-1)
  - [Using a src Layout](#using-a-src-layout)
  - [Managing Package Version](#managing-package-version-1)
  - [Managing Dependency Packages](#managing-dependency-packages-1)
  - [Provide Console Scripts](#provide-console-scripts-1)
- [`setup.py` Commands](#setuppy-commands)
  - [Usage](#usage)
  - [`clean` Clean](#clean-clean)
- [References](#references)

## Installation

Ensure pip, setuptools, and wheel are up to date:

```shell
python -m pip install --upgrade pip
pip install --upgrade setuptools wheel build
```

## Configurations

Currently, the mainstream way to manage projects is with `pyproject.toml` as defined by PEP 518.

Previously, there were three patterns:

1. Legacy pattern: `setup.cfg` + `setup.py`
2. Compatibility pattern: `pyproject.toml` + `setup.cfg`
3. Future-oriented pattern: `pyproject.toml` only

Most projects now use `pyproject.toml`.

## Creating and Configuring a Project Using pyproject.toml

> Starting with PEP 621, the Python community selected `pyproject.toml` as a standard way of specifying project metadata.

### pyproject.toml

The `pyproject.toml` file acts as a configuration file for packaging-related tools.

> This specification was originally defined in PEP 518 and PEP 621.

- [Configuring setuptools using pyproject.toml files](https://setuptools.pypa.io/en/latest/userguide/pyproject_config.html)
- [pyproject.toml specification](https://packaging.python.org/en/latest/specifications/pyproject-toml/#pyproject-toml-spec)

**`[build-system]` table**:  
Declares Python-level dependencies required for the project's build system.

**`[project]` table**:  
Declares project metadata.

**`[tool]` table**:  
Configuration for Python-related tools.

### How the Project Was Initialized

Create and move to the project folder:

```shell
mkdir -p apps/examples-packaging-setup2
cd apps/examples-packaging-setup2
```

Create package:

```shell
mkdir -p apps/examples_packaging_setup2/
touch apps/examples_packaging_setup2/__init__.py
```

Create a `pyproject.toml` file for your project configuration:

```toml
[project]
name = "examples-packaging-setup2"
description = "Python examples of packaging using setuptools."
license = {text = "MIT"}
readme = "README.md"
requires-python = ">=3.11"
version = "0.2.0"
dependencies = []

[build-system]
build-backend = "setuptools.build_meta"
requires = ["setuptools"]

[tool.setuptools.package-dir]
"examples_packaging_setup2" = "apps/examples_packaging_setup2"

[tool.setuptools.packages.find]
exclude = ["tests*"]
where = ["src"]
```

Final directory structure:

```console
.
├── README.md
├── pyproject.toml
└── src
    └── examples_packaging_setup2
        └── __init__.py
```

### Build Package

```shell
python -m build
```

### Managing Package Version

- [Single-sourcing the package version](https://packaging.python.org/en/latest/guides/single-sourcing-package-version/)

Manage the version globally in the top-level `__init__.py`:

```python
__version__ = "0.2.1"
```

`pyproject.toml` (diff):

```diff
[project]
- version = 0.2.0
+ dynamic = ["version"]

+ [tool.setuptools.dynamic]
+ version = {attr = "examples_packaging_setup2.__version__"}
```

### Managing Dependency Packages

- [Dependencies Management in Setuptools](https://setuptools.pypa.io/en/latest/userguide/dependency_management.html)

List dependencies in `pyproject.toml`:

```toml
[project]
dependencies = [
  "requests",
  'importlib-metadata; python_version<"3.10"',
]
```

Declare optional dependencies:

```toml
dev = [
  "flake8>=7.0.0",
  "build",
  "mypy>=1.11.0",
  "black>=24.4.0",
  "isort>=5.13.0",
  "pyclean",
]
doc = [
  "sphinx"
]
```

Install with extras:

```shell
pip install -e .[dev,doc]
```

### Provide Console Scripts

How to provide console scripts in a setuptools package:

- [Entry Points](https://setuptools.pypa.io/en/latest/userguide/entry_point.html)

Create entry point:

`console/command.py`:

```python
def main() -> None:
    print("Hello setuptools with pyproject.toml.")
```

`pyproject.toml`:

```toml
[project.scripts]
examples-setup2-cli = "examples_packaging_setup2.console.command:main"
```

Install the package locally:

```shell
pip install -e .
```

Run:

```console
$ examples-setup2-cli

Hello setuptools with pyproject.toml.
```

## Creating and Configuring a Project Using setup.cfg

This is a legacy pattern.

### Controlling Files in the Distribution

**`setup.py`**:  
Configures your project and provides a CLI for packaging tasks.

> When a PEP 517 build is invoked, setuptools will emulate a dummy setup.py file.  
> Editable installs are not supported, so you'll eventually need a setup.py.

**`setup.cfg`**:  
INI file containing option defaults for setup.py commands.

**`README.md`**:  
All projects should contain a readme file.

**`MANIFEST.in`**:  
Needed to package additional files not automatically included in a source distribution.

**`LICENSE.txt`**:  
Every package should include a license file.

### How the Project Was Initialized

Create and move to the project folder:

```shell
mkdir -p apps/examples-packaging-setup
cd apps/examples-packaging-setup
```

Create package:

```shell
mkdir -p examples_packaging_setup/
touch examples_packaging_setup/__init__.py
```

Create project settings files:

`setup.cfg`:

```ini
[metadata]
name = examples-packaging-setup
version = 0.1.0
description = "Python examples of packaging using setuptools."
license = MIT

[options]
install_requires =
    requests
    importlib-metadata; python_version<"3.10"
```
<!-- /* spell-checker:words importlib */ -->

If you use `setup.cfg`, `setup.py` is very simple:

```python
from setuptools import setup  # type: ignore

setup()
```

Final directory structure:

```console
.
├── README.md
├── setup.cfg
├── setup.py
└── examples_packaging_setup
    └── __init__.py
```

### Build Package

```shell
python -m build
```

If you get the error `No module named build`:

```shell
pip install build
```

### Using a src Layout

- [Package Discovery and Namespace Packages](https://setuptools.pypa.io/en/latest/userguide/package_discovery.html#legacy-namespace-packages)

The src layout requires additional configuration:

```console
.
├── README.md
├── setup.cfg
├── setup.py
└── src
    └── examples_packaging_setup
        └── __init__.py
```

`setup.cfg`:

```ini
[options]
packages = find:
package_dir =
    =src

[options.packages.find]
where = src
exclude =
    tests*
```

### Managing Package Version

- [Single-sourcing the package version](https://packaging.python.org/en/latest/guides/single-sourcing-package-version/)

Manage the version globally in the top-level `__init__.py`:

```python
__version__ = "0.1.1"
```

`setup.cfg` (diff):

```diff
[metadata]
- version = 0.1.0
+ version = attr: examples_packaging_setup.__version__
```

### Managing Dependency Packages

- [Dependencies Management in Setuptools](https://setuptools.pypa.io/en/latest/userguide/dependency_management.html)

List dependencies in `install_requires`:

```ini
[options]
install_requires =
    requests
    importlib-metadata; python_version<"3.10"
```

Declare optional dependencies:

```ini
[options.extras_require]
dev =
    black>=24.4.0
    build>=1.2.0
    flake8>=7.0.0
    mypy>=1.11.0
    isort>=5.13.0
    pyclean
doc =
    sphinx
```

Install with extras:

```shell
pip install -e .[dev,doc]
```

### Provide Console Scripts

How to provide console scripts in a setuptools package:

- [Entry Points](https://setuptools.pypa.io/en/latest/userguide/entry_point.html)

Create entry point:

`console/command.py`:

```python
def main() -> None:
    print("Hello setuptools.")
```

`setup.cfg`:

```ini
[options.entry_points]
console_scripts =
    examples-setup-cli = examples_packaging_setup.console.command:main
```

Install the package locally:

```shell
pip install -e .
```

Run:

```console
$ examples-setup-cli

Hello setuptools.
```

## `setup.py` Commands

- [Running setuptools commands](https://setuptools.pypa.io/en/latest/deprecated/commands.html#running-setuptools-commands)

### Usage

```shell
python setup.py --help-commands
python setup.py --help clean
```

### `clean` Clean

```shell
python setup.py clean --all
```

## References

- [Setuptools User Guide](https://setuptools.pypa.io/en/latest/userguide/index.html#)
- [Distributing packages using setuptools](https://packaging.python.org/ja/latest/guides/distributing-packages-using-setuptools/)
- [Setuptools GitHub Repository](https://github.com/pypa/setuptools)

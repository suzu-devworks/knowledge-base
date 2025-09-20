# Setuptools

Setuptools is a collection of enhancements to the Python `distutils` that allow developers to more easily build and distribute Python packages, especially ones that have dependencies on other packages.

<!-- markdownlint-disable MD024 -->
<!-- /* spell-checker:words distutils */ -->

- [Setuptools](#setuptools)
  - [Installation](#installation)
  - [Configurations](#configurations)
  - [Creating and Configuring a project using a pyproject.toml file](#creating-and-configuring-a-project-using-a-pyprojecttoml-file)
    - [pyproject.toml](#pyprojecttoml)
    - [How the project was initialized](#how-the-project-was-initialized)
    - [Build package](#build-package)
    - [Managing package version](#managing-package-version)
    - [Managing dependency packages](#managing-dependency-packages)
    - [Provide console scripts](#provide-console-scripts)
  - [Creating and Configuring a Project Using the setup.cfg File](#creating-and-configuring-a-project-using-the-setupcfg-file)
    - [Controlling files in the distribution](#controlling-files-in-the-distribution)
    - [How the project was initialized](#how-the-project-was-initialized-1)
    - [Build package](#build-package-1)
    - [Using a src layout](#using-a-src-layout)
    - [Managing package version](#managing-package-version-1)
    - [Managing dependency packages](#managing-dependency-packages-1)
    - [Provide console scripts](#provide-console-scripts-1)
  - [`setup.py` commands](#setuppy-commands)
    - [Usage](#usage)
    - [`clean` Clean](#clean-clean)
  - [References](#references)

## Installation

Ensure pip, setuptools, and wheel are up to date.

```shell
python -m pip install --upgrade pip
pip install --upgrade setuptools wheel build
```

## Configurations

Currently, the mainstream way to manage projects is with pyproject.toml. as defined by PEP 518.

When I looked previously, there were three examples with different patterns, but now most of them seem to have been changed to `pyproject.toml`.

1. legacy pattern of `setup.cfg` + `setup.py`
2. `pyproject.toml` + `setup.cfg`(main) compatibility pattern
3. Future-oriented patterns in `pyproject.toml`

## Creating and Configuring a project using a pyproject.toml file

> Starting with PEP 621, the Python community selected pyproject.toml as a standard way of specifying project metadata.

### pyproject.toml

The pyproject.toml file acts as a configuration file for packaging-related tools.

> This specification was originally defined in PEP 518 and PEP 621.

- [Configuring setuptools using pyproject.toml files](https://setuptools.pypa.io/en/latest/userguide/pyproject_config.html)
- [pyproject.toml specification](https://packaging.python.org/en/latest/specifications/pyproject-toml/#pyproject-toml-spec)

**`[build-system]` table**:

Declares Python-level dependencies that must be installed for the project's build system to run successfully.

**`[build-system]` table**:

Declaring project metadata.

**`[tool]` table**:

All tools related to Python projects, not just build tools, can be given configuration data by the user.

### How the project was initialized

Create and move project folder:

```shell
mkdir -p apps/examples-packaging-setup2
cd apps/examples-packaging-setup2
```

Create package:

```shell
mkdir -p apps/examples_packaging_setup2/
touch apps/examples_packaging_setup2/__init__.py
```

Create a `pyproject.toml` file for your project configuration.
There is no command after all.

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

The final directory will look like this:

```console
.
├── README.md
├── pyproject.toml
└── src
    └── examples_packaging_setup2
        └── __init__.py
```

### Build package

```shell
python -m build
```

### Managing package version

- [Single-sourcing the package version](https://packaging.python.org/en/latest/guides/single-sourcing-package-version/)

The version is managed globally in the top-level `__init__.py`.

```py
__version__ = "0.2.1"
```

`pyproject.toml`(diff):

```diff
[project]
- version = 0.2.0
+ dynamic = ["version"]

+ [tool.setuptools.dynamic]
+ version = {attr = "examples_packaging_setup2.__version__"}
```

### Managing dependency packages

- [Dependencies Management in Setuptools](https://setuptools.pypa.io/en/latest/userguide/dependency_management.html)

There are no commands, they are handwritten.

Dependencies will be installed together when you list them in `pyproject.toml`:

```toml
[project]
dependencies = [
  "requests",
  'importlib-metadata; python_version<"3.10"',
]
```

Allows you to declare dependencies that are not installed by default in `pyproject.toml`:

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
  "sphinx
]
```

Specify the key when installing:

```shell
pip install -e .[dev,doc]
```

### Provide console scripts

How to provide console scripts in the setuptools package.

- [Entry Points](https://setuptools.pypa.io/en/latest/userguide/entry_point.html)

Create entry point

`console/command.py`:

```py
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

When you run it:

```console
$ examples-setup2-cli

Hello setuptools with pyproject.toml.
```

## Creating and Configuring a Project Using the setup.cfg File

This is a legacy pattern.

### Controlling files in the distribution

**`setup.py`**:

Serves two primary functions:

1. your project are configured
2. the command line interface for running various commands that relate to packaging tasks.

> When a PEP 517 build is invoked, setuptools will emulate a dummy setup.py file.  
> However, editable installs are not supported, so you'll eventually need a setup.py.

**`setup.cfg`**:

Ini file containing option defaults for the setup.py command.

**`README.md`**:

All projects should contain a readme file.

**`MANIFEST.in`**:

needed when you need to package additional files that are not automatically included in a source distribution.

**`LICENSE.txt`**:

Every package should include a license file detailing the terms of distribution.

### How the project was initialized

Create and move project folder:

```shell
mkdir -p apps/examples-packaging-setup
cd apps/examples-packaging-setup
```

Create package:

```shell
mkdir -p examples_packaging_setup/
touch examples_packaging_setup/__init__.py
```

Create two files for project settings.
There are no commands.

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

If you use `setup.cfg`, `setup.py` is very simple like this:

```py
from setuptools import setup # type: ignore

setup()
```

The final directory will look like this:

```console
.
├── README.md
├── setup.cfg
├── setup.py
└── examples_packaging_setup
    └── __init__.py
```

### Build package

```shell
python -m build
```

If you get the error `No module named` build:

```shell
pip install build
```

### Using a src layout

- [Package Discovery and Namespace Packages](https://setuptools.pypa.io/en/latest/userguide/package_discovery.html#legacy-namespace-packages)

This type of layout, called the src layout, requires additional configuration.

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

```diff
[options]
packages = find:
package_dir =
    =src

[options.packages.find]
where = src
exclude =
    tests*
```

### Managing package version

- [Single-sourcing the package version](https://packaging.python.org/en/latest/guides/single-sourcing-package-version/)

The version is managed globally in the top-level `__init__.py`.

```py
__version__ = "0.1.1"
```

`setup.cfg`(diff):

```diff
[metadata]
- version = 0.1.0
+ version = attr: examples_packaging_setup.__version__
```

### Managing dependency packages

- [Dependencies Management in Setuptools](https://setuptools.pypa.io/en/latest/userguide/dependency_management.html)

There are no commands, they are handwritten.

Dependencies will be installed together when you list them in `install_requires`:

```ini
[options]
install_requires =
    requests
    importlib-metadata; python_version<"3.10"
```

Allows you to declare dependencies that are not installed by default:

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

Specify the key when installing:

```shell
pip install -e .[dev,doc]
```

### Provide console scripts

How to provide console scripts in the setuptools package.

- [Entry Points](https://setuptools.pypa.io/en/latest/userguide/entry_point.html)

Create entry point

`console/command.py`:

```py
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

When you run it:

```console
$ examples-setup-cli

Hello setuptools.
```

## `setup.py` commands

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

- <https://setuptools.pypa.io/en/latest/userguide/index.html#>
- <https://packaging.python.org/ja/latest/guides/distributing-packages-using-setuptools/>
- <https://github.com/pypa/setuptools>

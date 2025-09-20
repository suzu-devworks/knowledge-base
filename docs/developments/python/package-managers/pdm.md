# PDM

PDM is a modern Python package and dependency manager supporting the latest PEP standards. It is more than just a package manager.

[![pdm-managed](https://img.shields.io/endpoint?url=https%3A%2F%2Fcdn.jsdelivr.net%2Fgh%2Fpdm-project%2F.github%2Fbadge.json)](https://pdm-project.org)

`pyproject.toml` supports PEP 621.

> [!WARNING]
> **PEP 582 has been rejected**
>
> This feature was the reason for PDM's creation, so PDM will retain support for it. However, using virtual environments is recommended.

## Table of Contents <!-- omit in toc -->

- [Installation](#installation)
  - [Using the Standalone Installer](#using-the-standalone-installer)
  - [Self update](#self-update)
- [Configurations](#configurations)
  - [Shell Completion](#shell-completion)
- [Creating and Configuring a Project](#creating-and-configuring-a-project)
  - [How the Project Was Initialized](#how-the-project-was-initialized)
  - [Install Dependency Packages](#install-dependency-packages)
  - [The src Layout](#the-src-layout)
  - [Use a Task Runner Like npm](#use-a-task-runner-like-npm)
  - [Dynamic Versioning](#dynamic-versioning)
  - [Provide Console Scripts](#provide-console-scripts)
- [Package Management](#package-management)
  - [`init` Project Setup](#init-project-setup)
  - [`venv` Use Virtualenv](#venv-use-virtualenv)
  - [`sync/update/install` Restore Packages](#syncupdateinstall-restore-packages)
  - [`add` Add Package](#add-add-package)
  - [`update` Update Package](#update-update-package)
  - [`update` Update Outdated Packages](#update-update-outdated-packages)
  - [`remove` Remove Package](#remove-remove-package)
  - [`list` List Packages](#list-list-packages)
  - [`build` Build Package](#build-build-package)
- [References](#references)

## Installation

### Using the Standalone Installer

Recommended installation method:

```shell
curl -sSL https://pdm-project.org/install-pdm.py | python3 -
```

Or using pip in a virtual environment:

```shell
pip install pdm
```

### Self update

Update PDM itself:

```shell
pdm self update
```

## Configurations

### Shell Completion

PDM supports generating completion scripts for Bash, Zsh, Fish, or PowerShell.

For Bash:

```shell
pdm completion bash > pdm.bash-completion
sudo mv pdm.bash-completion /etc/bash_completion.d/
```

See [Shell Completion](https://pdm.fming.dev/latest/#shell-completion).

## Creating and Configuring a Project

### How the Project Was Initialized

Create and move to the project folder:

```shell
mkdir -p apps/examples-packaging-pdm
cd apps/examples-packaging-pdm
```

Initialize the project:

```shell
pdm init -n --dist
```

If you omit options, you will be prompted for input.  
`-n, --non-interactive` and `--dist` are recommended for automation.

Example directory structure:

```console
.
├── .gitignore
├── .pdm-python
├── README.md
├── pyproject.toml
├── src
│   └── examples_packaging_pdm
│       └── __init__.py
└── tests
    └── __init__.py
```

### Install Dependency Packages

Install development dependencies:

```shell
pdm add -d flake8 mypy black isort pytest-cov pyclean
```

### The src Layout

PDM autodetects the `src` layout.

- [The src layout](https://backend.pdm-project.org/build_config/#the-src-layout)

### Use a Task Runner Like npm

PDM supports custom script shortcuts in the `[tool.pdm.scripts]` section of `pyproject.toml`.

**`pyproject.toml`**:

```toml
[tool.pdm.scripts]
clean = "pyclean ."
clean_all_dirs_ = {shell = "find . | grep -E \"(/__pycache__$|\\.pyc$|\\.pyo$|/build$|/dist$)\" | xargs rm -rf"}
lint = "flake8"
test = "pytest"
```

Run a task:

```shell
pdm run clean
```

### Dynamic Versioning

See [Dynamic project version](https://backend.pdm-project.org/metadata/#dynamic-project-version).

**`__init__.py`**:

```python
__version__ = "0.1.1"
```

`pyproject.toml` (diff):

```diff
- version = "0.1.0"
+ dynamic = ["version"]

+ [tool.pdm.version]
+ path = "apps/examples_packaging_pdm/__init__.py"
+ source = "file"
```

### Provide Console Scripts

How to provide console scripts in a PDM package:

- [Console scripts - PEP 621 Metadata](https://pdm-project.org/latest/reference/pep621/#console-scripts)

Create an entry point:

```python
def main() -> None:
    print("Hello pdm.")
```

`pyproject.toml`:

```toml
[project.scripts]
examples-pdm-cli = "examples_packaging_pdm.console.command:main"
```

Install the package locally:

```shell
pdm install
```

Run the script:

```console
$ examples-pdm-cli

Hello pdm.
```

## Package Management

### `init` Project Setup

Initialize a project and answer prompts to create `pyproject.toml`:

```shell
pdm init
```

Example prompt:

```console
$ pdm init

Creating a pyproject.toml for PDM...
Please enter the Python interpreter to use
1. /workspaces/examples-py/.venv/bin/python (3.11)
2. /usr/local/bin/python3.11 (3.11)
3. /usr/bin/python3.9 (3.9)
Please select (0):
Using Python interpreter: /workspaces/examples-py/.venv/bin/python (3.11)

Is the project a library that is installable?
If yes, we will need to ask a few more questions to include the project name and build backend [y/n] (n): y
Project name (examples-py):
Project version (0.1.0):
Project description ():
Which build backend to use?
0. pdm-backend
1. setuptools
2. flit-core
3. hatchling
4. pdm-pep517
Please select (0):
License(SPDX name) (MIT):
Author name (A.suzuki):
Author email (suzu.devworks@gmail.com):
Python requires('*' to allow any) (>=3.11):
Changes are written to pyproject.toml.
```

### `venv` Use Virtualenv

PDM automatically creates a virtual environment the first time you run `pdm install`.

```shell
pdm install
```

Virtual environments are used if the project interpreter (stored in `.pdm-python`, check with `pdm info`) is from a virtualenv.

Create a virtualenv manually:

```shell
pdm venv create 3.12
```

Select a virtualenv:

```shell
pdm use -i -f .venv
```

Omitting the argument makes it interactive.

List virtualenvs:

```console
$ pdm venv list

Virtualenvs created with this project:

*  in-project: /workspaces/examples-py-packaging/apps/examples-packaging-pdm/.venv
```

Remove a virtualenv:

```shell
pdm venv remove in-project
```

### `sync/update/install` Restore Packages

- `pdm sync` installs packages from lock files.
- `pdm update` updates the lock file and then syncs.
- `pdm install` checks for changes, updates the lock file if needed, and then syncs.

`pdm install` installs the current project in editable mode.

### `add` Add Package

Add a package:

```shell
pdm add {packages...}
```

Add a development-only dependency:

```shell
pdm add --dev {packages...}
pdm add --dev --group {group} {packages...}
```

Add editable dependencies:

```shell
pdm add -e {location} --dev
```

### `update` Update Package

Update a package:

```shell
pdm update {packages...}
```

### `update` Update Outdated Packages

Show outdated packages:

```shell
pdm list --outdated
```

Update all dependencies:

```shell
pdm update
```

### `remove` Remove Package

Remove a package:

```shell
pdm remove {packages...}
```

### `list` List Packages

List installed packages:

```shell
pdm list
```

Show a dependency tree:

<!-- spell-checker: disable  -->
```console
$ pdm list --tree

black 24.4.2 [ required: >=24.4.2 ]
├── click 8.1.7 [ required: >=8.0.0 ]
├── mypy-extensions 1.0.0 [ required: >=0.4.3 ]
├── packaging 24.1 [ required: >=22.0 ]
├── pathspec 0.12.1 [ required: >=0.9.0 ]
└── platformdirs 4.2.2 [ required: >=2 ]
examples-packaging-pdm 0.1.3 [ required: This project ]
flake8 7.1.0 [ required: >=7.1.0 ]
├── mccabe 0.7.0 [ required: <0.8.0,>=0.7.0 ]
├── pycodestyle 2.12.0 [ required: <2.13.0,>=2.12.0 ]
└── pyflakes 3.2.0 [ required: <3.3.0,>=3.2.0 ]
isort 5.13.2 [ required: >=5.13.2 ]
mypy 1.11.0 [ required: >=1.11.0 ]
├── mypy-extensions 1.0.0 [ required: >=1.0.0 ]
└── typing-extensions 4.12.2 [ required: >=4.6.0 ]
pip 24.0 [ Not required ]
pyclean 3.0.0 [ required: >=3.0.0 ]
pytest-cov 5.0.0 [ required: >=5.0.0 ]
├── coverage[toml] 7.6.0 [ required: >=5.2.1 ]
└── pytest 8.2.2 [ required: >=4.6 ]
    ├── iniconfig 2.0.0 [ required: Any ]
    ├── packaging 24.1 [ required: Any ]
    └── pluggy 1.5.0 [ required: <2.0,>=1.5 ]
```
<!-- spell-checker: enable  -->

### `build` Build Package

Build the package:

```shell
pdm build
```

Example output:

```console
$ pdm build --no-sdist

Building wheel from sdist...
Built wheel at /workspaces/examples-py-packaging/apps/examples-packaging-pdm/dist/examples_packaging_pdm-0.1.3-py3-none-any.whl
```

## References

- [PDM Documentation](https://pdm.fming.dev/latest/)
- [PDM GitHub Repository](https://github.com/pdm-project/pdm)

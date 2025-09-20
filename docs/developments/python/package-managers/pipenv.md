# Pipenv

Pipenv is a Python virtual environment management tool that bridges the gaps between pip, pyenv, and virtualenv.

It combines pip and venv.  
Note: Pipenv does not provide wheel package creation; use another tool such as setuptools for packaging.

## Table of Contents <!-- omit in toc -->

- [Installation](#installation)
- [Creating and Configuring a Project](#creating-and-configuring-a-project)
  - [How the Project Was Initialized](#how-the-project-was-initialized)
  - [Install Packages](#install-packages)
  - [Custom Script Shortcuts](#custom-script-shortcuts)
  - [Provide Console Scripts](#provide-console-scripts)
- [Project Management](#project-management)
  - [Project Setup and Create Virtualenv](#project-setup-and-create-virtualenv)
  - [Remove the Virtualenv](#remove-the-virtualenv)
  - [Spawn a Shell Within the Virtualenv](#spawn-a-shell-within-the-virtualenv)
  - [`run` Spawn a Command Installed into the Virtualenv](#run-spawn-a-command-installed-into-the-virtualenv)
  - [`install/sync` Restore Packages](#installsync-restore-packages)
  - [`install` Add Packages](#install-add-packages)
  - [`update` Update Package](#update-update-package)
  - [`update` Update Outdated Packages](#update-update-outdated-packages)
  - [`uninstall` Remove Packages](#uninstall-remove-packages)
  - [`clean` Uninstall All Packages](#clean-uninstall-all-packages)
  - [`graph` List Packages](#graph-list-packages)
  - [Build Wheel Package](#build-wheel-package)
- [Tips](#tips)
  - [Where Are Site-Packages Located?](#where-are-site-packages-located)
  - [Read Environment File](#read-environment-file)
- [References](#references)

## Installation

Install using pip (recommended in a venv):

```shell
pip install --user pipenv
```

## Creating and Configuring a Project

### How the Project Was Initialized

Create and move to the project folder:

```shell
mkdir -p apps/examples-packaging-pipenv
cd apps/examples-packaging-pipenv
```

Generate a `Pipfile`:

```shell
# Use pyenv if you want other Python versions.
# A virtual environment is not created if executed inside an existing venv.
pipenv --python 3.11
export PIPENV_VENV_IN_PROJECT=true
```

Create the initial package directory:

```shell
mkdir -p src/examples_packaging_pipenv/
touch src/examples_packaging_pipenv/__init__.py
```

Example directory structure:

```console
.
├── Pipfile
├── Pipfile.lock
├── README.md
└── src
    └── examples_packaging_pipenv
        └── __init__.py
```

### Install Packages

Install development dependencies:

```shell
pipenv install --dev flake8 mypy black isort pytest-cov
```

`Pipfile` and `Pipfile.lock` are updated.

### Custom Script Shortcuts

Add scripts to your `Pipfile`:

```ini
[scripts]
format = "black -l 119 ."
lint = "flake8 --show-source ."
start = "python main.py runserver"
test = "pytest"
```

Run a script:

```shell
pipenv run start
```

### Provide Console Scripts

Pipenv cannot create packages as commands, but you can define Python commands using custom script shortcuts.

Example entry point:  
`src/examples_packaging_pipenv/console/command.py`:

```python
import sys

def main():
    print("Hello pipenv.")

if __name__ == "__main__":
    sys.exit(main())
```

Add to `Pipfile`:

```ini
[scripts]
start = "python src/examples_packaging_pipenv/console/command.py"
```

Run:

```shell
pipenv run start
```

Output:

```console
Hello pipenv.
```

## Project Management

### Project Setup and Create Virtualenv

Generate a Pipfile:

```shell
pipenv --python {py-version}
```

To create the virtualenv inside your project folder:

```shell
export PIPENV_VENV_IN_PROJECT=true
pipenv --python 3.8
```

If the specified Python version is not installed, Pipenv will prompt to install it via pyenv.

### Remove the Virtualenv

```shell
pipenv --rm
```

### Spawn a Shell Within the Virtualenv

```shell
pipenv shell
```

### `run` Spawn a Command Installed into the Virtualenv

```shell
pipenv run {command}
```

### `install/sync` Restore Packages

From `Pipfile`:

```shell
pipenv install
pipenv install --dev
```

From `Pipfile.lock`:

```shell
pipenv sync
pipenv sync --dev
```

From `requirements.txt`:

```shell
pipenv install -r ./requirements.txt
```

### `install` Add Packages

Add a package:

```shell
pipenv install {packages...}
pipenv install --dev {packages...}
```

### `update` Update Package

Update a package and sync:

```shell
pipenv update {packages...}
```

### `update` Update Outdated Packages

Show outdated packages:

```shell
pipenv update --outdated
```

Update all dependencies:

```shell
pipenv update
```

### `uninstall` Remove Packages

Remove a package:

```shell
pipenv uninstall {packages...}
```

### `clean` Uninstall All Packages

Uninstall all packages not specified in `Pipfile.lock`:

```shell
pipenv clean
```

### `graph` List Packages

Show dependency tree:

```shell
pipenv graph
```

### Build Wheel Package

**Pipenv does not provide package creation. Use another tool such as setuptools to build wheel packages.**

## Tips

### Where Are Site-Packages Located?

Show the virtualenv location:

```shell
pipenv --venv
```

### Read Environment File

If you prepare a `.env` file in the project folder, it will be automatically loaded when using `pipenv shell` or `pipenv run`.

Example `.env`:

```ini
DEBUG=1
```

Test:

```shell
pipenv run python
```

```python
import os
print(os.environ['DEBUG'])  # '1'
```

## References

- [Pipenv Documentation](https://pipenv.pypa.io/en/latest/)
- [Pipenv GitHub Repository](https://github.com/pypa/pipenv)

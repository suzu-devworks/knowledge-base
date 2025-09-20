# Poetry

Poetry is a tool for dependency management and packaging in Python. It allows you to declare the libraries your project depends on and it will manage (install/update) them for you.

[![Poetry](https://img.shields.io/endpoint?url=https://python-poetry.org/badge/v0.json)](https://python-poetry.org/)

## Table of Contents <!-- omit in toc -->

- [Installation](#installation)
  - [Using pip](#using-pip)
  - [Self update](#self-update)
- [Creating and Configuring a Project](#creating-and-configuring-a-project)
  - [How the Project Was Initialized](#how-the-project-was-initialized)
  - [Install Packages](#install-packages)
  - [src-layout](#src-layout)
  - [Provide Console Scripts](#provide-console-scripts)
- [Commands](#commands)
  - [`new/init` Project Setup](#newinit-project-setup)
  - [Create Virtualenv](#create-virtualenv)
  - [Remove the Virtualenv](#remove-the-virtualenv)
  - [`shell` Spawn a Shell Within the Virtualenv](#shell-spawn-a-shell-within-the-virtualenv)
  - [`run` Spawn a Command Installed into the Virtualenv](#run-spawn-a-command-installed-into-the-virtualenv)
  - [`install` Restore Packages](#install-restore-packages)
  - [`add` Add Package](#add-add-package)
  - [`show` List Packages](#show-list-packages)
  - [`update` Update Package](#update-update-package)
  - [Update Outdated Packages](#update-outdated-packages)
  - [`remove` Uninstall Package](#remove-uninstall-package)
  - [`build` Build Package](#build-build-package)
- [Tips](#tips)
  - [Use a Task Runner Like npm](#use-a-task-runner-like-npm)
- [References](#references)

## Installation

### Using pip

You can install Poetry using pip:

```shell
pip install poetry
```

### Self update

Update Poetry itself:

```shell
poetry self update
```

## Creating and Configuring a Project

### How the Project Was Initialized

Create and move to the project folder:

```shell
mkdir -p apps/examples-packaging-poetry
cd apps/examples-packaging-poetry
```

Generate a template for your project:

```shell
poetry new .
```

Example directory structure:

```console
.
├── README.md
├── pyproject.toml
├── examples_packaging_poetry
│   └── __init__.py
└── tests
    └── __init__.py
```

### Install Packages

Install development dependencies:

```shell
poetry add --group dev flake8 mypy black isort pytest-cov pyclean
```

### src-layout

To use the `src` layout, move your package into a `src` directory and configure `pyproject.toml`:

```console
.
├── README.md
├── pyproject.toml
├── src
│   └── examples_packaging_poetry
│       └── __init__.py
└── tests
    └── __init__.py
```

`pyproject.toml`:

```toml
[tool.poetry]
packages = [{include = "examples_packaging_poetry", from = "src"}]
```

### Provide Console Scripts

To provide console scripts in a Poetry package:

Create an entry point, e.g. `src/examples_packaging_poetry/console/command.py`:

```python
def main() -> None:
    print("Hello poetry.")
```

Add to `pyproject.toml`:

```toml
[tool.poetry.scripts]
examples-poetry-cli = "examples_packaging_poetry.console.command:main"
```

Install the package locally:

```shell
poetry install
```

Run the script:

```console
$ examples-poetry-cli

Hello poetry.
```

## Commands

### `new/init` Project Setup

Create a new project:

```shell
mkdir {project}
cd {project}
poetry new .
```

Or initialize in an existing folder:

```shell
poetry init
```

`init` generates `pyproject.toml`.  
`new` also creates initial directories and files.

### Create Virtualenv

Install dependencies and create a virtual environment:

```shell
poetry install
```

By default, Poetry creates a virtual environment in `{cache-dir}/virtualenvs`.

To create the virtualenv inside your project folder:

```shell
poetry config virtualenvs.in-project true --local
```

Poetry will detect and respect an existing activated virtual environment.

### Remove the Virtualenv

```shell
poetry env remove {virtualenv}
```

### `shell` Spawn a Shell Within the Virtualenv

```shell
poetry shell
```

### `run` Spawn a Command Installed into the Virtualenv

```shell
poetry run {command}
```

### `install` Restore Packages

Install dependencies from `pyproject.toml`:

```shell
poetry install
```

Current projects are installed in editable mode by default.

### `add` Add Package

Add dependencies:

```shell
poetry add {packages...}
```

### `show` List Packages

List installed packages:

```shell
poetry show
```

### `update` Update Package

Update a specific package:

```shell
poetry update {packages...}
```

Equivalent to deleting `poetry.lock` and running install.

### Update Outdated Packages

Show outdated packages:

```shell
poetry show --outdated
```

Update all dependencies:

```shell
poetry update
```

`poetry.lock` will be updated.

### `remove` Uninstall Package

Remove a package:

```shell
poetry remove {packages...}
```

### `build` Build Package

Build a wheel and source distribution:

```shell
poetry build
```

## Tips

### Use a Task Runner Like npm

You can use [taskipy](https://github.com/taskipy/taskipy) as a complementary task runner for Python.

Install taskipy as a dev dependency:

```shell
poetry add --group dev taskipy
```

Add tasks to your `pyproject.toml`:

```toml
[tool.taskipy.tasks]
clean = "pyclean ."
test = "pytest"
```

Run a task:

```shell
poetry run task clean
# or
task clean
```

## References

- [Poetry Official Site](https://python-poetry.org/)
- [Poetry GitHub Repository](https://github.com/python-poetry/poetry)

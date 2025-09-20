# Rye

Rye is a comprehensive project and package management solution for Python.

[![Rye](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/rye/main/artwork/badge.json)](https://rye.astral.sh)

> [!NOTE]
> _If you're getting started with Rye, consider uv, the successor project from the same maintainers._

## Table of Contents <!-- omit in toc -->

- [Installation](#installation)
  - [Using the Standalone Installer](#using-the-standalone-installer)
  - [Self update](#self-update)
- [Configurations](#configurations)
  - [Use uv](#use-uv)
- [Creating and Configuring a Project](#creating-and-configuring-a-project)
  - [How the Project Was Initialized](#how-the-project-was-initialized)
  - [Install Dependency Packages](#install-dependency-packages)
  - [Provide Console Scripts](#provide-console-scripts)
- [Creating and Configuring Workspaces](#creating-and-configuring-workspaces)
  - [Workspaces Layout](#workspaces-layout)
  - [Create Root Project](#create-root-project)
  - [Append Packages Project](#append-packages-project)
- [Package Management](#package-management)
  - [`add` Add Dependencies](#add-add-dependencies)
  - [`remove` Remove Dependencies](#remove-remove-dependencies)
  - [`lock` Update Lockfiles Without Installing Dependencies](#lock-update-lockfiles-without-installing-dependencies)
  - [`list` List Dependencies](#list-list-dependencies)
  - [`sync` Update the Project’s Environment](#sync-update-the-projects-environment)
  - [`build` Build Python Packages](#build-build-python-packages)
  - [`run` Run a Command](#run-run-a-command)
- [References](#references)

## Installation

### Using the Standalone Installer

To install Rye, run:

```shell
curl -sSf https://rye.astral.sh/get | bash
```

You can customize the install script with environment variables:

- `RYE_INSTALL_OPTION="--yes"`: Skip all prompts.
- `RYE_TOOLCHAIN={path to python}`: Set the Python interpreter to use as the internal interpreter.

After installation, load the environment and enable Rye:

```shell
source "$HOME/.rye/env"
```

This is typically added to your shell profile during installation.

```console
$ tail -n 5 ~/.profile

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/.local/bin" ] ; then
    PATH="$HOME/.local/bin:$PATH"
fi
. "$HOME/.rye/env"
```

### Self update

Update Rye itself:

```shell
rye self update
```

## Configurations

### Use uv

uv is a Python package manager also developed by Astral.

Rye 0.37.0 and later have uv enabled by default.

```console
$ rye config --get behavior.use-uv
true
```

## Creating and Configuring a Project

### How the Project Was Initialized

Create a new project:

```shell
rye init apps/examples-packaging-rye
cd apps/examples-packaging-rye
```

Example directory structure:

```console
.
├── .gitignore
├── .python-version
├── README.md
├── pyproject.toml
└── src
    └── examples_packaging_rye
        └── __init__.py
```

After initialization, run:

```shell
rye sync
```

This creates a virtualenv in `.venv` and writes lockfiles to `requirements.lock` and `requirements-dev.lock`.

### Install Dependency Packages

Install development dependencies:

```shell
rye add -d ruff mypy pytest-cov pyclean
```

### Provide Console Scripts

To provide console scripts in a Rye package:

- [project.scripts - Python Project (pyproject.toml)](https://rye.astral.sh/guide/pyproject/#projectscripts)

Create an entry point:

```python
def main() -> None:
    print("Hello rye.")
```

Add to `pyproject.toml`:

```toml
[project.scripts]
examples-rye-cli = "examples_packaging_rye.console.command:main"
```

Install the package locally:

```shell
rye sync
```

Run the script:

```console
$ examples-rye-cli

Hello rye.
```

## Creating and Configuring Workspaces

### Workspaces Layout

Example structure:

```console
examples-py
├── packages
│   ├── examples-cli
│   │   ├── .gitignore
│   │   ├── pyproject.toml
│   │   ├── .python-version
│   │   ├── README.md
│   │   └── src
│   │       └── examples_cli
│   │           ├── __init__.py
│   │           └── __main__.py
│   └── examples-lib
│       ├── .gitignore
│       ├── pyproject.toml
│       ├── .python-version
│       ├── README.md
│       └── src
│           └── examples_lib
│               └── __init__.py
├── pyproject.toml
├── .python-version
├── README.md
├── requirements-dev.lock
└── requirements.lock
```

### Create Root Project

Create the root project as a virtual project:

```shell
rye init --virtual
```

Add workspace members to your `pyproject.toml`:

```toml
[tool.rye.workspace]
members = ["packages/*"]
```

Or with echo:

```shell
echo $'\n[tool.rye.workspace]\nmembers = ["packages/*"]' >> pyproject.toml 
```

Optionally add global dev packages:

```shell
rye add --dev ruff mypy pyclean
```

### Append Packages Project

Create each package project:

```shell
rye init packages/examples-lib
rye init --script packages/examples-cli
```

Add a reference to another package:

```shell
cd packages/examples-cli
rye add examples-lib --path ../examples-lib
cd ../../
```

## Package Management

### `add` Add Dependencies

```shell
# Specify a version
rye add "flask>=2.0"

# Add as a dev dependency
rye add --dev ruff
```

### `remove` Remove Dependencies

```shell
rye remove flask
```

### `lock` Update Lockfiles Without Installing Dependencies

```shell
rye lock --upgrade-package flask

# Update all packages
rye lock --update-all
```

### `list` List Dependencies

```shell
rye list
```

Example output:

```console
-e file:///workspaces/examples-py/packages/examples-cli
-e file:///workspaces/examples-py/packages/examples-lib
mypy==1.13.0
mypy-extensions==1.0.0
pyclean==3.0.0
ruff==0.8.3
typing-extensions==4.12.2
```

### `sync` Update the Project’s Environment

```shell
rye sync
```

### `build` Build Python Packages

```shell
rye build --all
```

### `run` Run a Command

```shell
rye run --pyproject packages/examples-cli/pyproject.toml examples-cli
```

## References

- [Rye Documentation](https://rye.astral.sh/)

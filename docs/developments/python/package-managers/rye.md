# Rye

Rye is a comprehensive project and package management solution for Python.

[![Rye](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/rye/main/artwork/badge.json)](https://rye.astral.sh)

> [!NOTE]
> _If you're getting started with Rye, consider uv, the successor project from the same maintainers._

- [Rye](#rye)
  - [Installation](#installation)
    - [Using the Standalone Installer](#using-the-standalone-installer)
    - [Self update](#self-update)
  - [Configurations](#configurations)
    - [Use uv](#use-uv)
  - [Creating and Configuring a project](#creating-and-configuring-a-project)
    - [How the project was initialized](#how-the-project-was-initialized)
    - [Install dependency packages](#install-dependency-packages)
    - [Provide Console Scripts](#provide-console-scripts)
  - [Creating and Configuring a workspaces](#creating-and-configuring-a-workspaces)
    - [Workspaces layout](#workspaces-layout)
    - [Create root project](#create-root-project)
    - [Append packages project](#append-packages-project)
  - [Package management](#package-management)
    - [`add` Add dependencies](#add-add-dependencies)
    - [`remove` Remove dependencies](#remove-remove-dependencies)
    - [`lock` Updates the lockfiles without installing dependencies](#lock-updates-the-lockfiles-without-installing-dependencies)
    - [`list` List dependencies](#list-list-dependencies)
    - [`sync` Update the project’s environment](#sync-update-the-projects-environment)
    - [`build` Build Python packages](#build-build-python-packages)
    - [`run` Run a command](#run-run-a-command)
  - [References](#references)

## Installation

### Using the Standalone Installer

To install you can run a curl command:

```shell
curl -sSf https://rye.astral.sh/get | bash
```

The install script that is piped to bash can be customized with some environment variables:

`RYE_INSTALL_OPTION="--yes"`:

Skip all prompts.

`RYE_TOOLCHAIN={path to python}`

Set it to point to the Python interpreter you want to use as the internal interpreter.

After installation, load the environment and enable rye.

```shell
source "$HOME/.rye/env"
```

It seems to be added to the shell profile during installation.

```console
$ tail -n 5 ~/.profile

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/.local/bin" ] ; then
    PATH="$HOME/.local/bin:$PATH"
fi
. "$HOME/.rye/env"
```

### Self update

```shell
rye self update
```

## Configurations

### Use uv

uv is a Python package manager also developed by Astral.

Rye 0.37.0 already has UV enabled.

```console
$ rye config --get behavior.use-uv

true
```

## Creating and Configuring a project

### How the project was initialized

Create project this command:

```shell
rye init apps/examples-packaging-rye
```

Move to the root of your project...

```shell
cd apps/examples-packaging-rye
```

The final directory will look like this:

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

Once that is done, you can use rye sync to get the first synchronization. After that, Rye will have created a virtualenv in `.venv` and written lockfiles into `requirements.lock` and `requirements-dev.lock`.

```shell
rye sync
```

### Install dependency packages

Install dependency packages for this project:

```shell
rye add -d ruff mypy pytest-cov pyclean
```

### Provide Console Scripts

How to provide console scripts in the PDM package.

- [project.scripts - Python Project (pyproject.toml)](https://rye.astral.sh/guide/pyproject/#projectscripts)

Create entry point:

```py
def main() -> None:
    print("Hello rye.")
```

`pyproject.toml`:

```toml
[project.scripts]
examples-rye-cli = "examples_packaging_rye.console.command:main"
```

Install the package locally:

```shell
rye sync
```

When you run it:

```console
$ examples-rye-cli

Hello rye.
```

## Creating and Configuring a workspaces

### Workspaces layout

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

### Create root project

Create the root project as a virtual project:

```shell
rye init --virtual
```

Add the following to your pyproject.toml:

```toml
[tool.rye.workspace]
members = ["packages/*"]
```

If you do it with echo:

```shell
echo $'\n[tool.rye.workspace]\nmembers = ["packages/*"]' >> pyproject.toml 
```

Optionally add packages for global use:

```shell
rye add --dev ruff mypy pyclean
```

### Append packages project

Create each project:

```shell
rye init packages/examples-lib
rye init --script packages/examples-cli
```

Add a reference project:

```shell
cd packages/examples-cli
rye add examples-lib --path ../examples-lib
cd ../../
```

## Package management

### `add` Add dependencies

```shell
# Specify a version
rye add "flask>=2.0"

# Add this as dev dependency
rye add --dev ruff
```

### `remove` Remove dependencies

```shell
rye remove flask
```

### `lock` Updates the lockfiles without installing dependencies

```shell
rye lock --upgrade-package flask

# All
rye lock--update-all
```

### `list` List dependencies

```shell
rye list
```

<!-- /* spell-checker:disable */ -->
```console
-e file:///workspaces/examples-py/packages/examples-cli
-e file:///workspaces/examples-py/packages/examples-lib
mypy==1.13.0
mypy-extensions==1.0.0
pyclean==3.0.0
ruff==0.8.3
typing-extensions==4.12.2
```
<!-- /* spell-checker:enable */ -->

### `sync` Update the project’s environment

```shell
rye sync
```

### `build` Build Python packages

```shell
rye build --all
```

### `run` Run a command

```shell
rye run --pyproject packages/examples-cli/pyproject.toml examples-cli
```

## References

- <https://rye.astral.sh/>

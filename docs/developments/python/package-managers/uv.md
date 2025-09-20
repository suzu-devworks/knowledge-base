# uv

An extremely fast Python package and project manager, written in Rust.

[![uv](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/uv/main/assets/badge/v0.json)](https://github.com/astral-sh/uv)

- [uv](#uv)
  - [Installation](#installation)
    - [Using the Standalone Installer](#using-the-standalone-installer)
    - [Use with Docker](#use-with-docker)
    - [Self update](#self-update)
  - [Configurations](#configurations)
    - [Whether to allow Python downloads](#whether-to-allow-python-downloads)
  - [Creating and Configuring a workspaces](#creating-and-configuring-a-workspaces)
    - [Workspaces layout](#workspaces-layout)
    - [Create workspace root](#create-workspace-root)
    - [Add packages projects](#add-packages-projects)
  - [Project templates](#project-templates)
    - [`init --app`(default)](#init---appdefault)
    - [`init --package`](#init---package)
    - [`init --no-package`](#init---no-package)
    - [`init --lib`](#init---lib)
    - [`init --script`](#init---script)
  - [Package management](#package-management)
    - [`add` Add dependencies](#add-add-dependencies)
    - [`remove` Remove dependencies](#remove-remove-dependencies)
    - [`lock` Update the project's lockfile](#lock-update-the-projects-lockfile)
    - [`tree` Display the project's dependency tree](#tree-display-the-projects-dependency-tree)
    - [`sync` Update the project’s environment](#sync-update-the-projects-environment)
    - [`build` Build Python packages](#build-build-python-packages)
    - [`run` Run a command](#run-run-a-command)
  - [References](#references)

## Installation

### Using the Standalone Installer

To install you can run a curl command:

```shell
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Use with Docker

There are two ways to use uv with Docker:

- Copy the binaries from the official image
- Install using the installer

Since this repository uses the DevContainers base image, we'll install it with the installer using postCreateCommand.

### Self update

```shell
uv self update
```

## Configurations

### Whether to allow Python downloads

[See here...](https://docs.astral.sh/uv/reference/settings/#python-downloads)

For `pyproject.toml`:

```toml
[tool.uv]
python-downloads = "manual"
```

## Creating and Configuring a workspaces

### Workspaces layout

```console
examples-py
├── .gitignore
├── packages
│   ├── examples-cli
│   │   ├── pyproject.toml
│   │   ├── README.md
│   │   └── src
│   │       └── examples_cli
│   │           └── __init__.py
│   └── examples-lib
│       ├── pyproject.toml
│       ├── README.md
│       └── src
│           └── examples_lib
│               ├── __init__.py
│               └── py.typed
├── pyproject.toml
├── .python-version
├── README.md
└── uv.lock
```

### Create workspace root

Create the root project:

```shell
uv init --author-from auto
```

We don't need the accompanying `hello.py`, so delete it.

```shell
rm hello.py
```

Add the following to your pyproject.toml:

```toml
[tool.uv.workspace]
members = ["packages/*"]

[build-system]
build-backend = "hatchling.build"
requires = ["hatchling"]

[tool.hatch.build.targets.wheel]
packages = ["src/examples_py"]
```

Specify `tool.hatch.build.targets.wheel.packages` to not collect unnecessary files during build.
No error occurs.

Optionally add packages for global use:

```shell
uv add --dev ruff mypy pyclean
uv add --dev pytest pytest-asyncio pytest-cov
```

Maybe uv tool is better, but I haven't studied it enough.

### Add packages projects

Create each project:

```shell
uv init --lib packages/examples-lib
uv init --package packages/examples-cli
```

Add a reference project:

```shell
uv add --project packages/examples-cli examples-lib
```

Should I add everything to the root?

```shell
uv add examples-cli examples-lib
```

It will be automatically filled in `pyproject.toml`

```toml
dependencies = [
  "examples-cli",
  "examples-lib",
]

[tool.uv.sources]
examples-cli = { workspace = true }
examples-lib = { workspace = true }
```

## Project templates

### `init --app`(default)

Create a project for an application.

```console
examples-app
├── .python-version
├── README.md
├── hello.py
└── pyproject.toml
```

### `init --package`

Set up the project to be built as a Python package.

```console
examples-package/
├── .python-version
├── README.md
├── pyproject.toml
└── src
    └── examples_package
        └── __init__.py
```

### `init --no-package`

Do not set up the project to be built as a Python package.

```console
examples-no-package
├── .python-version
├── README.md
├── hello.py
└── pyproject.toml
```

### `init --lib`

Create a project for a library.

```console
examples-lib
├── .python-version
├── README.md
├── pyproject.toml
└── src
    └── examples_lib
        ├── __init__.py
        └── py.typed
```

### `init --script`

Create a script.

```python
# /// script
# requires-python = ">=3.12"
# dependencies = []
# ///


def main() -> None:
    print("Hello from examples-script!")


if __name__ == "__main__":
    main()
```

## Package management

### `add` Add dependencies

```shell
uv add requests

# Specify a version constraint
uv add 'requests==2.31.0'

# Add a git dependency
uv add git+https://github.com/psf/requests
```

### `remove` Remove dependencies

```shell
uv remove requests
```

### `lock` Update the project's lockfile

```shell
uv lock --upgrade-package requests

# All 
uv lock --upgrade
```

### `tree` Display the project's dependency tree

```shell
uc tree

# Show the latest available version of each package in the tree
uv tree --outdated
```

<!-- /* spell-checker:disable */ -->
```console
Resolved 16 packages in 2ms
examples-py v0.1.0
├── examples-cli v0.1.0
│   └── examples-lib v0.1.0
├── examples-lib v0.1.0
├── mypy v1.13.0 (group: dev)
│   ├── mypy-extensions v1.0.0
│   └── typing-extensions v4.12.2
├── pyclean v3.0.0 (group: dev)
├── pytest v8.3.4 (group: dev)
│   ├── iniconfig v2.0.0
│   ├── packaging v24.2
│   └── pluggy v1.5.0
├── pytest-asyncio v0.24.0 (group: dev)
│   └── pytest v8.3.4 (*)
├── pytest-cov v6.0.0 (group: dev)
│   ├── coverage v7.6.9
│   └── pytest v8.3.4 (*)
└── ruff v0.8.2 (group: dev)
(*) Package tree already displayed
```
<!-- /* spell-checker:enable */ -->

### `sync` Update the project’s environment

root only:

```shell
uv sync
```

or specified package:

```shell
uv sync --project packages/examples-lib
```

or all packages:

```shell
uv sync --all-packages
```

### `build` Build Python packages

root only:

```shell
uv build
```

or specified package:

```shell
uv build --project packages/examples-lib
```

or all packages:

```shell
uv build --all-packages
```

### `run` Run a command

Run the command within the given project directory:

```shell
uv run --project packages/examples-cli {command} {command-options}
```

Change to the given directory prior to running the command:

```shell
uv run --directory packages/examples-cli {command} {command-options}
```

## References

- <https://docs.astral.sh/uv/>

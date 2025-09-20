
# Poetry

Poetry is a tool for dependency management and packaging in Python. It allows you to declare the libraries your project depends on and it will manage (install/update) them for you.

[![Poetry](https://img.shields.io/endpoint?url=https://python-poetry.org/badge/v0.json)](https://python-poetry.org/)

- [Poetry](#poetry)
  - [Installation](#installation)
    - [Using pip](#using-pip)
    - [Self update](#self-update)
  - [Creating and Configuring a project](#creating-and-configuring-a-project)
    - [How the project was initialized](#how-the-project-was-initialized)
    - [Install packages](#install-packages)
    - [src-layout](#src-layout)
    - [Provide Console Scripts](#provide-console-scripts)
  - [Commands](#commands)
    - [`new/init` Project setup](#newinit-project-setup)
    - [Create virtualenv](#create-virtualenv)
    - [Remove the virtualenv](#remove-the-virtualenv)
    - [`shell` Spawns a shell within the virtualenv](#shell-spawns-a-shell-within-the-virtualenv)
    - [`run` Spawns a command installed into the virtualenv](#run-spawns-a-command-installed-into-the-virtualenv)
    - [`install` Restore packages](#install-restore-packages)
    - [`add` Add package](#add-add-package)
    - [`show` List packages](#show-list-packages)
    - [`update` Update package](#update-update-package)
    - [Update outdated packages](#update-outdated-packages)
    - [`remove` Uninstall package](#remove-uninstall-package)
    - [`build` Build package](#build-build-package)
  - [Tips](#tips)
    - [Use a task runner like npm](#use-a-task-runner-like-npm)
  - [References](#references)

## Installation

### Using pip

I installed it with pip and it seems fine.

```shell
pip install poetry
```

### Self update

Poetry itself can be updated:

```shell
poetry self update
```

## Creating and Configuring a project

### How the project was initialized

Create and move project folder:

```shell
mkdir -p apps/examples-packaging-poetry
cd apps/examples-packaging-poetry
```

Generate a template for your project with the following command:

```shell
poetry new .
```

The final directory will look like this:

```console
.
├── README.md
├── pyproject.toml
├── examples_packaging_poetry
│   └── __init__.py
└── tests
    └── __init__.py
```

### Install packages

Install dependency packages for this project:

```shell
poetry add --group dev flake8 mypy black isort pytest-cov pyclean
```

### src-layout

If you want to change to src-layout, you need to set it in `pyproject.toml`.

- [packages](https://python-poetry.org/docs/pyproject#packages)

Move the whole folder:

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

How to provide console scripts in the poetry package.

Create entry point `apps/examples_packaging_poetry/console/command.py`:

```py
def main() -> None:
    print("Hello poetry.")
```

`pyproject.toml`:

```ini
[tool.poetry.scripts]
examples-poetry-cli = 'examples_packaging_poetry.console.command:main'
```

Install the package locally:

```shell
poetry install
```

When you run it:

```console
$ examples-poetry-cli

Hello poetry.
```

## Commands

### `new/init` Project setup

```shell
mkdir {project}
cd {project}

poetry new .

or

poetry init
```

`init` is generates `pyproject.toml`.  
`new` also creates an initial directories and files.

### Create virtualenv

Create a virtual environment when installing dependencies

```shell
poetry install
```

By default, Poetry creates a virtual environment in `{cache-dir}/virtualenvs`.

If you want a virtual environment in your project folder, enable the following settings in `poetry.toml`:

```shell
poetry config virtualenvs.in-project true --local
```
<!-- /* spell-checker:words virtualenvs */ -->

however, Poetry will detect and respect an existing virtual environment that has been externally activated.

### Remove the virtualenv

```shell
poetry env remove {virtualenv}
```

### `shell` Spawns a shell within the virtualenv

```shell
poetry shell
```

### `run` Spawns a command installed into the virtualenv

It is not a task start.

```shell
poetry run {command}
```

### `install` Restore packages

Wherever you have `pyproject.toml` do the following

```shell
poetry install
```

Current projects are installed in editable mode by default.

### `add` Add package

How to add dependencies:

```shell
poetry add {packages...}
```

### `show` List packages

```shell
poetry show
```

### `update` Update package

```shell
poetry update {packages...}
```

Same as deleting `poetry.lock` and running install.

### Update outdated packages

show outdated list:

```shell
poetry show --outdated
```

Update all dependency packages:

```shell
poetry update
```

`poetry.lock` should be updated.

### `remove` Uninstall package

```shell
poetry remove {packages...}
```

### `build` Build package

`poetry-core` is python wheels are supported.

```shell
poetry build
```

## Tips

### Use a task runner like npm

The `taskipy` is complementary task runner for python.

<!-- /* spell-checker:words taskipy */ -->

- <https://github.com/taskipy/taskipy>

To install taskipy as a dev dependency, simply run:

```shell
poetry add --group dev taskipy
```

In your pyproject.toml file, add a new section called `[tool.taskipy.tasks]`:

**`pyproject.toml`**:

```toml
[tool.taskipy.tasks]
clean = "pyclean ."
test = "pytest"
```

To called a task, simply run:

```shell
poetry run task clean

or

task clean
```

## References

- <https://python-poetry.org/>
- <https://github.com/python-poetry/poetry>

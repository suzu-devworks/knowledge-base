# PDM

PDM, as described, is a modern Python package and dependency manager supporting the latest PEP standards. But it is more than a package manager.

[![pdm-managed](https://img.shields.io/endpoint?url=https%3A%2F%2Fcdn.jsdelivr.net%2Fgh%2Fpdm-project%2F.github%2Fbadge.json)](https://pdm-project.org)

`pyproject.toml` is PEP 621 supported.

> [!WARNING]
> _PEP 582 has been rejected_
>
> _This is a rejected PEP. However, due to the fact that this feature is the reason for PDM's birth, PDM will retain the support. We recommend using virtual environments instead._

- [PDM](#pdm)
  - [Installation](#installation)
    - [Using the Standalone Installer](#using-the-standalone-installer)
    - [Self update](#self-update)
  - [Configurations](#configurations)
    - [Shell Completion](#shell-completion)
  - [Creating and Configuring a project](#creating-and-configuring-a-project)
    - [How the project was initialized](#how-the-project-was-initialized)
    - [Install dependency packages](#install-dependency-packages)
    - [The src layout](#the-src-layout)
    - [Use a task runner like npm](#use-a-task-runner-like-npm)
    - [Dynamic versioning](#dynamic-versioning)
    - [Provide Console Scripts](#provide-console-scripts)
  - [Package management](#package-management)
    - [`init` Project setup](#init-project-setup)
    - [`venv` Use virtualenv](#venv-use-virtualenv)
    - [`sync/update/install` Restore packages](#syncupdateinstall-restore-packages)
    - [`add` Add package](#add-add-package)
    - [`update` Update package](#update-update-package)
    - [`update` Update outdated packages](#update-update-outdated-packages)
    - [`remove` Remove package](#remove-remove-package)
    - [`list` List packages](#list-list-packages)
    - [`build` Builds package](#build-builds-package)
  - [References](#references)

## Installation

### Using the Standalone Installer

Recommended installation method:

```shell
curl -sSL https://pdm-project.org/install-pdm.py | python3 -
```

Using pip on venv:

```shell
pip install pdm
```

> Doesn't it remove itself when you run `pdm remove`?

### Self update

```shell
pdm self update
```

## Configurations

### Shell Completion

PDM supports generating completion scripts for Bash, Zsh, Fish or Powershell.

for bash:

```shell
pdm completion bash > pdm.bash-completion
sudo mv pdm.bash-completion /etc/bash_completion.d/
```

- [Shell Completion](https://pdm.fming.dev/latest/#shell-completion)

## Creating and Configuring a project

### How the project was initialized

Create and move project folder:

```shell
mkdir -p apps/examples-packaging-pdm
cd apps/examples-packaging-pdm
```

pdm can be initialized in a project using a command:

```shell
pdm init -n --dist
```

If you don't specify it, you will be bombarded with questions, so I specify `-n, --non-interactive` and `--dist`.

The final directory will look like this:

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

### Install dependency packages

Install dependency packages for this project:

```shell
pdm add -d flake8 mypy black isort pytest-cov pyclean
```

### The src layout

PDM seems to autodetect the src layout.

- [The src layout](https://backend.pdm-project.org/build_config/#the-src-layout)

### Use a task runner like npm

PDM also supports custom script shortcuts in the optional `[tool.pdm.scripts]` section of `pyproject.toml`.

**`pyproject.toml`**:

```toml
[tool.pdm.scripts]
clean = "pyclean ."
clean_all_dirs_ = {shell = "find . | grep -E \"(/__pycache__$|\\.pyc$|\\.pyo$|/build$|/dist$)\" | xargs rm -rf"}
lint = "flake8"
test = "pytest"
```

To called a task, simply run:

```shell
pdm run clean
```

### Dynamic versioning

-[Dynamic project version](https://backend.pdm-project.org/metadata/#dynamic-project-version)

**`__init__.py`**:

```py
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

How to provide console scripts in the PDM package.

- [Console scripts - PEP 621 Metadata](https://pdm-project.org/latest/reference/pep621/#console-scripts)

Create entry point:

```py
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

When you run it:

```console
$ examples-pdm-cli

Hello pdm.
```

## Package management

### `init` Project setup

You will need to answer a few questions, to help PDM to create a pyproject.toml file for you.

```shell
pdm init
```

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

### `venv` Use virtualenv

pdm automatically creates a virtual environment the first time you run `pdm install`.

```shell
pdm install
```

> Virtual environments will be used if the project interpreter (the interpreter stored in `.pdm-python`, which can be checked by `pdm info`) is from a virtualenv.

Virtual environments will be used if the project interpreter (the interpreter stored in .pdm-python, which can be checked by pdm info) is from a virtualenv.

Create a virtualenv yourself:

```shell
pdm venv create 3.12
```

Select virtualenv:

```shell
pdm use -i -f .venv
```

Omitting the argument makes it interactive.

```console
$ pdm use

Please enter the Python interpreter to use
 0. cpython@3.12 (/usr/local/bin/python)
 1. cpython@3.12 (/workspaces/examples-py-packaging/apps/examples-packaging-pdm/.venv/bin/python)
 2. cpython@3.12 (/usr/local/bin/python3.12)
```

List virtualenv:

<!-- /* spell-checker:disable */ -->
```console
$ pdm venv list

Virtualenvs created with this project:

*  in-project: /workspaces/examples-py-packaging/apps/examples-packaging-pdm/.venv
```
<!-- /* spell-checker:enable */ -->

Remove virtualenv

```shell
pdm venv remove in-project
```

### `sync/update/install` Restore packages

There are some similar commands for installing packages pinned to lockfiles, but with some differences.

- `pdm sync` installs packages from lock files.
- `pdm update` updates the lock file and then syncs.
- `pdm install` checks the project files for changes, updates the lock file if necessary, and then syncs.

`pdm install` installs the current project in editable mode.

### `add` Add package

```shell
pdm add {packages...}
```

When adding a development-only dependency:

```shell
pdm add  --dev {packages...}
pdm add  --dev --group {group} {packages...}
```

Editable dependencies:

```shell
pdm add -e {location} --dev
```

### `update` Update package

```shell
pdm update {packages...}
```

### `update` Update outdated packages

show outdated list:

```shell
pdm list --outdated
```

Update all dependency packages:

```shell
pdm update
```

### `remove` Remove package

```shell
pdm remove {packages...}
```

### `list` List packages

```shell
pdm list
```

Or show a dependency tree by:

<!-- /* spell-checker:disable */ -->
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
<!-- /* spell-checker:enable */ -->

### `build` Builds package

```shell
pdm build
```

```console
$ pdm build --no-sdist

Building wheel from sdist...
Built wheel at /workspaces/examples-py-packaging/apps/examples-packaging-pdm/dist/examples_packaging_pdm-0.1.3-py3-none-any.whl
```
<!-- /* spell-checker:words sdist */ -->

## References

- <https://pdm.fming.dev/latest/>
- <https://github.com/pdm-project/pdm>

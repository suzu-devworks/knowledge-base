# Pipenv

Pipenv is a Python virtualenv management tool that supports a multitude of systems and nicely bridges the gaps between pip, pyenv and virtualenv.

It's a combination of pip and venv. conda?  
Can't you make a wheel package?

- [Pipenv](#pipenv)
  - [Installation](#installation)
  - [Creating and Configuring a project](#creating-and-configuring-a-project)
    - [How the project was initialized](#how-the-project-was-initialized)
    - [Install packages](#install-packages)
    - [Custom Script Shortcuts](#custom-script-shortcuts)
    - [Provide console scripts](#provide-console-scripts)
  - [Project management](#project-management)
    - [Project setup and create virtualenv](#project-setup-and-create-virtualenv)
    - [Remove the virtualenv](#remove-the-virtualenv)
    - [Spawns a shell within the virtualenv](#spawns-a-shell-within-the-virtualenv)
    - [`run` Spawns a command installed into the virtualenv](#run-spawns-a-command-installed-into-the-virtualenv)
    - [`install/sync` Restore packages](#installsync-restore-packages)
    - [`install` Add packages](#install-add-packages)
    - [`update` Update package](#update-update-package)
    - [`update` Update outdated packages](#update-update-outdated-packages)
    - [`uninstall` Remove packages](#uninstall-remove-packages)
    - [`clean` Uninstall all packages](#clean-uninstall-all-packages)
    - [`graph` List packages](#graph-list-packages)
    - [Build wheel package](#build-wheel-package)
  - [Tips](#tips)
    - [Where you placed the site-package modules?](#where-you-placed-the-site-package-modules)
    - [Read environment file](#read-environment-file)
  - [References](#references)

## Installation

Install using pip on venv:

```shell
pip install --user pipenv
```

## Creating and Configuring a project

### How the project was initialized

Create and move project folder:

```shell
mkdir -p apps/examples-packaging-pipenv
cd apps/examples-packaging-pipenv
```

Generate a `Pipfile` with the following command:

<!-- /* spell-checker:words pipfile */ -->

```shell
# You need pyenv if you want other python versions.
# A virtual environment is not created because it was executed in venv.
pipenv --python 3.11
export PIPENV_VENV_IN_PROJECT=true
```

Generate the initial directory for the package with the following command:

```shell
mkdir -p apps/examples_packaging_pipenv/
touch apps/examples_packaging_pipenv/__init__.py
```

The final directory will look like this:

```console
.
├── Pipfile
├── Pipfile.lock
├── README.md
└── src
    └── examples_packaging_pipenv
        └── __init__.py
```

### Install packages

Install dependency packages for this project:

```shell
pipenv install --dev flake8 mypy black isort pytest-cov
```

`Pipfile` `Pipfile.lock` is updated.

### Custom Script Shortcuts

`Pipfile`:

```ini
[scripts]
format = "black -l 119 ."
lint = "flake8 --show-source ."
start = "python main.py runserver"
test = "pytest"
```

<!-- /* spell-checker:words runserver */ -->

```console
pipenv run start
```

### Provide console scripts

`pipenv` cannot create packages and therefore cannot convert packages to commands, but you can define Python commands using custom script shortcuts.

Create entry point:

`apps/examples_packaging_pipenv/console/command.py`:

```py
import sys


def main():
    print("Hello pipenv.")


if __name__ == "__main__":
    sys.exit(main())

```

`Pipfile` will look like this:

```ini
[scripts]
start = "python apps/examples_packaging_pipenv/console/command.py"
```

When you run it:

```shell
pipenv run start
```

```console
Courtesy Notice: Pipenv found itself running within a virtual environment, so it will automatically use that environment, instead of creating its own for any project. You can set PIPENV_IGNORE_VIRTUALENVS=1 to force pipenv to ignore that environment and create its own instead. You can set PIPENV_VERBOSITY=-1 to suppress this warning.
Hello pipenv.
```

It's a warning but I don't care.

## Project management

### Project setup and create virtualenv

Generate a Pipfile with the following command:

```shell
pipenv --python {py-version}
```

Enable environment variables if you want a virtual environment in your project folder.

```shell
export PIPENV_VENV_IN_PROJECT=true
```

```shell
pipenv --python 3.8
```

```console
Creating a virtualenv for this project…
Pipfile: /home/xxxxxx/workspaces/databases/test-pipenv/Pipfile
...

✔ Successfully created virtual environment!
Virtualenv location: /home/xxxxxx/.local/share/virtualenvs/test-pipenv-f7l9kGAQ
Creating a Pipfile for this project…
```

If you specify a version that is not installed, it will work with `pyenv`.

```shell
pipenv --python 3.6
```

<!-- /* spell-checker:disable */ -->
```console
Warning: Python 3.6 was not found on your system…
Would you like us to install CPython 3.6.12 with Pyenv? [Y/n]: y

Installing CPython 3.6.12 with /home/xxxxxx/.anyenv/envs/pyenv/bin/pyenv (this may take a few minutes)…
✔ Success!
Downloading Python-3.6.12.tar.xz...
-> https://www.python.org/ftp/python/3.6.12/Python-3.6.12.tar.xz
Installing Python-3.6.12...
Installed Python-3.6.12 to /home/xxxxxx/.anyenv/envs/pyenv/versions/3.6.12

Creating a virtualenv for this project…
Pipfile: /home/xxxxxx/workspaces/databases/test-pipenv/Pipfile
...

✔ Successfully created virtual environment!
Virtualenv location: /home/xxxxxx/.local/share/virtualenvs/test-pipenv-f7l9kGAQ
Creating a Pipfile for this project…
```
<!-- /* spell-checker:enable */ -->

When the pipenv environment is built with the virtual environment enabled:

```shell
pipenv --python 3.11
```

<!-- /* spell-checker:disable */ -->
```console
Courtesy Notice: Pipenv found itself running within a virtual environment, so it will automatically use that environment, instead of creating its own for any project. You can set PIPENV_IGNORE_VIRTUALENVS=1 to force pipenv to ignore that environment and create its own instead. You can set PIPENV_VERBOSITY=-1 to suppress this warning.
Creating a Pipfile for this project...
```
<!-- /* spell-checker:enable */ -->

### Remove the virtualenv

```shell
pipenv --rm
```

```console
Removing virtualenv (/home/xxxxxx/.local/share/virtualenvs/test-pipenv-f7l9kGAQ)…
```

### Spawns a shell within the virtualenv

```shell
pipenv shell
```

```console
Launching subshell in virtual environment…
 . /home/xxxxxx/.local/share/virtualenvs/test-pipenv-f7l9kGAQ/bin/activate

(test-pipenv) $ python --version

Python 3.6.12

(test-pipenv) $ exit
```
<!-- /* spell-checker:words subshell */ -->

### `run` Spawns a command installed into the virtualenv

```shell
pipenv run {command}
```

### `install/sync` Restore packages

from `Pipfile`

```shell
pipenv install
pipenv install --dev
```

from `Pipfile.lock`

```shell
pipenv sync
pipenv sync --dev
```

from `requirements.txt`

```shell
pipenv install -r ./requirements.txt
```

### `install` Add packages

Adding a package makes an entry in the `Pipfile`.

```shell
pipenv install {packages...}
pipenv install --dev {packages...}
```

```console
$ pipenv install numpy

Installing numpy…
Adding numpy to Pipfile's [packages]…
✔ Installation Succeeded
Pipfile.lock not found, creating…
Locking [dev-packages] dependencies…
Locking [packages] dependencies…
Building requirements...
Resolving dependencies...
✔ Success!
Updated Pipfile.lock (2cfc5e)!
Installing dependencies from Pipfile.lock (2cfc5e)…
  🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 0/0 — 00:00:00
```

### `update` Update package

Runs lock, then sync.

```shell
pipenv update {packages...}
```

### `update` Update outdated packages

show outdated list:

```shell
pipenv update --outdated

```

Update all dependency packages:

```shell
pipenv update
```

`Pipfile.lock` is updated.

### `uninstall` Remove packages

Un-installs a provided package and removes it from `Pipfile`.

```shell
pipenv uninstall {packages...}
```

### `clean` Uninstall all packages

Uninstalls all packages not specified in `Pipfile.lock`.

```shell
pipenv clean
```

### `graph` List packages

Tree display of dependencies:

```shell
pipenv graph
```

### Build wheel package

**`pipenv` does not provide package creation. You need to use another tool such as `setuptools`.**

## Tips

### Where you placed the site-package modules?

```shell
pipenv --venv
```

### Read environment file

If you prepare a `.env` file in the project folder, it seems to be automatically loaded when pipenv shell or pipenv run.

Write `.env` file:

```ini
DEBUG=1
```

Try running it:

```console
$ pipenv run python

>>> import os
>>> os.environ['DEBUG']
'1'
```

## References

- <https://pipenv.pypa.io/en/latest/>
- <https://github.com/pypa/pipenv>

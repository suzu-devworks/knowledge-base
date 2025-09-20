# pyenv-virtualenv

pyenv-virtualenv is a pyenv plugin that provides features to manage virtualenv and conda environments for Python on UNIX-like systems.

> [!WARNING]
> _This document is current as of 2019-06-08._

## Table of Contents <!-- omit in toc -->

- [Install](#install)
  - [Clone from GitHub](#clone-from-github)
  - [Using Homebrew on macOS](#using-homebrew-on-macos)
- [Usage](#usage)
  - [Create a new virtualenv](#create-a-new-virtualenv)
  - [Delete a virtualenv](#delete-a-virtualenv)
  - [Switch virtualenv](#switch-virtualenv)
- [References](#references)

## Install

### Clone from GitHub

```shell
git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
```

Reload your shell:

```shell
exec $SHELL -l
pyenv virtualenv --version
# pyenv-virtualenv 1.1.1 (virtualenv unknown)
```

Edit your `~/.bash_profile`:

```shell
export ANYENV_ROOT=${HOME}/.anyenv
if [ -d "${ANYENV_ROOT}" ]; then
    export PATH=${ANYENV_ROOT}/bin:$PATH
    eval "$(anyenv init -)"
    eval "$(pyenv virtualenv-init -)"
fi
```

### Using Homebrew on macOS

```shell
brew install pyenv-virtualenv
```

Edit your `~/.bash_profile`:

```shell
export PYENV_ROOT=${HOME}/.pyenv
if [ -d "${PYENV_ROOT}" ]; then
    export PATH=${PYENV_ROOT}/bin:$PATH
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)"
fi
```

To uninstall:

```shell
brew uninstall pyenv-virtualenv
brew uninstall pyenv

rm -fr ~/.pyenv
find . | grep -E "/\.python-version$" | xargs rm -fr
```

Remove related lines from `~/.bash_profile`.

## Usage

### Create a new virtualenv

```shell
pyenv virtualenv [{base}] {new-virtualenv}
```

Example:

```shell
pyenv virtualenv 3.7.3 3.7.3-pyenv
pyenv virtualenv v372-2
pyenv versions
```

Output:

```console
  system
  3.6.8
* 3.7.3 (set by /home/xxxxxx/.anyenv/envs/pyenv/version)
  3.7.3/envs/3.7.3-pyenv
  3.7.3/envs/v372-2
  3.7.3-pyenv
  v372-2
```

### Delete a virtualenv

Uninstall both the environment and its symbolic link:

```shell
pyenv uninstall {virtualenv}
```

Example:

```shell
pyenv uninstall v372-2
pyenv versions
```

Output:

```console
  system
  3.6.8
* 3.7.3 (set by /home/xxxxxx/.anyenv/envs/pyenv/version)
  3.7.3/envs/3.7.3-pyenv
  3.7.3-pyenv
```

### Switch virtualenv

Activate a virtualenv:

```shell
pyenv activate {virtualenv}
```

Example:

```shell
pyenv activate 3.7.3-pyenv
python --version  # Python 3.7.3

pyenv deactivate
python --version  # Python 3.6.8
```

## References

- [pyenv-virtualenv GitHub Repository](https://github.com/pyenv/pyenv-virtualenv)

# pyenv-virtualenv

pyenv-virtualenv is a pyenv plugin that provides features to manage virtualenv and conda environments for Python on UNIX-like systems.

> [!WARNING]
> _This document is current as of 2019-06-08._

- [pyenv-virtualenv](#pyenv-virtualenv)
  - [Install](#install)
    - [Clone github](#clone-github)
    - [Using Homebrew on macOS](#using-homebrew-on-macos)
  - [Usage](#usage)
    - [Create new virtualenv](#create-new-virtualenv)
    - [Delete virtualenv](#delete-virtualenv)
    - [Switch virtualenv](#switch-virtualenv)
  - [References](#references)

## Install

### Clone github

```shell
git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
```

```console
$ exec $SHELL -l
$ pyenv virtualenv --version

pyenv-virtualenv 1.1.1 (virtualenv unknown)
```

edit `~/.bash_profile`:

```shell
...
export ANYENV_ROOT=${HOME}/.anyenv
if [ -d "${ANYENV_ROOT}" ]; then
    export PATH=${ANYENV_ROOT}/bin:$PATH
    eval "$(anyenv init -)"
    eval "$(pyenv virtualenv-init -)"
fi
```

<!-- /* spell-checker:words anyenv */ -->

### Using Homebrew on macOS

```shell
brew install pyenv-virtualenv
```

edit `~/.bash_profile`:

```shell
...
export PYENV_ROOT=${HOME}/.pyenv
if [ -d "${PYENV_ROOT}" ]; then
    export PATH=${PYENV_ROOT}/bin:$PATH
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)"
fi
```

Un-installation is:

```shell
brew uninstall pyenv-virtualenv
brew uninstall pyenv

rm -fr  ~/.pyenv
find . | grep -E "/\.python-version$" | xargs rm -fr
```

Removed adding `~/.bash_profile`

## Usage

### Create new virtualenv

```shell
pyenv virtualenv [{base}] {new-virtualenv}
```

<!-- /* spell-checker:disable */ -->
```console
$ pyenv virtualenv 3.7.3 3.7.3-pyenv

Looking in links: /tmp/tmp40k9ceag
Requirement already satisfied: setuptools in /home/xxxxxx/.anyenv/envs/pyenv/versions/3.7.3/envs/3.7.3-pyenv/lib/python3.7/site-packages (40.8.0)
Requirement already satisfied: pip in /home/xxxxxx/.anyenv/envs/pyenv/versions/3.7.3/envs/3.7.3-pyenv/lib/python3.7/site-packages (19.0.3)

$ pyenv versions

  system
  3.6.8
* 3.7.3 (set by /home/xxxxxx/.anyenv/envs/pyenv/version)
  3.7.3/envs/3.7.3-pyenv
  3.7.3-pyenv

$ pyenv virtualenv v372-2

Looking in links: /tmp/tmp_o6ncce0
Requirement already satisfied: setuptools in /home/xxxxxx/.anyenv/envs/pyenv/versions/3.7.3/envs/v372-2/lib/python3.7/site-packages (40.8.0)
Requirement already satisfied: pip in /home/xxxxxx/.anyenv/envs/pyenv/versions/3.7.3/envs/v372-2/lib/python3.7/site-packages (19.0.3)

$ pyenv versions

  system
  3.6.8
* 3.7.3 (set by /home/xxxxxx/.anyenv/envs/pyenv/version)
  3.7.3/envs/3.7.3-pyenv
  3.7.3/envs/v372-2
  3.7.3-pyenv
  v372-2
```
<!-- /* spell-checker:enable */ -->

### Delete virtualenv

There are two indications, environment and symbolic link, but normal uninstall removes both.

```shell
pyenv uninstall {virtualenv}
```

```console
$ pyenv versions

  system
  3.6.8
* 3.7.3 (set by /home/xxxxxx/.anyenv/envs/pyenv/version)
  3.7.3/envs/3.7.3-pyenv
  3.7.3/envs/v372-2
  3.7.3-pyenv
  v372-2

$ pyenv uninstall v372-2

pyenv-virtualenv: remove /home/xxxxxx/.anyenv/envs/pyenv/versions/3.7.3/envs/v372-2? y

$ pyenv versions

  system
  3.6.8
* 3.7.3 (set by /home/xxxxxx/.anyenv/envs/pyenv/version)
  3.7.3/envs/3.7.3-pyenv
  3.7.3-pyenv
```

### Switch virtualenv

```shell
pyenv activate {virtualenv}
```

```console
$ pyenv activate 3.7.3-pyenv

pyenv-virtualenv: prompt changing will be removed from future release. configure `export PYENV_VIRTUALENV_DISABLE_PROMPT=1' to simulate the behavior.

(3.7.3-pyenv) $ python --version

Python 3.7.3

(3.7.3-pyenv) $ pyenv deactivate
$ python --version

Python 3.6.8
```

## References

- <https://github.com/pyenv/pyenv-virtualenv>

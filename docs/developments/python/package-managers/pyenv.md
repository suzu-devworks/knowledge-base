# pyenv

pyenv lets you easily switch between multiple versions of Python. It's simple, unobtrusive, and follows the UNIX tradition of single-purpose tools that do one thing well.

> [!WARNING]
> _This document is current as of 2019-06-08._

## Table of Contents <!-- omit in toc -->

- [Install](#install)
  - [Using anyenv](#using-anyenv)
  - [Clone from GitHub](#clone-from-github)
  - [Using Homebrew on macOS](#using-homebrew-on-macos)
- [Usage](#usage)
  - [List available versions](#list-available-versions)
  - [Install Python version](#install-python-version)
  - [Uninstall Python version](#uninstall-python-version)
  - [Switch version (global)](#switch-version-global)
  - [Switch version (local)](#switch-version-local)
- [Tips](#tips)
  - [Where are site-packages located?](#where-are-site-packages-located)
- [Troubleshooting](#troubleshooting)
  - [Suggested build environment on Fedora](#suggested-build-environment-on-fedora)
  - [Ignoring ensure pip failure: pip 7.1.2 requires SSL/TLS on macOS](#ignoring-ensure-pip-failure-pip-712-requires-ssltls-on-macos)
- [References](#references)

## Install

### Using anyenv

```shell
anyenv install pyenv
```

anyenv install is:

```shell
git clone https://github.com/anyenv/anyenv ~/.anyenv

echo 'export PATH="$HOME/.anyenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(anyenv init -)"' >> ~/.bash_profile
exec $SHELL -l
```

- <https://github.com/anyenv/anyenv>

### Clone from GitHub

```shell
git clone https://github.com/pyenv/pyenv.git ~/.pyenv

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bash_profile
exec "$SHELL"
```

### Using Homebrew on macOS

```shell
brew install pyenv
```

Edit `~/.bash_profile`:

```shell
export PYENV_ROOT=${HOME}/.pyenv
if [ -d "${PYENV_ROOT}" ]; then
    export PATH=${PYENV_ROOT}/bin:$PATH
    eval "$(pyenv init -)"
fi
```

To uninstall:

```shell
brew uninstall pyenv

rm -fr ~/.pyenv
find . | grep -E "/\.python-version$" | xargs rm -fr
```

Remove related lines from `~/.bash_profile`.

## Usage

### List available versions

```shell
pyenv install --list
```

```console
Available versions:
  2.1.3
  2.2.3
  ...
  3.6.8
  3.7.0
  3.7-dev
  3.7.1
  3.7.2
  3.7.3
  3.8-dev
  3.9-dev
  ...
  miniconda3-4.3.30
  ...
```

### Install Python version

```shell
pyenv install {version}
```

```console
$ pyenv versions

* system (set by /home/xxxxxx/.anyenv/envs/pyenv/version)

$ pyenv install 3.7.3

Downloading Python-3.7.3.tar.xz...
-> https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tar.xz
Installing Python-3.7.3...
Installed Python-3.7.3 to /home/xxxxxx/.anyenv/envs/pyenv/versions/3.7.3

$ pyenv versions

* system (set by /home/xxxxxx/.anyenv/envs/pyenv/version)
  3.7.3
```

### Uninstall Python version

```shell
pyenv uninstall {version}
```

### Switch version (global)

Version switching is achieved by rewriting the version name (environment name) to `~/.pyenv/version`.

```shell
pyenv global {version}
```

```console
$ pyenv versions

* system (set by /home/xxxxxx/.anyenv/envs/pyenv/version)
  3.6.8
  3.7.3

$ python --version

Python 2.7.16

$ pyenv global 3.7.3

$ python --version

Python 3.7.3

$ pyenv versions

  system
  3.6.8
* 3.7.3 (set by /home/xxxxxx/.anyenv/envs/pyenv/version)
```

### Switch version (local)

Switching is done by writing the version name to `.python-version` in the current directory.

```shell
pyenv local {version}
```

```console
$ python --version

Python 3.7.3

$ mkdir test-dir
$ cd test-dir

$ pyenv local 3.6.8

$ python --version

Python 3.6.8

$ cd ..
$ python --version

Python 3.7.3

$ cd test-dir/
$ python --version

Python 3.6.8

$ pyenv local --unset
$ python --version

Python 3.7.3
```

## Tips

### Where are site-packages located?

Check the location with pip:

```console
$ pip show pip

Name: pip
Version: 18.1
Summary: The PyPA recommended tool for installing Python packages.
Home-page: https://pip.pypa.io/
Author: The pip developers
Author-email: pypa-dev@groups.google.com
License: MIT
Location: /home/xxxxxx/.anyenv/envs/pyenv/versions/3.6.8/lib/python3.6/site-packages
Requires:
Required-by:
```

## Troubleshooting

### Suggested build environment on Fedora

> zipimport.ZipImportError: can't decompress data; zlib not available  
> make: \*\*\* [Makefile:1130: install] error 1

As written on the official site, header files and various tools are required because C is being built.

```shell
dnf install make gcc zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel tk-devel libffi-devel
```

### Ignoring ensure pip failure: pip 7.1.2 requires SSL/TLS on macOS

Apple no longer provides OpenSSL with Xcode, so you'll need to install openssl with Homebrew.  
Even if it is installed, if the build path is not visible, an error will occur.

```shell
brew install openssl

export CPPFLAGS="${CPPFLAGS} -I$(brew --prefix openssl)/include"
export LDFLAGS="${LDFLAGS} -L$(brew --prefix openssl)/lib"
pyenv install 3.5.0
```

## References

- [pyenv GitHub Repository](https://github.com/pyenv/pyenv)

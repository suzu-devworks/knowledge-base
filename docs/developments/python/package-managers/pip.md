# pip

Pip is a package-management system written in Python and is used to install and manage software packages. ([Wikipedia](https://en.wikipedia.org/wiki/Pip_(package_manager)))

## Table of Contents <!-- omit in toc -->

- [Tips](#tips)
  - [Write `requirements.txt`](#write-requirementstxt)
  - [Install the current project for development](#install-the-current-project-for-development)
  - [Build package](#build-package)
- [References](#references)

## Tips

### Write `requirements.txt`

Export installed packages to `requirements.txt`:

```shell
pip freeze > requirements.txt
```

### Install the current project for development

Install the current project in editable mode:

```shell
pip install -e .
```

> `-e, --editable <path/url>` installs the project as editable.

To uninstall:

```shell
pip uninstall <package-name>
```

### Build package

Build a source distribution and wheel:

```shell
python -m build
```

Example output:

```console
(PY3.9.1) % ls -1 dist
hellosetuptools-0.0.1.dev0-py3-none-any.whl
hellosetuptools-0.0.1.dev0.tar.gz
```
<!-- spell-checker:words hellosetuptools -->

## References

- [pip Documentation](https://pip.pypa.io/en/stable/)
- [pip GitHub Repository](https://github.com/pypa/pip)

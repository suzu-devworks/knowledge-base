# venv

The `venv` module supports creating lightweight virtual environments, each with their own independent set of Python packages installed in their site directories.

## Table of Contents <!-- omit in toc -->

- [Usage](#usage)
  - [Create new virtualenv](#create-new-virtualenv)
  - [Delete virtualenv](#delete-virtualenv)
  - [Activate and deactivate virtualenv](#activate-and-deactivate-virtualenv)
- [References](#references)

## Usage

### Create new virtualenv

Create a new virtual environment:

```shell
python -m venv {virtualenv}
```

Example:

```shell
python -m venv .venv
tree .venv
```

Example directory structure:

```console
.venv
├── bin
│   ├── Activate.ps1
│   ├── activate
│   ├── activate.csh
│   ├── activate.fish
│   ├── pip
│   ├── pip3
│   ├── pip3.11
│   ├── python -> /usr/local/bin/python
│   ├── python3 -> python
│   └── python3.11 -> python
├── include
│   └── python3.11
├── lib
│   └── python3.11
│       └── site-packages
...
├── lib64 -> lib
└── pyvenv.cfg

179 directories, 1466 files
```
<!-- /* spell-checker:words pyvenv */ -->

### Delete virtualenv

To delete a virtual environment, simply remove the folder:

```shell
rm -fr {virtualenv}
```

### Activate and deactivate virtualenv

**On macOS and Linux:**

```shell
python --version
# Python 3.6.8

. .venv/bin/activate
python --version
# Python 3.5.0

deactivate
python --version
# Python 3.6.8
```

**On Windows (PowerShell):**

```powershell
.venv\Scripts\Activate.ps1
```

**On Windows (Command Prompt):**

```cmd
.venv\Scripts\activate.bat
```

## References

- [venv — Creation of virtual environments — Python documentation](https://docs.python.org/ja/3/library/venv.html)

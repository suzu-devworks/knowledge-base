# venv

The venv module supports creating lightweight “virtual environments”, each with their own independent set of Python packages installed in their site directories.

- [venv](#venv)
  - [Usage](#usage)
    - [Create new virtualenv](#create-new-virtualenv)
    - [Delete virtualenv](#delete-virtualenv)
    - [Switch virtualenv](#switch-virtualenv)
  - [References](#references)

## Usage

### Create new virtualenv

```shell
python -m venv {virtualenv}
```

```console
$ python -m venv .venv
$ tree .venv

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

The virtualenv goes inside the project folder, so just delete the folder.

```shell
rm -fr {virtualenv}
```

### Switch virtualenv

For macos, linux:

```console
$ python --version

Python 3.6.8

$ . .venv/bin/activate
(.venv) $ python --version

Python 3.5.0

$ deactivate

$ python --version

Python 3.6.8
```

For Windows:

```console
PS> .venv¥Scripts¥Activate.ps1
```

## References

- <https://docs.python.org/ja/3/library/venv.html>

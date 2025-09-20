# mypy

- [Troubleshooting](#troubleshooting)
  - [error: Duplicate module named "tests"](#error-duplicate-module-named-tests)

## Troubleshooting

### error: Duplicate module named "tests"

In workspace configurations for uv or rye, the `tests` folder may be registered as a module name, which can cause problems.

Check with verbose output:

```shell
uv run mypy --config-file=.mypy.ini -v .
```

Example output:

```console
OG:  Mypy Version:           1.14.1
LOG:  Config File:            /workspaces/examples-py-web/.mypy.ini
LOG:  Configured Executable:  /workspaces/examples-py-web/.venv/bin/python
LOG:  Current Executable:     /workspaces/examples-py-web/.venv/bin/python
LOG:  Cache Dir:              .mypy_cache
LOG:  Compiled:               True
LOG:  Exclude:                []
...

LOG:  Found source:           BuildSource(path='./packages/examples-flask-media/tests/__init__.py', module='tests', ...)
LOG:  Found source:           BuildSource(path='./packages/examples-playwright-started/tests/__init__.py', module='tests', ...)
...

packages/examples-playwright-started/tests/__init__.py: error: Duplicate module named "tests" (also at "./packages/examples-flask-media/tests/__init__.py")
packages/examples-playwright-started/tests/__init__.py: note: See https://mypy.readthedocs.io/en/stable/running_mypy.html#mapping-file-paths-to-modules for more info
packages/examples-playwright-started/tests/__init__.py: note: Common resolutions include: a) using `--exclude` to avoid checking one of them, b) adding `__init__.py` somewhere, c) using `--explicit-package-bases` or adjusting MYPYPATH
Found 1 error in 1 file (errors prevented further checking)
```
<!-- spell-checker:words mypypath -->

`tests` are considered as packages.

Example directory structure:

```console
.
├── packages
│   ├── examples-flask-media
│   │   ├── pyproject.toml
│   │   ├── README.md
│   │   ├── src
│   │   │   ├── examples
│   │   │   │   └── __init__.py
│   │   │   └── examples_flask_media
│   │   │       └── __init__.py
│   │   └── tests
│   │       └── __init__.py
│   └── examples-playwright-started
│       ├── pyproject.toml
│       ├── README.md
│       ├── src
│       │   └── examples_playwright_started
│       │       └── __init__.py
│       └── tests
│           └── __init__.py
├── pyproject.toml
├── README.md
└── uv.lock
```

The most common solution is to use the `--explicit-package-bases` option, but this may not resolve the issue.

The fully qualified name of a module is determined by mypy options such as `--no-namespace-packages` and `--explicit-package-bases`, which are not discussed here.

mypy does not allow the same module name in the workspace environment.

Possible solutions:

1. Rename your test directories to unique names, although this is non-standard for Python's src-layout.
2. Do not put `__init__.py` in `tests`; instead, create a folder for the namespace and put `__init__.py` there, similar to `src`.
3. When performing a bulk check, use the `--exclude` option to exclude `tests`.
4. Run mypy from the location of each `pyproject.toml` in your workspace (may be slow).

Currently, using namespace directories is recommended.

Example fix:

```diff
        └── tests
-           └── __init__.py
+           └── examples_flask_media_tests
+               └── __init__.py
```

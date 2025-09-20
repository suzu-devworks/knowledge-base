# mypy

- [mypy](#mypy)
  - [Troubleshooting](#troubleshooting)
    - [error: Duplicate module named "tests"](#error-duplicate-module-named-tests)

## Troubleshooting

### error: Duplicate module named "tests"

In the workspace configuration of uv or rye, the tests folder is sometimes registered as the module name, which can be a problem.

Check with verbose:

```shell
uv run mypy --config-file=.mypy.ini -v .
```

```console
OG:  Mypy Version:           1.14.1
LOG:  Config File:            /workspaces/examples-py-web/.mypy.ini
LOG:  Configured Executable:  /workspaces/examples-py-web/.venv/bin/python
LOG:  Current Executable:     /workspaces/examples-py-web/.venv/bin/python
LOG:  Cache Dir:              .mypy_cache
LOG:  Compiled:               True
LOG:  Exclude:                []
...

LOG:  Found source:           BuildSource(path='./packages/examples-flask-media/tests/__init__.py', module='tests', has_text=False, base_dir='/workspaces/examples-py-web/packages/examples-flask-media', followed=False)
...

LOG:  Found source:           BuildSource(path='./packages/examples-playwright-started/tests/__init__.py', module='tests', has_text=False, base_dir='/workspaces/examples-py-web/packages/examples-playwright-started', followed=False)
...

LOG:  Metadata abandoned for tests: new attributes are missing
LOG:  Metadata not found for tests
LOG:  Using cached AST for ./packages/examples-playwright-started/tests/__init__.py (tests)
LOG:  Build finished in 0.047 seconds with 1 modules, and 3 errors
packages/examples-playwright-started/tests/__init__.py: error: Duplicate module named "tests" (also at "./packages/examples-flask-media/tests/__init__.py")
packages/examples-playwright-started/tests/__init__.py: note: See https://mypy.readthedocs.io/en/stable/running_mypy.html#mapping-file-paths-to-modules for more info
packages/examples-playwright-started/tests/__init__.py: note: Common resolutions include: a) using `--exclude` to avoid checking one of them, b) adding `__init__.py` somewhere, c) using `--explicit-package-bases` or adjusting MYPYPATH
Found 1 error in 1 file (errors prevented further checking)
```
<!-- spell-checker:words mypypath -->

`tests` are considered as packages.

this directory structure should now look like this:

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

The most common solution is to use the --explicit-package-bases option. But that didn't change the outcome.

The fully qualified name of a module is determined by the mypy options --no-namespace-packages and --explicit-package-bases,
which we won't discuss here.

We found that mypy does not allow the same module name in the workspace environment.

Let's consider some possible solutions.

1. Rename your tests to unique names. although this looks odd with the standard Python src-layout.
2. Don't put `__init__.py` in `tests`, instead create a folder for the namespace and put `__init__.py` there, just like `src`.
3. When performing a bulk check, use the `--exclude` option to exclude `tests`.
4. Look for pyproject.toml in your workspace and run mypy in that location, but this is slow.

Currently, it seems appropriate to use namespace directories.

```diff
        └── tests
-           └── __init__.py
+           └── examples_flask_media_tests
+               └── __init__.py
```

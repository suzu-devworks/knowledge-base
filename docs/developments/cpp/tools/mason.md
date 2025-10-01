# Make

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Installation](#installation)
  - [Install from pypi](#install-from-pypi)
  - [How can we use this in a DevContainers?](#how-can-we-use-this-in-a-devcontainers)
- [Nested](#nested)
- [References](#references)

## Overview

Meson is an open source build system meant to be both extremely fast, and, even more importantly, as user friendly as possible.

In the most common case, you will need the Ninja executable for using the ninja backend, which is the default in Meson. This backend can be used on all platforms and with all toolchains, including GCC, Clang, Visual Studio, MinGW, ICC, ARMCC, etc.

<!-- spell-checker: words ARMCC -->

## Installation

Installing Mason is very easy using pip.

### Install from pypi
<!-- spell-checker: words pypi -->

> [!WARNING]
> In Python 3.12 and later versions, PEP 668 is enabled, and attempting to install packages globally
> using `pip3 install` outside of a virtual environment will often result in
> an error message: "error: externally-managed-environment".
> To avoid this, create a virtual environment using `venv`  and run `pip` within that environment.

```shell
python3 -m venv .venv
.venv/bin/python -m pip install --no-cache-dir meson ninja
```
<!-- spell-checker: words venv cmakelang -->

### How can we use this in a DevContainers?

It's the same as with CMake.

```Dockerfile
# PEP 668 compliant package install
RUN set -x \
    && python3 -m venv /opt/python/venv \
    && /opt/python/venv/bin/python -m pip install --no-cache-dir meson ninja \
    && ln -s /opt/python/venv/bin/meson /usr/local/bin/ \
    && ln -s /opt/python/venv/bin/ninja /usr/local/bin/ \
    ;
```

## Nested

I understand there are two options: `subdir` and `subproject`, and I assume that `subproject` represents a more independent module. However, using `subproject` requires a subdirectory, and the fact that you can't use `.` in the subdirectory name feels a bit inconvenient when migrating from other build tools. I think I need to investigate this further.

<!-- spell-checker: words subproject -->

Therefore, this section will explain an example of using the simpler `subdir` option within a single, large project.

I haven't yet looked into resolving dependency issues, I apologize.

root `meson.build` is:

```meson
project('algorithms', 'c', 'cpp',
    version : '1.0.0',
    default_options : ['c_std=c11', 'cpp_std=c++11', 'warning_level=3']
)

subdir('aaa')
subdir('bbb')
```

`aaa/meson.build` is:

```meson
executable('aaa', 'aaa.c')
```

`bbb/meson.build` is:

```meson
executable('bbb', 'bbb.c')
```

## References

- [The Meson Build system](https://mesonbuild.com/index.html)

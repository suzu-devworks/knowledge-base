# CMake

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Installation](#installation)
  - [Install from pypi](#install-from-pypi)
  - [How can we use this in a DevContainers?](#how-can-we-use-this-in-a-devcontainers)
- [Getting Started](#getting-started)
  - [Generate a Project Buildsystem](#generate-a-project-buildsystem)
  - [Build a Project](#build-a-project)
- [`CMakeLists.txt`](#cmakeliststxt)
  - [`cmake_minimum_required`](#cmake_minimum_required)
  - [`project`](#project)
  - [`add_executable`](#add_executable)
  - [`set` - Specifying the C++ Standard](#set---specifying-the-c-standard)
  - [`add_compile_options`](#add_compile_options)
  - [`add_compile_definitions`](#add_compile_definitions)
- [Nested](#nested)
  - [Parent `CMakeLists.txt`](#parent-cmakeliststxt)
  - [`add_subdirectory`](#add_subdirectory)
- [References](#references)

## Overview

CMake is a free, cross-platform, software development tool for building applications via compiler-independent instructions.

## Installation

The version of CMake included in Debian 12 (Bookworm) is a bit outdated; if you need the latest version of CMake, it's best to install it using pip.

```console
$ /usr/bin/cmake --version

cmake version 3.25.1
```

### Install from pypi
<!-- spell-checker: words pypi -->

> [!WARNING]
> In Python 3.12 and later versions, PEP 668 is enabled, and attempting to install packages globally
> using `pip3 install` outside of a virtual environment will often result in
> an error message: "error: externally-managed-environment".
> To avoid this, create a virtual environment using `venv`  and run `pip` within that environment.

```shell
python3 -m venv .venv
.venv/bin/python -m pip install --no-cache-dir cmake cmakelang
```
<!-- spell-checker: words venv cmakelang -->

### How can we use this in a DevContainers?

I'm still against using `--break-system-packages`, so I'll consider another approach.

Searching online brings up options like using `pipx`, but executing it in a `Dockerfile` places the files in `/root` since it references `$HOME/.local/bin`. This means it becomes inaccessible when trying to use it with the `vscode` user in a devcontainer. This feature supposedly has a `--global` option to move the virtual environment to a shared location, but it appears to be a recent addition, and it wasn't available in the version obtainable via `apt`.

I think obtaining it from PyPI is the best option after all. It would be best to create a globally accessible virtual environment, perhaps in `/opt`, and link it so that it can be referenced from `/usr/local/bin`.

```Dockerfile
# PEP 668 compliant package install
RUN set -x \
    && python3 -m venv /opt/python/venv \
    && /opt/python/venv/bin/python -m pip install --no-cache-dir cmake cmakelang \
    && ln -s /opt/python/venv/bin/cmake* /usr/local/bin/ \
    ;
```

## Getting Started

### Generate a Project Buildsystem

Run CMake with one of the following command signatures to specify the source and build trees and generate a buildsystem:

```shell
cmake [<options>] -B <path-to-build> [-S <path-to-source>]
cmake [<options>] <path-to-source>
cmake [<options>] <path-to-source>
```

To build a software project using CMake, you need to generate a project build system. Since CMake generates this build system, it is common practice to create it in a separate folder.

```shell
mkdir build && cd build
cmake ../
```

### Build a Project

## `CMakeLists.txt`

A `CMakeLists.txt` file is a plain text file that contains the project description written in the CMake language.

For simple projects, a file with just 3 commands is often sufficient; it's customary to write the commands in lowercase.

```txt
cmake_minimum_required(VERSION 3.20)
project(Hello)
add_executable(Hello Hello.c)
```

### `cmake_minimum_required`

```txt
cmake_minimum_required(VERSION <min>[...<policy_max>] [FATAL_ERROR])
```

Sets the minimum required version of cmake for a project.

If the running version of CMake is lower than the `<min>` required version it will stop processing the project and report an error.

### `project`

```txt
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [COMPAT_VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])
```

Sets the name of the project, and stores it in the variable `PROJECT_NAME`.

Supported languages are `C`, `CXX` ...

### `add_executable`

```txt
add_executable(<name> <options>... <sources>...)¶
```

Add an executable target called `<name>` to be built from the source files listed in the command invocation.

### `set` - Specifying the C++ Standard

```txt
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

One way to enable support for a specific C++ standard in CMake is by using the `CMAKE_CXX_STANDARD` and `CMAKE_CXX_STANDARD_REQUIRED` variables.

### `add_compile_options`

Adds options to the COMPILE_OPTIONS directory property. These options are used when compiling targets from the current directory and below.

```txt
add_compile_options(-Werror -Wall -Wextra -Wpedantic)
add_compile_options(-g)
```
<!-- spell-checker: words Werror Wextra Wpedantic -->

This command can be used to add any options. However, for adding preprocessor definitions and include directories it is recommended to use the more specific commands `add_compile_definitions()` and `include_directories()`.

### `add_compile_definitions`

Add preprocessor definitions to the compilation of source files.

``txt
add_compile_definitions(-DDEBUG)
``
<!-- spell-checker: words DDEBUG -->

## Nested

### Parent `CMakeLists.txt`

```txt
cmake_minimum_required(VERSION 3.20)
project("algorithms")
add_subdirectory(sorting)
```

### `add_subdirectory`

```txt
add_subdirectory(sorting)
```

Adds a subdirectory to the build. The source_dir specifies the directory in which the source CMakeLists.txt and code files are located.

## References

- [CMake Documentation and Community](https://cmake.org/documentation/)

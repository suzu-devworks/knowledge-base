# Prettier

Prettier is an opinionated code formatter. It enforces a consistent style by parsing your code and re-printing it with its own rules that take the maximum line length into account, wrapping code when necessary.

[![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg?style=flat-square)](https://github.com/prettier/prettier)

- [Prettier](#prettier)
  - [Installation](#installation)
    - [Install package](#install-package)
    - [`.prettierrc`](#prettierrc)
    - [`.prettierignore`](#prettierignore)
  - [Command line](#command-line)
  - [Configuration](#configuration)
    - [Use package.json scripts](#use-packagejson-scripts)
    - [VSCode extension](#vscode-extension)
  - [References](#references)

## Installation

### Install package

```shell
npm install --save-dev --save-exact prettier
```

```shell
yarn add --dev --exact prettier
```

```shell
pnpm add --save-dev --save-exact prettier
```

### `.prettierrc`

```json
{
  "endOfLine": "lf",
  "printWidth": 120,
  "semi": false,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "es5",
  "useTabs": false
}
```

### `.prettierignore`

see <https://prettier.io/docs/ignore>.

```ignore
# .prettierignore

# So by default it will be:
# **/.git
# **/.svn
# **/.hg
# **/node_modules

# Ignore artifacts:
build
coverage

dist/
temp/
LICENSE.md
pnpm-lock.yaml
pnpm-workspace.yaml
```

## Command line

Use the prettier command to run Prettier from the command line.

see <https://prettier.io/docs/cli>.

```shell
npx prettier --write .
```

```shell
yarn exec prettier --write .
```

```shell
pnpm exec prettier --write .
```

## Configuration

### Use package.json scripts

```json
  "scripts": {
    "format": "prettier --write --cache ."
  },
```

### VSCode extension

use [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

Format on save in VSCode.

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,

  "[javascript]": {
    "editor.defaultFormatter": "<another formatter>"
    "editor.formatOnSave": true,
  }
}
<!-- spell-checker:words esbenp -->
```

## References

- <https://prettier.io/>
- <https://github.com/prettier/prettier>

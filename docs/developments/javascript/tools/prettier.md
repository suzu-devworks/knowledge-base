# Prettier

Prettier is an opinionated code formatter. It enforces a consistent style by parsing your code and re-printing it with its own rules, taking the maximum line length into account and wrapping code when necessary.

[![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg?style=flat-square)](https://github.com/prettier/prettier)

## Table of Contents <!-- omit in toc -->

- [Installation](#installation)
  - [Install package](#install-package)
  - [`.prettierrc`](#prettierrc)
  - [`.prettierignore`](#prettierignore)
- [Command Line](#command-line)
- [Configuration](#configuration)
  - [Use package.json scripts](#use-packagejson-scripts)
  - [VSCode Extension](#vscode-extension)
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

Example configuration:

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

See [Prettier ignore documentation](https://prettier.io/docs/ignore).

```ignore
# .prettierignore

# Default ignores:
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

## Command Line

Use the Prettier command to format your code from the command line.

See [Prettier CLI documentation](https://prettier.io/docs/cli).

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

Add a format script to your `package.json`:

```json
"scripts": {
  "format": "prettier --write ."
}
```

### VSCode Extension

Use the [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) extension for Visual Studio Code.

To format on save in VSCode, add the following to your settings:

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true
}
```
<!-- spell-checker: words esbenp -->

To use a different formatter for JavaScript files:

```json
{
  "[javascript]": {
    "editor.defaultFormatter": "<another formatter>",
    "editor.formatOnSave": true
  }
}
```

## References

- [Prettier Official Site](https://prettier.io/)
- [Prettier GitHub Repository](https://github.com/prettier/prettier)

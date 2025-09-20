# stylelint

A powerful CSS linter that helps you avoid errors and enforce conventions.

## Table of Contents <!-- omit in toc -->

- [Installation](#installation)
  - [Install packages](#install-packages)
  - [`stylelint.config.js`](#stylelintconfigjs)
- [Command Line](#command-line)
- [Plugins](#plugins)
  - [Order](#order)
- [Configuration](#configuration)
  - [VSCode Extension](#vscode-extension)
- [References](#references)

## Installation

### Install packages

You can use `npm init stylelint` to launch `create-stylelint`, but the output configuration file will be `.stylelintrc.json`.  
Here, we configure manually.

```shell
npm install --save-dev stylelint stylelint-config-standard-scss
```

```shell
yarn add -D stylelint stylelint-config-standard-scss
```

```shell
pnpm add -D stylelint stylelint-config-standard-scss
```

### `stylelint.config.js`

See [Stylelint configuration guide](https://stylelint.io/user-guide/configure).

```js
/** @type {import('stylelint').Config} */
export default {
  extends: ["stylelint-config-standard-scss"],
}
```

## Command Line

See [Stylelint CLI documentation](https://stylelint.io/user-guide/cli).

```shell
npx stylelint "**/*.css"
```

```shell
yarn dlx stylelint "**/*.css"
```

```shell
pnpm dlx stylelint "**/*.css"
```

## Plugins

For property order, you can use the `stylelint-order` plugin and the `stylelint-config-recess-order` preset.

### Order

```shell
npm install --save-dev stylelint stylelint-order stylelint-config-recess-order
```

```js
/** @type {import('stylelint').Config} */
export default {
  extends: ["stylelint-config-recess-order"],
}
```

## Configuration

### VSCode Extension

Use the [Stylelint extension](https://marketplace.visualstudio.com/items?itemName=stylelint.vscode-stylelint) for Visual Studio Code.

## References

- [Stylelint Official Site](https://stylelint.io/)
- [Stylelint GitHub Repository](https://github.com/stylelint/stylelint)

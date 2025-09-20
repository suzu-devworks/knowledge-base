# stylelint

A mighty CSS linter that helps you avoid errors and enforce conventions.

- [stylelint](#stylelint)
  - [Installation](#installation)
    - [Install packages](#install-packages)
    - [`stylelint.config.js`](#stylelintconfigjs)
  - [Command line](#command-line)
  - [Plugins](#plugins)
    - [Order](#order)
  - [Configuration](#configuration)
    - [VSCode extension](#vscode-extension)
  - [References](#references)

## Installation

### Install packages

You can also launch `create-stylelint` with the npm init command, but the format of the output configuration file seems to be `.stylelintrc.json`, so we will do it manually.

```shell
npm install --save-dev stylelint stylelint-config-standard-scss
```

```shell
yarn add -D stylelint stylelint-config-standard-scss
```

```shell
pnpm add -D  stylelint stylelint-config-standard-scss

```

### `stylelint.config.js`

see <https://stylelint.io/user-guide/configure>.

```js
/** @type {import('stylelint').Config} */
export default {
  extends: ["stylelint-config-standard-scss"],
}
```

## Command line

see <https://stylelint.io/user-guide/cli>

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

I don't have any particular preferences, so recess-order is fine.
More detailed settings can be done with the stylelint-order plugin.

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

### VSCode extension

use [stylelint](https://marketplace.visualstudio.com/items?itemName=stylelint.vscode-stylelint)

## References

- <https://stylelint.io/>
- <https://github.com/stylelint/stylelint>

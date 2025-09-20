# ESLint

ESLint statically analyzes your code to quickly find problems.  
It is built into most text editors and can be run as part of your continuous integration pipeline.

## Table of Contents <!-- omit in toc -->

- [Installation](#installation)
  - [Install and configure packages](#install-and-configure-packages)
  - [`eslint.config.js`](#eslintconfigjs)
- [Command Line](#command-line)
- [Configuration](#configuration)
  - [Use package.json scripts](#use-packagejson-scripts)
  - [VSCode Extension](#vscode-extension)
- [References](#references)

## Installation

### Install and configure packages

You can install and configure ESLint using one of the following commands:

```shell
npm init @eslint/config@latest
```

```shell
yarn create @eslint/config
```

```shell
pnpm create @eslint/config@latest
```

### `eslint.config.js`

See [ESLint configuration documentation](https://eslint.org/docs/latest/use/configure/).

```js
import { defineConfig } from "eslint/config"
import globals from "globals"
import js from "@eslint/js"

export default defineConfig([
  { files: ["**/*.js"], languageOptions: { globals: globals.browser } },
  { files: ["**/*.js"], plugins: { js }, extends: ["js/recommended"] },
])
```

## Command Line

See [ESLint CLI documentation](https://eslint.org/docs/latest/use/command-line-interface).

```shell
npx eslint .
```

```shell
yarn dlx eslint .
```

```shell
pnpm dlx eslint .
```

## Configuration

### Use package.json scripts

Add a lint script to your `package.json`:

```json
"scripts": {
  "lint": "eslint . --cache"
}
```

### VSCode Extension

Use the [ESLint extension](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) for Visual Studio Code.

To disable formatting by ESLint:

```json
{
  "eslint.format.enable": false
}
```

## References

- [ESLint Official Site](https://eslint.org/)
- [ESLint GitHub Repository](https://github.com/eslint/eslint)

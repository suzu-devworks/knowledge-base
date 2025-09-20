# ESLint

ESLint statically analyzes your code to quickly find problems. It is built into most text editors and you can run ESLint as part of your continuous integration pipeline.

- [ESLint](#eslint)
  - [Installation](#installation)
    - [Install and configure packages](#install-and-configure-packages)
    - [`eslint.config.js`](#eslintconfigjs)
  - [Command line](#command-line)
  - [Configuration](#configuration)
    - [Use package.json scripts](#use-packagejson-scripts)
    - [VSCode extension](#vscode-extension)
  - [References](#references)

## Installation

### Install and configure packages

You can install and configure ESLint using this command:

```shell
npm install --save-dev stylelint stylelint-config-standard-scss



npm init @eslint/config@latest
```

```shell
yarn create @eslint/config
```

```shell
pnpm create @eslint/config@latest
```

### `eslint.config.js`

see <https://eslint.org/docs/latest/use/configure/>.

```js
import { defineConfig } from "eslint/config"
import globals from "globals"
import js from "@eslint/js"

export default defineConfig([
  { files: ["**/*.js"], languageOptions: { globals: globals.browser } },
  { files: ["**/*.js"], plugins: { js }, extends: ["js/recommended"] },
])
```

## Command line

see <https://eslint.org/docs/latest/use/command-line-interface>.

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

```json
  "scripts": {
    "lint": "eslint . --cache",
  },
```

### VSCode extension

use [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)

No formatting in ESLint.

```json
{
  "eslint.format.enable": false
}
```

## References

- <https://eslint.org/>
- <https://github.com/eslint/eslint>

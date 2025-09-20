# lint-staged

Run tasks like formatters and linters against staged git files to prevent unwanted code from entering your code base!

## Table of Contents <!-- omit in toc -->

- [Installation](#installation)
  - [Install packages](#install-packages)
  - [`package.json`](#packagejson)
- [Command Line](#command-line)
- [References](#references)

## Installation

Many examples use [husky](https://typicode.github.io/husky/) for git hooks, but here we use [simple-git-hooks](https://github.com/toplenboren/simple-git-hooks) for simplicity and reliability.

### Install packages

```shell
npm install --save-dev lint-staged simple-git-hooks
```

```shell
pnpm add --D lint-staged simple-git-hooks
```

### `package.json`

```json
{
  "simple-git-hooks": {
    "pre-commit": "pnpm run lint-staged"
  },
  "lint-staged": {
    "*": ["prettier --write --cache --ignore-unknown"],
    "packages/**/*.{js,mjs,cjs,ts,mts,cts}": ["eslint --cache --fix"]
  }
}
```

## Command Line

```shell
pnpm run lint-staged
```

## References

- [lint-staged Open Collective](https://opencollective.com/lint-staged)
- [lint-staged GitHub Repository](https://github.com/lint-staged/lint-staged)

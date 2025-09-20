# lint-staged

Run tasks like formatters and linters against staged git files and don't let 💩 slip into your code base!

- [lint-staged](#lint-staged)
  - [Installation](#installation)
    - [Install packages](#install-packages)
    - [`package.json`](#packagejson)
  - [Command line](#command-line)
  - [References](#references)

## Installation

When I search, I find many examples of using it in combination with [husky](https://typicode.github.io/husky/), but there seems to be something sinister about husky, so here I'm combining it with [simple-git-hooks](https://github.com/toplenboren/simple-git-hooks), which I often see when using lint-staged.

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

## Command line

```shell
pnpm run lint-staged
```

## References

- <https://opencollective.com/lint-staged>
- <https://github.com/lint-staged/lint-staged>

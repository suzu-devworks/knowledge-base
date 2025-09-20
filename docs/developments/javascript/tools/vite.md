# Vite

Vite is a blazing fast frontend build tool powering the next generation of web applications.

## Table of Contents <!-- omit in toc -->

- [Multi-Page App](#multi-page-app)
- [Environment Variables](#environment-variables)
- [Use Sass](#use-sass)
- [Use Alias](#use-alias)
- [References](#references)

## Multi-Page App

See [Vite Multi-Page App Guide](https://vite.dev/guide/build.html#multi-page-app).

Example project structure:

```console
├── dist/
├── public/
├── src/
│   ├── pages/
│   │   ├── counters
│   │   │   └── index.html
│   │   └── nested
│   │       └── index.html
│   ├── .env
│   ├── index.html
│   └── vite-end.d.ts
├── package.json
└── vite.config.js
```

Sample `vite.config.js`:

```js
import { extname, resolve } from "node:path"
import { defineConfig } from "vite"
import { glob } from "glob"

const __dirname = import.meta.dirname
const root = resolve(__dirname, "src")

export default defineConfig({
  root,
  publicDir: resolve(__dirname, "public"),
  build: {
    outDir: resolve(__dirname, "dist"),
    emptyOutDir: true,
    rollupOptions: {
      input: {
        index: resolve(root, "index.html"),
        "pages/counters/index": resolve(root, "pages/counters/index.html"),
        "pages/nested/index": resolve(root, "pages/nested/index.html"),
      },
    },
  },
})
```

- Move the `root` to `src`
- Specify `publicDir` and `outDir` since the root is changed
- Use `emptyOutDir` because `outDir` is outside the root

To automatically include all HTML files under `pages`, use glob:

```js
import { glob } from "glob"

const files = await glob("pages/**/*.html", { cwd: root })
const pages = Object.fromEntries(files.map((path) => [path.replace(extname(path), ""), resolve(root, path)]))

export default defineConfig({
  root,
  build: {
    rollupOptions: {
      input: {
        index: resolve(root, "index.html"),
        ...pages,
      },
    },
  },
})
```
<!-- spell-checker: words extname -->

This will treat each HTML file as an entry point.

Build command:

```shell
pnpm run build
```

Example build output:

<!-- spell-checker: disable -->
```console
vite v7.1.1 building for production...
✓ 10 modules transformed.
../dist/index.html                                    0.55 kB │ gzip: 0.33 kB
../dist/pages/counters/index.html                     0.56 kB │ gzip: 0.34 kB
../dist/pages/nested/index.html                       0.90 kB │ gzip: 0.49 kB
../dist/assets/index-By8knhmW.js                      0.43 kB │ gzip: 0.27 kB
../dist/assets/modulepreload-polyfill-B5Qt9EMX.js     0.71 kB │ gzip: 0.40 kB
../dist/assets/pages/nested/index-CQRa76CE.js         0.87 kB │ gzip: 0.43 kB
../dist/assets/pages/counters/index-C8HyB_Um.js       2.39 kB │ gzip: 1.32 kB
✓ built in 90ms
```
<!-- spell-checker: enable -->

## Environment Variables

See [Vite Environment Variables Guide](https://vite.dev/guide/env-and-mode.html).

Environment variable files:

```console
.env                # loaded in all cases
.env.local          # loaded in all cases, ignored by git
.env.[mode]         # only loaded in specified mode
.env.[mode].local   # only loaded in specified mode, ignored by git
```

Vite exposes variables prefixed with `VITE_` under `import.meta.env` as strings.

Example `.env`:

```properties
VITE_SOME_KEY=123
DB_PASSWORD=foobar
```

Usage:

```js
console.log(import.meta.env.VITE_SOME_KEY) // "123"
console.log(import.meta.env.DB_PASSWORD) // undefined
```

HTML constant replacement:

```html
<h1>Vite is running in %MODE%</h1>
<p>Using data from %VITE_API_URL%</p>
```

## Use Sass

See [Vite CSS Preprocessor Options](https://ja.vite.dev/config/shared-options.html#css-preprocessoroptions).

Install Sass:

```shell
pnpm add sass-embedded -D
```

## Use Alias

See [Vite Alias Configuration](https://ja.vite.dev/config/shared-options.html#resolve-alias).

`vite.config.js`:

```js
const __dirname = import.meta.dirname
const root = resolve(__dirname, "src")

export default defineConfig({
  resolve: {
    alias: {
      "@": `${root}`,
    },
  },
})
```

`tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": [
        "src/*"
      ]
    }
  }
}
```

## References

- [Vite Official Site](https://vite.dev/)

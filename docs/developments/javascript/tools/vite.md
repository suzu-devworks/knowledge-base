# Vite

Vite is a blazing fast frontend build tool powering the next generation of web applications.

- [Vite](#vite)
  - [Multi-Page App](#multi-page-app)
  - [Environment Variables](#environment-variables)
  - [Use sass](#use-sass)
  - [Use alias](#use-alias)
  - [Reference](#reference)

## Multi-Page App

- [see ...](https://vite.dev/guide/build.html#multi-page-app)

In the end it will look like this:

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

`vite.config.js`:

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
- Since the `root` will be changed, specify `publicDir`
- Since the `root` will be changed, specify `outDir`
- Since `outDir` will be outside the root, specify `emptyOutDir`

Furthermore, since we don't want to specify inputs individually, we'll enclose \*.html under pages and retrieve it using glob.

```js
import { glob } from "glob"

const files = await glob("pages/**/*.html", { cwd: root })
const pages = Object.fromEntries(files.map((path) => [path.replace(extname(path), ""), resolve(root, path)]))
console.log(pages)

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

Since I only deleted the extname of the key, assets will become a directory, but that's okay.
<!-- spell-checker: words extname -->

```shell
pnpm run build
```

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

- [see ...](https://vite.dev/guide/env-and-mode.html)

```console
.env                # loaded in all cases
.env.local          # loaded in all cases, ignored by git
.env.[mode]         # only loaded in specified mode
.env.[mode].local   # only loaded in specified mode, ignored by git
```

Vite exposes env variables under import.meta.env object as strings automatically.

`.env`:

```properties
VITE_SOME_KEY=123
DB_PASSWORD=foobar
```

Used:

```js
console.log(import.meta.env.VITE_SOME_KEY) // "123"
console.log(import.meta.env.DB_PASSWORD) // undefined
```

HTML Constant Replacement:

```html
<h1>Vite is running in %MODE%</h1>
<p>Using data from %VITE_API_URL%</p>
```

## Use sass

- [see ...](https://ja.vite.dev/config/shared-options.html#css-preprocessoroptions)

```shell
pnpm add sass-embedded -D
```

## Use alias

- [see ...](https://ja.vite.dev/config/shared-options.html#css-preprocessoroptions)

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

```ts
{
  "compilerOptions": {
    /* use alias */
    "baseUrl": ".",
    "paths": {
      "@/*": [
        "src/*"
      ]
    }
  },
}
```

## Reference

- [Vite](https://vite.dev/)

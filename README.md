# @dufeut/xkin

Browser-ready bundles for **Monaco Editor**, **Babel**, **Prettier**, **SASS**, **CSSO**, **Terser**, **Showdown** and **Preact**, all behind one global `Xkin` API. You don't need a bundler; add them with `<script>` tags.

[![npm](https://img.shields.io/npm/v/@dufeut/xkin)](https://www.npmjs.com/package/@dufeut/xkin)
[![license](https://img.shields.io/npm/l/@dufeut/xkin)](LICENSE)

## Install

```bash
npm install @dufeut/xkin
# or
pnpm add @dufeut/xkin
```

The package is published under the `@dufeut` scope. Install `@dufeut/xkin`, not `xkin`.

## Quick start

### From a CDN

```html
<script src="https://cdn.jsdelivr.net/npm/@dufeut/xkin/dist/xkin.editor.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@dufeut/xkin/dist/xkin.tools.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@dufeut/xkin/dist/xkin.styles.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@dufeut/xkin/dist/xkin.engine.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@dufeut/xkin/dist/xkin.min.js"></script>
```

Use `https://unpkg.com/@dufeut/xkin/dist/...` if you prefer unpkg. In production, pin a version, for example `@dufeut/xkin@0.1.0`.

### From `node_modules`

Serve or copy the package's `dist/` folder and reference it:

```html
<script src="/node_modules/@dufeut/xkin/dist/xkin.editor.min.js"></script>
<script src="/node_modules/@dufeut/xkin/dist/xkin.tools.min.js"></script>
<script src="/node_modules/@dufeut/xkin/dist/xkin.styles.min.js"></script>
<script src="/node_modules/@dufeut/xkin/dist/xkin.engine.min.js"></script>
<script src="/node_modules/@dufeut/xkin/dist/xkin.min.js"></script>
```

> Keep `dist/editor/` next to `xkin.editor.min.js`. Monaco loads its language workers and chunks from that folder at runtime.

### Hello world

```html
<div id="editor" style="height: 400px"></div>
<script>
  const editor = Xkin.editor({
    element: document.getElementById("editor"),
    value: "const App = () => <h1>Hello</h1>;",
    language: "typescript",
  });
</script>
```

## Bundles

Each bundle is independent. Load only the ones you need, but load `xkin.min.js` **last**, because it wraps the others.

| File                 | Global       | Provides                                 | Needed for                                       |
| -------------------- | ------------ | ---------------------------------------- | ------------------------------------------------ |
| `xkin.editor.min.js` | `XkinEditor` | Monaco Editor (+ `dist/editor/` workers) | `editor`, `set_*`, models, types, `set_compiler` |
| `xkin.tools.min.js`  | `XkinTools`  | Babel, Prettier, Terser, Showdown        | `tsx`, `format`, `markdown`, `mdx`               |
| `xkin.styles.min.js` | `XkinStyles` | SASS, CSSO, PostCSS (CSS Modules)        | `sass`, `css_modules`                            |
| `xkin.engine.min.js` | `XkinEngine` | Preact + `preact-render-to-string`       | `engine`, `render_mdx`                           |
| `xkin.min.js`        | `Xkin`       | Unified API + Nanostores                 | Always                                           |
| `xkin.d.ts`          | —            | Type declarations for the `Xkin` global  | Editor autocompletion (optional)                 |

Monaco ships with these languages: `json`, `html`, `css`, `scss`, `javascript`, `typescript`, `python`, `sql`, `graphql`, `markdown`, `yaml`.

---

## API reference

All methods use `snake_case`. Methods marked _async_ return a `Promise`.

| Area                  | Methods                                              |
| --------------------- | ---------------------------------------------------- |
| [Editor](#editor)     | `editor`, `set_theme`, `set_language`, `set_content` |
| [Models](#models)     | `create_model`, `get_model`, `delete_model`          |
| [Types](#types)       | `add_types`, `set_types`, `get_types`, `$types`      |
| [Compiler](#compiler) | `set_compiler`                                       |
| [Tools](#tools)       | `tsx`, `format`, `markdown`, `mdx`                   |
| [Styles](#styles)     | `sass`, `css_modules`                                |
| [Engine](#engine)     | `engine`, `render_mdx`                               |
| [Store](#store)       | `store`                                              |

### Editor

`Xkin.editor(options)` creates a Monaco editor and returns the Monaco editor instance. JSX/TSX is enabled out of the box, with `h`/`Fragment` as the pragma.

```js
const editor = Xkin.editor({
  element: document.getElementById("container"),
  value: "console.log('hello');",
  language: "typescript",
  theme: "vs-dark",
  read_only: false,
  minimap: false,
  font_size: 14,
});
```

| Option          | Type          | Default        | Description                                     |
| --------------- | ------------- | -------------- | ----------------------------------------------- |
| `element`       | `HTMLElement` | —              | Container element (required)                    |
| `value`         | `string`      | `""`           | Initial content                                 |
| `language`      | `string`      | `"javascript"` | Monaco language id                              |
| `theme`         | `string`      | `"vs-dark"`    | `"vs"`, `"vs-dark"`, `"hc-black"`, `"hc-light"` |
| `read_only`     | `boolean`     | `false`        | Make the editor read-only                       |
| `minimap`       | `boolean`     | `false`        | Show the minimap                                |
| `scroll_beyond` | `boolean`     | `false`        | Allow scrolling past the last line              |
| `font_size`     | `number`      | `14`           | Font size in px                                 |
| `auto_layout`   | `boolean`     | `true`         | Resize automatically with the container         |
| `...rest`       | —             | —              | Passed through to `monaco.editor.create`        |

```js
Xkin.set_theme("vs"); // global theme
Xkin.set_language(editor.getModel(), "javascript"); // change model language
```

#### set_content

Replaces the editor content **without losing undo history**. Use it after formatting, for example:

```js
const formatted = await Xkin.format({ source: editor.getValue() });
Xkin.set_content(editor, formatted);
// Ctrl+Z still works
```

### Models

Low-level Monaco models addressed by virtual file paths (`/lib/utils.ts` → `file:///lib/utils.ts`). Models let files import each other inside the editor.

```js
Xkin.create_model(
  "/lib/utils.ts",
  "export const add = (a: number, b: number) => a + b;",
);
Xkin.create_model("/data.json", "{}", "json"); // optional language (default "typescript")
Xkin.get_model("/lib/utils.ts"); // model or null
Xkin.delete_model("/lib/utils.ts");
```

If a model already exists at that path, `create_model` updates its content instead of creating a duplicate.

### Types

Inject global `.d.ts` declarations into Monaco's TypeScript/JavaScript language service. The type list is reactive (a Nanostores atom).

```js
Xkin.add_types([
  { path: "globals.d.ts", content: "declare const $router: Router;" },
]);                     // merge by path

Xkin.set_types([...]);  // replace all
Xkin.get_types();       // read current list

Xkin.$types.subscribe((libs) => console.log("Types:", libs.length));
```

#### Autocompletion for the `Xkin` API

The package includes a self-contained `xkin.d.ts`. Inject it so the editor offers autocompletion for `Xkin` itself:

```js
const url = "https://cdn.jsdelivr.net/npm/@dufeut/xkin/dist/xkin.d.ts";
const types = await fetch(url).then((r) => r.text());
Xkin.add_types([{ path: "xkin.d.ts", content: types }]);
```

### Compiler

`Xkin.set_compiler(options)` configures the TypeScript compiler options for both TS and JS. Enum options accept readable strings or Monaco's numeric values.

```js
Xkin.set_compiler({
  jsx: "React",
  jsxFactory: "h",
  jsxFragmentFactory: "Fragment",
  target: "ESNext",
  module: "ESNext",
  moduleResolution: "NodeJs",
});
```

| Option             | Accepted strings                                                                   |
| ------------------ | ---------------------------------------------------------------------------------- |
| `jsx`              | `"None"`, `"Preserve"`, `"React"`, `"ReactNative"`, `"ReactJSX"`, `"ReactJSXDev"`  |
| `target`           | `"ES3"`, `"ES5"`, `"ES2015"` – `"ES2022"`, `"ESNext"`                              |
| `module`           | `"None"`, `"CommonJS"`, `"AMD"`, `"UMD"`, `"System"`, `"ES2015"`, `"ESNext"`, etc. |
| `moduleResolution` | `"Classic"`, `"NodeJs"`, `"Node16"`, `"NodeNext"`, `"Bundler"`                     |

> `set_compiler` replaces the compiler options. It does not merge them. Include every option you need.

---

### Tools

#### tsx _(async)_

Transforms TypeScript/JSX to JavaScript with Babel, using the `h`/`Fragment` pragma. Terser can minify the output.

```js
const { code } = await Xkin.tsx({
  source: "const App = () => <div>Hello</div>;",
  compress: true, // default false
  mangle: true, // default false
});
```

#### format _(async)_

Formats code with Prettier.

```js
const formatted = await Xkin.format({
  source: "const x=1;const y=2;",
  parser: "babel", // default
  tabWidth: 2,
  printWidth: 80,
  semi: true,
  singleQuote: false,
  useTabs: false,
});
```

<details>
<summary>Available Prettier parsers</summary>

| Parser           | Languages              |
| ---------------- | ---------------------- |
| `babel`          | JavaScript, JSX        |
| `babel-ts`       | TypeScript, TSX        |
| `typescript`     | TypeScript (native)    |
| `css`            | CSS                    |
| `scss`           | SCSS                   |
| `less`           | Less                   |
| `html`           | HTML                   |
| `vue`            | Vue SFC                |
| `angular`        | Angular templates      |
| `markdown`       | Markdown               |
| `mdx`            | MDX                    |
| `graphql`        | GraphQL                |
| `yaml`           | YAML                   |
| `json`           | JSON                   |
| `json-stringify` | JSON (stringify style) |

</details>

#### markdown

Converts Markdown to HTML with Showdown. This call is synchronous. `options` are passed straight to [Showdown's options](https://github.com/showdownjs/showdown#valid-options).

```js
const html = Xkin.markdown({
  source: "# Hello\n\nThis is **bold** text.",
  options: {
    tables: true,
    tasklists: true,
    strikethrough: true,
    ghCodeBlocks: true,
    simplifiedAutoLink: true,
    openLinksInNewWindow: true,
    emoji: true,
  },
});
```

#### mdx _(async)_

Compiles Markdown with embedded `ui-*` components into a JSON-serializable tree. Pair it with [`render_mdx`](#render_mdx) to render the tree to HTML with your own components.

```js
const { tree, symbols } = await Xkin.mdx({
  source: `# Profile

<ui-card title="Hello" />

Some **markdown** text.`,
  md: { tables: true }, // Showdown options
});

// symbols => ["card"]                 (ui-* tags used, without the prefix)
// tree    => { tag, props, children } (plain objects, safe to JSON.stringify)
```

- Custom components must use the `ui-` prefix (`<ui-button />`, `<ui-card>…</ui-card>`).
- `symbols` lists each component name used, without the `ui-` prefix. Use it to load only the components a document needs.
- The tree is plain data, so you can cache it, store it, or send it over the network.

---

### Styles

#### sass _(async)_

Compiles SCSS to CSS. Set `compressed: true` to minify the result with CSSO.

```js
const { css } = await Xkin.sass({
  source: "$color: red; .box { color: $color; }",
  compressed: true,
});
```

#### css*modules *(async)\_

Scopes CSS class names and accepts SCSS input. Names are built as `namespace__class__hash` with an FNV-1a hash.

```js
const { css, tokens } = await Xkin.css_modules({
  source: "$color: red; .title { color: $color; }",
  namespace: "app", // optional prefix
  idSize: 8, // hash length, default 8
});
// tokens => { title: "app__title__a1b2c3d4" }
```

---

### Engine

#### engine

Gives direct access to the Preact runtime.

```js
const { h, Fragment, render, createElement, renderToString } = Xkin.engine;

render(h("h1", null, "Hello"), document.body);
```

#### render_mdx

Renders a tree from [`mdx`](#mdx-async) to an HTML string. Map each `ui-*` tag to a Preact component, keyed by the **full tag name**:

```js
const { h } = Xkin.engine;

const { tree } = await Xkin.mdx({
  source: '# Hi\n\n<ui-card title="Hello" />',
});

const html = Xkin.render_mdx(tree, {
  "ui-card": ({ title }) => h("div", { class: "card" }, title),
});
// => "<div><h1 id=\"hi\">Hi</h1>…<div class=\"card\">Hello</div>…</div>"
```

Tags with no mapping render as plain elements.

---

### Store

Re-exports [Nanostores](https://github.com/nanostores/nanostores) for reactive state.

```js
const { atom, computed, map } = Xkin.store;

const $count = atom(0);
const $double = computed($count, (n) => n * 2);

$double.subscribe((v) => console.log(v));
$count.set(2); // logs 4
```

---

## Development

```bash
pnpm install
pnpm build        # builds every bundle into dist/
pnpm test
```

To try every feature, open `index.html` in a browser after building. It is the interactive playground.

Individual bundles: `build:editor`, `build:tools`, `build:styles`, `build:engine`, `build:main`.

## License

[MIT](LICENSE)

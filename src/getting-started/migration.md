---
layout: layout.njk
title: 迁移
eleventyNavigation:
  key: getting-started-migration
  title: 🚚 迁移
  order: 5
---

在大多数情况下，Parcel 2 的工作方式与 Parcel 1 非常相似，但升级时需要更改一些内容。

## 开始

让我们通过几个基本步骤从 Parcel 1 升级到 Parcel 2。

### 包名

从 Parcel 1 升级到 Parcel 2 时首先要注意的是 npm 包名称已从 `parcel-bundler` 更改为 `parcel`. 您需要相应地更新您的依赖 `package.json` 项。

{% migration %}
{% samplefile "package.json" %}

```json/2
{
  "devDependencies": {
    "parcel-bundler": "^1.12.5"
  }
}
```

{% endsamplefile %}
{% samplefile "package.json" %}

```json/2
{
  "devDependencies": {
    "parcel": "^2.0.0"
  }
}
```

{% endsamplefile %}
{% endmigration %}

您也可以使用包管理器来执行此操作，例如 `npm` 或 `yarn`.

```shell
yarn remove parcel-bundler
yarn add parcel --dev
```

### 缓存位置

Parcel 缓存的默认位置也从 `.cache` 更改为 `.parcel-cache`. 您需要修改您的 `.gitignore` 或类似的帐户以解决此问题：

{% migration %}
{% samplefile ".gitignore" %}

```text/0
.cache
```

{% endsamplefile %}
{% samplefile ".gitignore" %}

```text/0
.parcel-cache
```

{% endsamplefile %}
{% endmigration %}

## 代码更改

### `<script type="module">`

在 Parcel 1 中，从 HTML 文件中的标签引用的 JavaScript `<script>` 文件被视为模块，支持 ES 模块和 CommonJS 语法来导入和导出值。但是，这与浏览器的实际工作方式不匹配，其中“classic scripts”不支持导入和导出，并且顶级变量被视为全局变量。

Parcel 2 匹配浏览器行为经典`<script>` 标签不支持导入或导出。使用 `<script type="module">` 元素来引用模块。这也将自动 `nomodule` 为旧浏览器生成一个版本，具体取决于您的 `browserslist`。有关详细信息，请参阅 [Differential bundling](/features/targets/#differential-bundling) 。

{% migration %}
{% samplefile "index.html" %}

```html/3
<!doctype html>
<html>
  <head>
    <script src="app.js"></script>
  </head>
</html>
```

{% endsamplefile %}

{% samplefile "index.html" %}

```html/3
<!doctype html>
<html>
  <head>
    <script type="module" src="app.js"></script>
  </head>
</html>
```

{% endsamplefile %}
{% endmigration %}

{% error %}

**注意**: 添加 `type="module"` 属性也会影响脚本的加载行为。经典脚本是“渲染阻塞”，这意味着在脚本执行完成之前不会解析 HTML 文档的其余部分。模块脚本不会阻塞渲染，并且会延迟执行，直到 HTML 被完全解析。Parcel 会自动插入 `defer` 属性以匹配旧浏览器和开发模式中的此行为。这意味着像 `document.write` 这样的功能在模块脚本中不起作用。如果您依赖这些功能，请迁移到现代 API 或继续为您的应用程序的该部分使用经典脚本。 请阅读 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#other_differences_between_modules_and_standard_scripts) 上的文档以了解有关模块和经典脚本之间差异的更多信息。
{% enderror %}

有关经典脚本与模块脚本的更多详细信息，请参 [Classic scripts](/languages/javascript/#classic-scripts)。

### 从 JavaScript 导入非代码资源

在 Parcel 1 中，导入任何非 JavaScript 文件（例如图像或视频）都会生成一个 URL。在 Parcel 2 中，这仍然适用于图像等已知文件类型，但其他没有默认支持的文件类型将需要更改代码。

在 JavaScript 中引用 URL 的首选方法是使用 [URL 构造函数](/languages/javascript/#url-dependencies). 但是，您也可以选择在 `import` 语句中为依赖说明符添加前缀 `url:`。

{% migration %}
{% samplefile "index.js" %}

```js/0
import downloadUrl from "./download.zip";

document.body.innerHTML = `<a href="${downloadUrl}">Download</a>`;
```

{% endsamplefile %}
{% samplefile %}

```js/0
const downloadUrl = new URL('download.zip', import.meta.url);

document.body.innerHTML = `<a href="${downloadUrl}">Download</a>`;
```

{% endsamplefile %}
{% endmigration %}

或者，您可以使用自定义 `.parcelrc` 来选择旧行为。将 `@parcel/transformer-raw` 插件全局注册来用于您需要的扩展。

{% sample %}
{% samplefile %}

```shell
yarn add @parcel/config-default @parcel/transformer-raw --dev
```

{% endsamplefile %}
{% samplefile ".parcelrc" %}

```json/3
{
  "extends": "@parcel/config-default",
  "transformers": {
    "*.{zip,tgz}": ["@parcel/transformer-raw"]
  }
}
```

{% endsamplefile %}
{% endsample %}

### 转译

Parcel 1 自动转译您的 JavaScript 以支持一组默认浏览器。Parcel 2 默认不再进行任何转译。这意味着如果您在源代码中使用现代 JavaScript 语法，这就是 Parcel 将输出的内容。要启用转译，请在 package.json 中设置 `browserslist` 字段以定义支持的浏览器目标。

{% sample %}
{% samplefile "package.json" %}

```js/2
{
  "name": "my-project",
  "browserslist": "> 0.5%, last 2 versions, not dead",
  "scripts": {
    "start": "parcel index.html",
    "build": "parcel build index.html"
  },
  "devDependencies": {
    "parcel": "latest"
  }
}
```

{% endsamplefile %}
{% endsample %}

### Babel

与 Parcel 1 一样，Parcel 2 会自动检测 `.babelrc` 和其他 Babel 配置文件。但是，如果您只使用 `@babel/preset-env`, `@babel/preset-typescript`, 和 `@babel/preset-react`, 则可能不再需要 Babel。Parcel 无需 Babel 配置即可自动支持所有这些功能，并且 Parcel 的默认转译器比 Babel 快得多。

如果您只使用上述预设，您可以完全删除您的 Babel 配置。这将使用 Parcel 的默认转译器，这将显着提高您的构建性能。确保在您的配置 `package.json` 中存在 `browserslist` 来匹配以前使用 `@babel/preset-env` 需要转译后的目标。

如果你的 Babel 配置中有自定义预设或插件，你可以保留这些但删除上面列出的预设。这也应该会提高性能（尽管会少一点）。有关更多详细信息，请参阅 JavaScript 文档中的[Babel](/languages/javascript/#babel)。

在此示例中，`.babelrc` 仅包含 `@babel/preset-env` and `@babel/preset-react`，因此可以将其删除并用 `package.json` 中的`browserslist` 进行替换.

{% migration %}
{% samplefile ".babelrc" %}

```json/0-7
{
  "presets": [
    ["@babel/preset-env", {
      "targets": "> 0.25%, not dead"
    }],
    "@babel/preset-react"
  ]
}
```

{% endsamplefile %}
{% samplefile "package.json" %}

```json/1
{
  "browserslist": "> 0.25%, not dead"
}
```

{% endsamplefile %}
{% endmigration %}

### Typescript

Parcel 1 使用 `tsc`（官方 TypeScript 编译器）转译了 TypeScript。Parcel 2 现在改用 [SWC](https://swc.rs) ，这显着提高了转译性能。

但是，默认转译器对 `tsconfig.json` 支持有限制。 如果您使用 JSX 相关选项和其他的自定义编译器选项 `experimentalDecorators`，则可以使用 TSC 替换 Parcel 的默认 TypeScript 转换器 `@parcel/transformer-typescript-tsc`。为此，请安装默认配置和 TSC 插件，并在项目的根目录创建一个 `.parcelrc` 文件。

{% sample %}
{% samplefile %}

```shell
yarn add @parcel/config-default @parcel/transformer-typescript-tsc --dev
```

{% endsamplefile %}
{% samplefile ".parcelrc" %}

```json/3
{
  "extends": "@parcel/config-default",
  "transformers": {
    "*.{ts,tsx}": ["@parcel/transformer-typescript-tsc"]
  }
}
```

{% endsamplefile %}
{% endsample %}

有关将 TypeScript 与 Parcel 一起使用的更多信息，请参阅 [TypeScript 文档](/languages/typescript)。

### flow

就像 Parcel 1 一样，Parcel 2 在 `flow-bin` 安装时会自动支持类型检查。目前是使用 `@babel/preset-flow`实现的。 如果您的 Babel 配置中只有该预设，则可以[如上所述](#babel)将其删除。

与 Parcel 1 不同，您的 Babel 配置会覆盖 Parcel 2 中的默认值，而不是被合并到其中。如果你有 Flow 以外的自定义 Babel 插件，你也需要添加 `@babel/preset-flow`。

### 导入 GraphQL

导入 GraphQL 文件(`.gql`)时，导入仍会被解析/内联（使用 `graphql-import-macro`），但您现在将处理后的 GraphQL 查询作为字符串而不是 Apollo AST 获得。

{% migration %}
{% samplefile "DataComponent.js" %}

```js
import fetchDataQuery from "./fetchData.gql"; // fetchDataQuery is the parsed AST

const DataComponent = () => {
  const { data } = useQuery(fetchDataQuery, {
    fetchPolicy: "cache-and-network",
  });

  // ...
};
```

{% endsamplefile %}
{% samplefile "DataComponent.js" %}

```js/0,4
import gql from "graphql-tag";
import fetchDataQuery from "./fetchData.gql"; // fetchDataQuery is a string

// Convert to the Apollo Specific Query AST
const parsedFetchDataQuery = gql(fetchDataQuery);

const DataComponent = () => {
  const { data } = useQuery(parsedFetchDataQuery, {
    fetchPolicy: "cache-and-network",
  });

  // ...
};
```

{% endsamplefile %}
{% endmigration %}

{% note %}

使用 Parcel 2 的新插件架构，创建一个在构建时将字符串解析为 AST 的插件（就像 Parcel 1 所做的那样）非常容易。有关详细信息，请参阅 [Transformer](/plugin-system/transformer/) 文档。

{% endnote %}

### `package.json#main`

许多 `package.json` 文件（例如由 生成的文件 `npm init`）包含一个 `main` 字段，大多数工具（对于非库项目）都会忽略该字段。但是，当 Parcel 看到字段 `main` 时，会推断您的项目是一个库并将其用作输出路径。对于大多数 Web 应用程序，应删除此行。

{% migration %}
{% samplefile "package.json" %}

```json/1
{
  "main": "index.js",
  "scripts": {
    "start": "parcel index.html",
    "build": "parcel build index.html"
  }
}
```

{% endsamplefile %}
{% samplefile "package.json" %}

```json
{
  "scripts": {
    "start": "parcel index.html",
    "build": "parcel build index.html"
  }
}
```

{% endsamplefile %}
{% endmigration %}

如果您确实需要保留 `main` 字段，并且希望 Parcel 忽略它，您可以添加 `"targets": { "main": false }` 到您的 `package.json`中。有关详细信息，请参阅[Library targets](/features/targets/#library-targets)。

## CLI

### `--target`

在 Parcel 1 中， `--target` CLI 选项控制编译代码的环境。在 Parcel 2 中，这是在 `package.json` 中配置的。例如，将 `engines` 字段设置为包含 `node` 或者 `electron` 键将相应地更改目标。

{% migration %}
{% samplefile %}

```bash
parcel build index.js --target node
```

{% endsamplefile %}
{% samplefile "package.json" %}

```json5/2
{
  "engines": {
    "node": "10"
  }
}
```

{% endsamplefile %}
{% endmigration %}

您还可以在 Parcel 2 中同时构建多个目标。有关详细信息，请参阅 [目标](/features/targets/)。

### `--experimental-scope-hoisting`

Parcel 2 默认启用 scope hoisting。要禁用它，请添加 `--no-scope-hoist`.

{% migration %}
{% samplefile %}

```bash
parcel build index.js --experimental-scope-hoisting
parcel build index.js
```

{% endsamplefile %}
{% samplefile %}

```bash
parcel build index.js
parcel build index.js --no-scope-hoist
```

{% endsamplefile %}
{% endmigration %}

### `--bundle-node-modules`

`node_modules` 要在以 Node.js 为目标时作为捆绑包，您现在应该在目标配置中指定：

{% migration %}
{% samplefile %}

```bash
parcel build index.js --target node --bundle-node-modules
```

{% endsamplefile %}
{% samplefile "package.json" %}

```json5/3,7
{
  "targets": {
    "default": {
      "includeNodeModules": true
    }
  },
  "engines": {
    "node": "10"
  }
}
```

{% endsamplefile %}
{% endmigration %}

{% note %}

此选项比 CLI 参数更通用。例如，您还可以选择性地包含包。有关详细信息，请参阅目标文档中的[includeNodeModules](/features/targets/#includenodemodules)。

{% endnote %}

### `--out-dir`

`--out-dir` CLI 选项已重命名为以 `--dist-dir` 或者匹配 `package.json` 中的`distDir`。有关详细信息，请参阅[目标](/features/targets/#targets)。

{% migration %}
{% samplefile %}

```bash
parcel build index.html --out-dir www
```

{% endsamplefile %}
{% samplefile %}

```bash
parcel build index.html --dist-dir www
```

{% endsamplefile %}
{% endmigration %}

### `--out-file`

`--out-file` CLI 选项已被移除, 后续需要在 `package.json`中配置改路径。 有关详细信息，请参阅 [Multiple targets](/features/targets/#multiple-targets) 和 [Library targets](/features/targets/#library-targets)

{% migration %}
{% samplefile %}

```bash
parcel build index.js --out-file lib.js
```

{% endsamplefile %}
{% samplefile "package.json" %}

```json5/3
{
  "name": "my-library",
  "version": "1.0.0",
  "main": "lib.js"
}
```

{% endsamplefile %}
{% endmigration %}

### `--log-level`

日志级别现在有名称而不是数字 (`none`, `error`, `warn`, `info`, `verbose`)。

{% migration %}
{% samplefile %}

```bash
parcel build index.js --log-level 1
```

{% endsamplefile %}
{% samplefile %}

```bash
parcel build index.js --log-level error
```

{% endsamplefile %}
{% endmigration %}

### `--global`

此选项已被删除，无需替换（目前）。

{% migration %}
{% samplefile %}

```bash
parcel build index.js --global mylib
```

{% endsamplefile %}
{% samplefile %}
{% endsamplefile %}
{% endmigration %}

### `--no-minify`

此选项已重命名为 `--no-optimize`。

{% migration %}
{% samplefile %}

```bash
parcel build index.js --no-minify
```

{% endsamplefile %}
{% samplefile %}

```bash
parcel build index.js --no-optimize
```

{% endsamplefile %}
{% endmigration %}

## API

Parcel 2 可以通过 `@parcel/core` 包以编程方式使用 API，而不是 `parcel-bundler`. API 发生了显着变化。有关详细信息，请参阅[Parcel API](/features/parcel-api/)。

### 绑定事件

Parcel 1 让您可以使用 API 绑定并监听类似 `buildEnd` 或 `buildError`事件。但是 API 已更改，但您仍然可以侦听如下事件：

{% migration %}
{% samplefile "index.js" %}

```js/0
import Bundler from "parcel-bundler"

const bundler = new Bundler({ /* ... */ })
bundler.bundle()

bundler.on("buildEnd", () => { /* ... */ })
bundler.on("buildError", (error) => { /* ... */ })
```

{% endsamplefile %}
{% samplefile %}

```js/0
import Parcel from "@parcel/core"

const bundler = new Parcel({ /* ... */ })

bundler.watch((err, buildEvent) => {
  if (buildEvent.type === "buildSuccess") { /* ... */ }
  if (buildEvent.type === "buildFailure") { /* ... */ }
})
```

{% endsamplefile %}
{% endmigration %}

## 插件

Parcel 2 中的插件系统已完全更改。Parcel 1 插件与 Parcel 2 不兼容。有关 Parcel 2 插件 API 的详细信息，请参阅[插件系统](/plugin-system/overview/)。

### 使用插件

在 Parcel 1 中，将插件安装到您的项目中，并且 `package.json` 在依赖项中列出它们会自动启用它们。在 Parcel 2 中，插件配置在 `.parcelrc`. 有关详细信息，请参阅[Parcel 插件](/features/plugins/)。

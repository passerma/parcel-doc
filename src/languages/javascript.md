---
layout: layout.njk
title: JavaScript
eleventyNavigation:
  key: languages-js
  title: <img src="/assets/lang-icons/javascript.svg" alt=""/> JavaScript
  order: 0
---

Parcel 包括对 JavaScript 的一流支持，包括 ES 模块和 CommonJS、多种类型的依赖项、浏览器目标的自动转换、JSX 和 TypeScript 支持等等。

## 模块 Modules

模块允许您将代码分解为不同的文件，并通过导入和导出值在它们之间共享功能。这可以帮助您将代码构建成独立的部分，并使用定义良好的接口在它们之间进行通信。

Parcel 包括对 ES 模块和 CommonJS 语法的支持。模块说明符按照 [依赖解析 Dependency resolution](/features/dependency-resolution/) 中的描述解析。

### ES modules

ES 模块语法是在 JavaScript 文件之间导入和导出值的标准方法。对于新代码，它应该优于 CommonJS。 [`import`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) 语句可用于引用 [`export`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) 声明在另一个文件中。

这个例子从 `math.js` 中导入了一个 `multiply` 函数，并使用它来实现 `square` 函数。

{% sample %}
{% samplefile "square.js" %}

```javascript
import { multiply } from "./math.js";

export function square(x) {
  return multiply(x, x);
}
```

{% endsamplefile %}
{% samplefile "math.js" %}

```javascript
export function multiply(a, b) {
  return a * b;
}
```

{% endsamplefile %}
{% endsample %}

要了解有关 ES 模块的更多信息，请参阅 [MDN] (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 上的文档。

### CommonJS

CommonJS 是 Node 支持的遗留模块系统，并被 npm 上的库广泛使用。如果您正在编写新代码，您通常应该更喜欢上面描述的 ES 模块语法。 CommonJS 提供了一个 `require` 函数，可以用来访问另一个文件暴露的 `exports` 对象。

此示例使用 `require` 加载 `math.js` 模块，并在其导出对象上使用 `multiply` 函数来实现 `square` 函数。

{% sample %}
{% samplefile "square.js" %}

```javascript
let math = require("./math");

exports.square = function (x) {
  return math.multiply(x, x);
};
```

{% endsamplefile %}
{% samplefile "math.js" %}

```javascript
exports.multiply = function (a, b) {
  return a * b;
};
```

{% endsamplefile %}
{% endsample %}

除了 `require` 和 `exports`，Parcel 还支持 `module.exports`，以及 `__dirname` 和 `__filename` 模块全局变量，以及用于访问环境变量的 `process.env`。有关更多详细信息，请参阅 [Node emulation](/features/node-emulation/) 文档。

要了解有关 CommonJS 的更多信息，请参阅 [Node.js 文档](https://nodejs.org/dist/latest-v16.x/docs/api/modules.html)。

### Dynamic import

ES 模块 `import` 语句和 CommonJS `require` 函数都*同步*加载依赖：即可以立即引用该模块，而无需等待网络请求。通常，同步导入的模块与它们的父模块一起捆绑到同一个文件中，或者在另一个已经加载的包中引用。

动态 [`import()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#dynamic_imports) 函数可用于*异步*加载依赖项。这允许根据需要延迟加载代码，并且是减少应用程序初始页面加载的文件大小的好方法。 `import()` 返回一个 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)，它会在加载依赖项时通知您。

```javascript
import("./pages/about").then(function (page) {
  page.render();
});
```

有关如何使用动态导入的更多详细信息，请参阅 [代码拆分 code splitting](/features/code-splitting/) 文档。

### 经典脚本 Classic scripts

模块（ES 模块和 CommonJS）的好处之一是它们隔离了功能。这意味着在顶级作用域中声明的变量不能在模块外部访问，除非它们被导出。这通常是一个很好的特性，但是 JavaScript 中的模块是相对较新的，并且有许多不希望被隔离的遗留库和脚本。

**经典脚本**，是一个 JavaScript 文件，它通过 HTML 中的传统 `<script>` 标签（没有 `type="module"`）或其他方式（例如 Workers）加载。经典脚本将顶级范围内的变量视为全局变量，即使在不同的脚本之间也可以访问。

例如，当加载像 jQuery 这样的库时，`$` 变量对页面上的其他脚本全局可用。如果 jQuery 作为模块加载，除非手动将其作为属性分配给 `window` 对象，否则 `$` 将无法访问。

```html
<script src="jquery.js"></script>
<script>
  // The $ variable is now available globally.
  $(".banner").show();
</script>
```

此外，经典脚本不支持通过 ES 模块或 CommonJS 同步导入或导出，并且 [Node emulation](/features/node-emulation/) 被禁用。然而，动态 `import()` _is_ 支持从经典脚本中加载模块。

Parcel 匹配经典脚本和模块的浏览器行为。如果您希望在代码中使用导入或导出，则需要使用 `<script type="module">` 从 HTML 文件中引用您的 JavaScript。对于工人，使用 `{type: 'module'}` 选项（见下文）。如果缺少此信息，您将看到如下所示的诊断信息。

![Screenshot of an error message showing "Browser scripts cannot have imports or exports. Add the type='module' attribute to the script tag."](/blog/rc0/script-module-error.png)

## `import.meta`

[`import.meta`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import.meta) 对象包含有关它被引用的模块的信息。当前包裹支持单个属性`import.meta.url`，其中包括文件系统上模块的`file://`url。此 URL 与您的项目根目录相关（例如，初始化 Git 的目录）以避免暴露任何构建系统详细信息。

```swift
console.log(import.meta);
// => {url: 'file:///src/App.js'}

console.log(import.meta.url);
// => 'file:///src/App.js'
```

## URL dependencies

您可以使用 [`URL`](https://developer.mozilla.org/en-US/docs/Web/API/URL/URL) 构造函数从 JavaScript 文件引用非 JavaScript 资产，例如图像或视频。这些是通过使用 `import.meta.url` 作为基本 URL 参数相对于模块解析的。第一个参数必须是要识别的字符串文字（不是变量或表达式）。

此示例引用与 JavaScript 文件位于同一目录中的名为 `hero.jpg` 的图像，并从中创建一个 `<img>` 元素。

```javascript/1
let img = document.createElement('img');
img.src = new URL('hero.jpg', import.meta.url);
document.body.appendChild(img);
```

Parcel 将像处理任何其他依赖项一样处理 URL 依赖项引用的任何文件。例如，图像将由图像转换器处理，您可以使用 [查询参数 query parameters](/features/dependency-resolution/#query-parameters) 指定调整大小和转换它们的选项。如果没有为特定文件类型配置转换器，则该文件将不加修改地复制到 dist 目录中。生成的文件名还将包含 [内容哈希](/features/production/#content-hashing)，以在浏览器中进行长期缓存。

## Workers

Parcel 内置了对 Web Worker、Service Worker 和 Worklet 的支持，它们允许将工作转移到不同的线程。

### Web workers

Web worker 是最通用的 worker 类型。它们允许您在后台线程中运行任意 CPU 繁重的工作，以避免阻塞用户界面。 Worker 是使用 [`Worker`](https://developer.mozilla.org/en-US/docs/Web/API/Worker/Worker) 构造函数创建的，并使用 `URL` 构造函数引用另一个 JavaScript 文件，如所述以上。

要在 worker 中使用 ES 模块或 CommonJS 语法，请使用上面 [Classic scripts](#classic-scripts) 中描述的 `type: 'module'` 选项。如有必要，Parcel 将根据您的 [targets](/features/targets/) 和当前浏览器支持将您的 worker 编译为非模块 worker。

```javascript
new Worker(new URL("worker.js", import.meta.url), { type: "module" });
```

Parcel 还支持 [`SharedWorker`](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker/SharedWorker) 构造函数，它创建一个可以在不同浏览器窗口或 iframe 中访问的 worker .它支持与上述 Worker 相同的选项。

要了解有关网络工作者的更多信息，请参阅 [MDN] (https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) 上的文档。

### Service workers

Service Worker 在后台运行并提供离线缓存、后台同步和推送通知等功能。它们是使用 [`navigator.serviceWorker.register`](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerContainer/register) 函数创建的，并使用 `URL` 构造函数来引用另一个 JavaScript 文件。

要在 service worker 中使用 ES 模块或 CommonJS 语法，请使用上面 [Classic scripts](#classic-scripts) 中描述的 `type: 'module'` 选项。如有必要，Parcel 将根据您的 [targets](/features/targets/) 和当前浏览器支持将您的 service worker 编译为非模块 worker。

```javascript
navigator.serviceWorker.register(
  new URL("service-worker.js", import.meta.url),
  { type: "module" }
);
```

{% note %}

**注意**：Service Worker 不支持动态 `import()`。

{% endnote %}

Service Worker 通常用于预缓存静态资产，如 JavaScript、CSS 和图像。 `@parcel/service-worker` 可用于从您的 service worker 中访问捆绑 URL 列表。它还提供了一个“版本”哈希，每次清单都会改变。您可以使用它来生成缓存名称。

首先，将其作为依赖项安装到您的项目中。

```shell
yarn add @parcel/service-worker
```

这个例子展示了如何在安装 service worker 时预先缓存所有的 bundle，并在它被激活时清理旧版本。

{% sample %}
{% samplefile "service-worker.js" %}

```javascript
import { manifest, version } from "@parcel/service-worker";

async function install() {
  const cache = await caches.open(version);
  await cache.addAll(manifest);
}
addEventListener("install", (e) => e.waitUntil(install()));

async function activate() {
  const keys = await caches.keys();
  await Promise.all(keys.map((key) => key !== version && caches.delete(key)));
}
addEventListener("activate", (e) => e.waitUntil(activate()));
```

{% endsamplefile %}
{% endsample %}

要了解有关服务工作者的更多信息，请参阅 [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers) 上的文档和 [Offline Cookbook](https:// /web.dev/offline-cookbook/) 在 web.dev 上。

### Classic script workers

`type: 'module'` 选项也可以省略，以将工作人员和服务工作人员视为经典脚本而不是模块。 [`importScripts`](https://developer.mozilla.org/en-US/docs/Web/API/WorkerGlobalScope/importScripts) 函数可用于在经典脚本工作者中加载附加代码，但是，代码将被加载在运行时，不会被包裹处理。这是因为 `importScripts` 解析相对于页面的 URL，而不是工作脚本。因此，`importScripts` 的参数必须是完全限定的绝对 URL，而不是相对路径。

以下示例显示了受支持和不受支持的模式。

```javascript
// ✅ absolute URL
importScripts("http://some-cdn.com/worker.js");

// ✅ computed URL
importScripts(location.origin + "/worker.js");

// 🚫 relative path
importScripts("worker.js");

// 🚫 absolute path
importScripts("/worker.js");
```

### Worklets

Parcel 还支持 worklets，包括 [CSS Houdini paint worklets](https://developers.google.com/web/updates/2018/01/paintapi) 以及 [web audio worklets](https://developer.mozilla. org/en-US/docs/Web/API/Web_Audio_API/Using_AudioWorklet）。这些让您可以在浏览器中连接到渲染过程或音频处理管道的低级方面。

使用以下语法自动检测 Paint worklets ：

```javascript
CSS.paintWorklet.addModule(new URL("worklet.js", import.meta.url));
```

Web 音频工作集不是静态可分析的，因此对于这些，您可以使用 `worklet:` 方案将 URL 导入为正确环境编译的工作集文件。

```javascript
import workletUrl from "worklet:./worklet.js";

context.audioWorklet.addModule(workletUrl);
```

worklets 始终是模块——没有经典的脚本 worklet。这意味着 worklet 不需要 `type: 'module'` 选项，并且不支持 `importScripts`。

此外，worklets 不支持动态 `import()`。

## 转译 Transpilation

Parcel 支持开箱即用地转译 JSX、TypeScript 和 Flow，以及转译现代 JavaScript 语法以支持旧版浏览器。此外，支持 Babel 以启用实验性或自定义 JavaScript 转换。

### JSX

Parcel 开箱即用地支持 JSX。 JSX 在 `.jsx` 或 `.tsx` 文件中自动启用，或者当以下库之一被定义为 package.json 中的依赖项时：

- `react`
- `preact`
- `nervjs`
- `hyperapp`

正确的 JSX 编译指示也会根据您使用的库自动推断。 Parcel 还会自动检测已安装的 React 或 Preact 版本，并启用 [modern JSX transform](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html) 如果支持。

JSX 编译也可以使用 `tsconfig.json` 或 `jsconfig.json` 文件进行配置。这允许覆盖运行时、编译指示和其他选项。有关详细信息，请参阅 [TSConfig 参考](https://www.typescriptlang.org/tsconfig)。

{% sample %}
{% samplefile "tsconfig.json" %}

```json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "jsxImportSource": "preact"
  }
}
```

{% endsamplefile %}
{% endsample %}

### Flow

当 `flow-bin` 在项目的根 package.json 中列为依赖项时，会自动启用 [Flow](https://flow.org) 支持。您还必须在要编译的文件中使用`@flow` 指令。

Parcel 目前使用 Babel 来剥离流类型。如果您有自定义 Babel 配置，则需要自己添加 Flow 插件。有关详细信息，请参阅 [Babel](#babel)。

### TypeScript

请参阅[TypeScript](/languages/typescript/)。

### 浏览器兼容性 Browser compatibility

默认情况下，Parcel 不会为旧浏览器执行任何 JavaScript 语法转换。这意味着如果您使用现代语言功能编写代码，这就是 Parcel 将输出的内容。您可以使用 package.json 中的 `browserslist` 字段声明应用支持的浏览器。声明此字段时，Parcel 将相应地转译您的代码，以确保与您支持的浏览器兼容。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "browserslist": "> 0.5%, last 2 versions, not dead"
}
```

{% endsamplefile %}
{% endsample %}

请参阅 [Targets](/features/targets/) 文档以获取有关如何配置此功能的更多详细信息，以及 Parcel 对自动 [差异捆绑](/features/targets/#differential-bundling) 的支持。

默认情况下，Parcel 支持所有标准 JavaScript 功能，以及在一个或多个浏览器中提供的提案。您还可以使用 `tsconfig.json` 或 `jsconfig.json` 文件启用对未来提案的支持。目前，唯一支持的提案是 [decorators](https://github.com/tc39/proposal-decorators)，您可以通过 TypeScript 使用它。

{% sample %}
{% samplefile "tsconfig.json" %}

```json
{
  "compilerOptions": {
    "experimentalDecorators": true
  }
}
```

{% endsamplefile %}
{% endsample %}

可以使用 [Babel](#babel) 启用更多实验性功能。

### Babel

[Babel](https://babeljs.io/) 是一个流行的 JavaScript 转译器，拥有庞大的插件生态系统。将 Babel 与 Parcel 一起使用与单独使用它或与其他构建工具一起使用的方式相同。创建一个 Babel 配置文件，例如 `.babelrc`，Parcel 会自动获取它。

Parcel 支持项目范围的配置文件，例如 `babel.config.json`，以及文件相关配置，例如 `.babelrc`。有关配置的详细信息，请参阅 [Babel docs](https://babeljs.io/docs/en/config-files)。

{% warning %}

**注意**：应避免使用 JavaScript Babel 配置（例如 `babel.config.js`）。这些会导致 Parcel 的缓存效率较低，这意味着每次重新启动 Parcel 时都会重新编译所有 JS 文件。为避免这种情况，请改用基于 JSON 的配置格式（例如 `babel.config.json`）。

{% endwarning %}

#### Default presets

Parcel 包括对浏览器目标（相当于`@babel/preset-env`）、JSX（相当于`@babel/preset-react`）、TypeScript（相当于`@babel/preset-typescript`）和 Flow（默认相当于`@babel/preset-flow`）。如果这些是您项目中唯一需要的转换，那么您可能根本不需要 Babel。

如果您有一个现有项目的 Babel 配置仅包含上述预设，您可以将其删除。这可以显着提高构建性能，因为 Parcel 的内置转译器比 Babel 快得多。

#### 自定义插件 Custom plugins

如果您有自定义 Babel 预设或上面列出的插件之外的插件，您可以将 Babel 配置为仅包含这些自定义插件并省略标准预设。这将通过允许 Babel 做更少的工作并让 Parcel 处理默认转换来提高构建性能。

{% sample %}
{% samplefile "babel.config.json" %}

```json
{
  "plugins": ["@emotion/babel-plugin"]
}
```

{% endsamplefile %}
{% endsample %}

由于 Parcel 使用 Babel 转译 Flow，您需要在 Babel 配置中保留 `@babel/preset-flow` 以及任何自定义插件。否则，您的 Babel 配置会覆盖 Parcel 的默认设置。可以删除上面列出的其他 Babel 预设。

`@babel/preset-env` 和 `@babel/plugin-transform-runtime` 不知道 Parcel 的 [Targets](/features/targets/)，这意味着 [differential bundling](/features/targets/#differential-bundling）将无法正常工作。这可能会导致不必要的转译和更大的包大小。此外，`@babel/preset-env` 默认会转译 ES 模块，这可能会导致 [scope hoisting](/features/scope-hoisting/) 出现问题。

`@babel/preset-env` 和 `@babel/plugin-transform-runtime` 不是必需的，因为 Parcel 会自动处理浏览器目标的转译。但是，如果您出于某种原因需要它们，则可以使用 Parcel 的包装器，它们知道 Parcel 的目标。

{% sample %}
{% samplefile "babel.config.json" %}

```json
{
  "presets": ["@parcel/babel-preset-env"],
  "plugins": ["@parcel/babel-plugin-transform-runtime"]
}
```

{% endsamplefile %}
{% endsample %}

#### 与其他工具一起使用

虽然 Parcel 默认包含转译，但您可能仍需要将 Babel 与其他工具一起使用，例如 [Jest](https://jestjs.io) 之类的测试运行器和 [ESLint](https://eslint.org) 之类的 linter .如果是这种情况，您可能无法完全删除您的 Babel 配置。您可以让 Parcel 忽略您的 Babel 配置，这将具有性能优势并防止上述其他问题。

要禁用 Parcel 中的 Babel 转移，请重写默认的用于 JavaScript 的 Parcel 配置以排除`@parcel/transformer-babel`.。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "transformers": {
    "*.{js,mjs,jsx,cjs,ts,tsx}": [
      "@parcel/transformer-js",
      "@parcel/transformer-react-refresh-wrap"
    ]
  }
}
```

{% endsamplefile %}
{% endsample %}

这将允许其他工具继续使用您的 Babel 配置，但禁用 Parcel 中的 Babel 转译。

## 生产 Production

在生产模式下，Parcel 包括优化以减少代码的文件大小。有关其工作原理的更多详细信息，请参阅 [Production](/features/production/)。

### 缩小 Minification

在生产模式下，Parcel 会自动缩小您的代码以减小捆绑包的文件大小。默认情况下，Parcel 使用 [Terser](https://github.com/terser/terser) 执行缩小。要配置 Terser，您可以在项目根目录中创建一个 `.terserrc` 文件。有关可用选项的信息，请参阅 [Terser 文档](https://github.com/terser/terser#minify-options)。

### Tree shaking

在生产构建中，Parcel 静态分析每个模块的导入和导出，并删除所有未使用的内容。这称为“Tree shaking”或“死代码消除 dead code elimination”。静态和动态 `import()`、CommonJS 和 ES 模块，甚至跨具有 CSS 模块的语言都支持摇树。

Parcel 还尽可能将模块连接到单个作用域中，而不是将每个模块包装在单独的函数中。这称为“scope hoisting”。这有助于使缩小更有效，并通过使模块之间的引用静态而不是动态对象查找来提高运行时性能。

请参阅 [scope hoisting](/features/scope-hoisting/) 文档以获取使摇树更有效的提示。

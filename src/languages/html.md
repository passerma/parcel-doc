---
layout: layout.njk
title: HTML
eleventyNavigation:
  key: languages-html
  title: <img src="/assets/lang-icons/html5.svg" alt=""/> HTML
  order: 1
---

Parcel 包括对开箱即用的 HTML 的一流支持。 HTML 文件通常是您提供给 Parcel 的入口文件，所有依赖项（包括 JavaScript、CSS、图像和指向其他页面的链接）都从那里开始构建您的整个应用程序。

## 依赖 Dependencies

Parcel 检测 HTML 中对其他文件的大多数引用（例如 `<script>`、`<img>`、`<video>` 或 `<meta property="og:image">`）并处理它们。这些引用被重写，以便它们链接到正确的输出文件。

文件名相对于当前 HTML 文件进行解析，但您也可以使用 [absolute](/features/dependency-resolution/#absolute-specifiers) 和 [tilde](/features/dependency-resolution/#tilde-specifiers) 说明符.详见[Dependency resolution](/features/dependency-resolution/)。

### Stylesheets

`<link rel="stylesheet">` 元素可用于引用 HTML 中的样式表。您可以引用 CSS 文件或任何其他可编译为 CSS 的文件，例如 [SASS](/languages/sass/)、[Less](/languages/less/) 或 [Stylus](/languages/stylus)。

{% sample %}
{% samplefile "index.html" %}

```html/3
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="./style.less" />
  </head>
  <body>
    <h1>My Parcel app</h1>
  </body>
</html>
```

{% endsamplefile %}
{% samplefile "style.less" %}

```less
h1 {
  color: darkslategray;
}
```

{% endsamplefile %}
{% endsample %}

有关 Parcel 如何处理 CSS 的详细信息，请参阅 [CSS](/languages/css/) 文档。

### Scripts

`<script>` 元素可用于从 HTML 引用脚本文件。您可以引用 JavaScript 文件或任何其他编译为 JavaScript 的文件，例如 [TypeScript](/languages/typescript/)、[JSX](/languages/javascript/#jsx) 或 [CoffeeScript](/languages/coffeescript /)。

{% sample %}
{% samplefile "index.html" %}

```html/3
<!DOCTYPE html>
<html>
  <head>
    <script type="module" src="app.ts" />
  </head>
  <body>
    <h1>My Parcel app</h1>
  </body>
</html>
```

{% endsamplefile %}
{% samplefile "app.ts" %}

```js
console.log("Hello world!");
```

{% endsamplefile %}
{% endsample %}

`type="module"` 属性应该用于指示文件是 [ES 模块](/languages/javascript/#es-modules) 或 [CommonJS](/languages/javascript/#commonjs) 文件。如果省略，则将引用的文件视为经典脚本。有关这方面的更多信息，请参阅 [Classic scripts](/languages/javascript/#classic-scripts)。

当使用 `<script type="module">` 时，如果您的某些浏览器目标不支持 ES 模块，Parcel 也会自动生成 `<script nomodule>` 版本。有关更多详细信息，请参阅 [Differential bundling](/features/targets/#differential-bundling)。

Parcel 还支持 `<script>` 元素的 `async` 和 `defer` 属性。当脚本是`async`时，它可以在运行时以任意顺序加载。因此，Parcel 将异步脚本视为“孤立的”。这意味着异步脚本不能与页面上的其他脚本共享任何依赖关系，这可能会导致模块重复。因此，最好避免使用 `async` 脚本，除非在特定情况下情况。

`defer` 属性具有与 `async` 类似的效果（非渲染阻塞），但保证脚本按照它们在 HTML 文件中定义的顺序执行。使用 `<script type="module">` 时，会自动启用 `defer`，不需要声明。

有关 `<script>` 元素的更多信息，请参阅 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script)，以及 [JavaScript](/languages/javascript/)文档了解 Parcel 如何处理 JavaScript 的详细信息。

### Images

Parcel 支持通过 [`<img>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img) 元素引用的图像。 `src` 属性可用于引用图像文件。

```html
<img src="image.jpg" alt="An image" />
```

Parcel 还支持 [`srcset`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#using_the_srcset_attribute) 属性，它允许引用不同大小或图像的多个版本决议。

```html
<img src="logo@1x.png" srcset="logo@2x.png 2x" alt="logo" />
```

此外，Parcel 还支持 [`<picture>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture) 元素，这为提供多个替代图像提供了更大的灵活性通过 [`<source>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/source) 元素。

Parcel 的 [image transformer](/recipes/image/) 也可用于从单个源文件生成图像的多个版本。这是使用 [query parameters](/features/dependency-resolution/#query-parameters) 来提供宽度、高度和格式来转换和调整源图像大小的。

```html
<picture>
  <source
    type="image/webp"
    srcset="image.jpg?as=webp&width=400, image.jpg?as=webp&width=800 2x"
  />
  <source
    type="image/jpeg"
    srcset="image.jpg?width=400, image.jpg?width=800 2x"
  />
  <img src="image.jpg?width=400" width="400" />
</picture>
```

### 链接和 iframe

Parcel 支持 [`<a>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a) 元素从 HTML 文件链接到另一个页面。

```html
<a href="other.html">Other page</a>
```

[`<iframe>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe) 元素可用于将 HTML 页面嵌入到另一个页面中。

```html
<iframe src="iframe.html"></iframe>
```

默认情况下，从 HTML 文件引用的其他资产将在其编译文件名中包含 [content hash](/features/production/#content-hashing)，而由 `<a>` 或 `<iframe>` 元素引用的文件将不是。这是因为这些 URL 通常是人类可读的，并且随着时间的推移需要有一个稳定的名称。 Bundle 命名可以被[Namer plugins](/plugin-system/namer/)覆盖。

### Video, audio, and other assets

[`<video>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video)、[`<audio>`](https://developer.mozilla. org/en-US/docs/Web/HTML/Element/audio), [`<track>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/track), [`<embed>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/embed) 和 [`<object>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/object) 元素。引用的 URL 由 Parcel 处理并重写以包含 [content hash](/features/production/#content-hashing)。

### Open Graph and Schema.org 元数据

Parcel 支持 [Open Graph](https://ogp.me)、[Twitter Cards](https://developer.twitter.com/en/docs/twitter-for-websites/cards/overview/markup)、[VK ](https://vk.com/dev/publications)、[Schema.org](https://schema.org) 和 [Microsoft pinned site](https://docs.microsoft.com/en-us /previous-versions/windows/internet-explorer/ie-developer/platform-apis/dn255024(v=vs.85)?redirectedfrom=MSDN) 使用 [`<meta>`](https://developer. mozilla.org/en-US/docs/Web/HTML/Element/meta) 标签。这些元素中引用的图像和其他 URL 由 Parcel 处理并重写以包含 [content hash](/features/production/#content-hashing)。

```html
<meta property="og:image" content="100x100.png" />
```

### JSON-LD

Parcel 支持通过 `<script>` 标签嵌入 HTML 中的 [JSON-LD](https://json-ld.org) 元数据。 JSON-LD 中引用的图像和其他 URL 由 Parcel 处理并重写以包含 [content hash](/features/production/#content-hashing)。这是由 `@parcel/transformer-jsonld` 插件处理的，它会在需要时自动安装到您的项目中。

在此示例中，从 `logo` 对象引用的图像将由 Parcel 处理。

```html
<script type="application/ld+json">
  {
    "@context": "http://schema.org",
    "@type": "LocalBusiness",
    "name": "Joe's Pizza",
    "description": "Delicious pizza for over 30 years.",
    "telephone": "555-111-2345",
    "openingHours": "Mo,Tu,We,Th,Fr 09:00-17:00",
    "logo": {
      "@type": "ImageObject",
      "url": "images/logo.png",
      "width": 180,
      "height": 120
    }
  }
</script>
```

### Web manifests

支持 `<link rel="manifest">` 元素引用 [Web manifest](https://developer.mozilla.org/en-US/docs/Web/Manifest)。这是一个 JSON 文件，其中包含有关应用程序的各种元数据，并允许将其安装到用户的主屏幕或桌面。 Parcel 处理此文件中的 `icons` 和 `screenshots` 键中引用的 URL。 Web 清单可以写在 `.json` 或 `.webmanifest` 文件中。

```html
<link rel="manifest" href="manifest.json" />
```

## 内联脚本和样式

带有文本内容的 `<script>` 和 `<style>` 标记也像独立文件一样被处理，并且生成的包被插入回 HTML 文件中。脚本标签上的 `type="module"` 属性的工作方式与上述外部脚本一样。但是，如果您的某些浏览器目标本身不支持 ES 模块，Parcel 只会将内联脚本编译为非模块脚本，并且不会输出 `<script type="module">` 以保留生成的 HTML 小的。

{% sample %}
{% samplefile "index.html" %}

```html/4,7-8
<!DOCTYPE html>
<html>
  <body>
    <style>
      @import "./style.scss";
    </style>
    <script type="module">
      import value from "./other.ts";
      console.log(value);
    </script>
  </body>
</html>
```

{% endsamplefile %}
{% endsample %}

{% warning %}

**注意**：谨慎使用，因为大的内联脚本/样式可能不利于（感知的）加载速度。

{% endwarning %}

## 内联`style`属性

[`style`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/style) 属性可用于任何 HTML 元素来定义 CSS 样式。 Parcel 将处理内联 CSS，并将结果重新插入到 `style` 属性中。这包括以下引用的 URL，例如背景图像，以及为您的目标浏览器转换现代 CSS。

```html
<div style="background: url(background.jpg)">Hello!</div>
```

## 内联 SVG

Parcel 支持 HTML 格式的 [inline SVG](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/SVG_In_HTML_Introduction)。通过 [`<image>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/image) 元素引用的图像和通过 [`<use>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/use) 元素，并且 SVG 中的内联脚本和样式也按上述方式处理。

```html
<!DOCTYPE html>
<html>
  <body>
    <svg xmlns:xlink="http://www.w3.org/1999/xlink">
      <rect x="10" y="10" width="50" height="50" fill="red" />
      <use xlink:href="icon.svg" />
    </svg>
  </body>
</html>
```

## 并行脚本和样式

引用脚本或样式时，有时 Parcel 需要将另一个依赖文件插入到已编译的 HTML 文件中。例如，当引用一个导入 CSS 文件的 JavaScript 文件时，Parcel 将在 `<head>` 中插入一个 `<link>` 元素，以与 JavaScript 并行加载此样式表。

{% sample %}
{% samplefile "index.html" %}

```html
<!DOCTYPE html>
<html>
  <head>
    <script type="module" src="app.js"></script>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

{% endsamplefile %}
{% samplefile "app.js" %}

```javascript
import "./app.css";

let app = document.createElement("div");
app.className = "app";
app.textContent = "My Parcel app!";
root.appendChild(app);
```

{% endsamplefile %}
{% samplefile "app.css" %}

```css
.app {
  background: red;
}
```

{% endsamplefile %}
{% endsample %}

Compiled HTML:

```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" src="app.f8e9c6.css" />
    <script type="module" src="app.26fce9.js"></script>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

这也可能发生在脚本中。例如，如果两个页面依赖于同一个 JavaScript 库（例如 React 或 Lodash），它可能会被拆分成自己的包并单独加载。 Parcel 将在编译后的 HTML 中插入一个 `<script>` 标记，以并行加载这个“共享包”。详情请参阅 [Code Splitting](/features/code-splitting/)。

## PostHTML

[PostHTML](https://github.com/posthtml/posthtml) 是一个用插件转换 HTML 的工具。您可以通过使用以下名称之一创建配置文件来配置 PostHTML：`.posthtmlrc`（JSON，**强烈**推荐）、`.posthtmlrc.js` 或 `posthtml.config.js`。

在您的应用中安装插件：

```bash
yarn add posthtml-doctype --dev
```

Then, create a config file:

{% sample %}
{% samplefile ".posthtmlrc" %}

```json
{
  "plugins": {
    "posthtml-doctype": { "doctype": "HTML 5" }
  }
}
```

{% endsamplefile %}
{% endsample %}

插件在插件对象中指定为键，选项使用对象值定义。如果插件没有选项，只需将其设置为空对象即可。

此示例使用 [posthtml-include](https://github.com/posthtml/posthtml-include) 来内联另一个 HTML 文件。

{% sample %}
{% samplefile ".posthtmlrc" %}

```json
{
  "plugins": {
    "posthtml-include": {}
  }
}
```

{% endsamplefile %}
{% samplefile "index.html" %}

```html
<html>
  <head>
    <title>Home</title>
  </head>
  <body>
    <include src="header.html"></include>
    <main>My content</main>
  </body>
</html>
```

{% endsamplefile %}
{% samplefile "header.html" %}

```html
<header>This is my header</header>
```

{% endsamplefile %}
{% endsample %}

### posthtml.config.js

对于一些需要将函数作为配置选项传递的插件，或者基于 `process.env.NODE_ENV` 设置插件，您需要使用 `posthtml.config.js` 文件而不是 `.posthtmlrc` 进行配置。

{% warning %}

**注意**：如果可能，应避免使用 JavaScript 配置文件。这些会导致 Parcel 的缓存效率降低，这意味着每次重新启动 Parcel 时都会重新编译所有 HTML 文件。为避免这种情况，请改用基于 JSON 的配置格式（例如 `.posthtmlrc`）。

{% endwarning %}

此示例使用 [posthtml-shorten](https://github.com/Rebelmail/posthtml-shorten) 使用自定义缩短功能缩短 URL。

```bash
yarn add posthtml-shorten --dev
```

{% sample %}
{% samplefile "posthtml.config.js" %}

```js
module.exports = {
  plugins: {
    "posthtml-shorten": {
      shortener: {
        process: function (url) {
          return new Promise((resolve, reject) => {
            resolve(url.replace(".html", ""));
          });
        },
      },
      tag: ["a"], // Allowed tags for URL shortening
      attribute: ["href"], // Attributes to replace on the elements
    },
  },
};
```

{% endsamplefile %}
{% samplefile "index.html" %}

```html
<html>
  <head>
    <title>Home</title>
  </head>
  <body>
    <a href="http://example.com/test.html">Example</a>
  </body>
</html>
```

{% endsamplefile %}
{% endsample %}

## Production

在生产模式下，Parcel 包括优化以减少代码的文件大小。有关其工作原理的更多详细信息，请参阅 [Production](/features/production/)。

### 缩小 Minification

在生产模式下，Parcel 会自动缩小您的代码以减小捆绑包的文件大小。默认情况下，Parcel 使用 [htmlnano](https://github.com/posthtml/htmlnano) 来执行 HTML 缩小。要配置 htmlnano，您可以在项目根目录中创建一个 `.htmlnanorc` 或 `.htmlnanorc.json` 文件。

例如保留 HTML 注释

{% sample %}
{% samplefile ".htmlnanorc" %}

```json
{
  "removeComments": false
}
```

{% endsamplefile %}
{% endsample %}

或不缩小 SVG 元素。

{% sample %}
{% samplefile ".htmlnanorc" %}

```json
{
  "minifySvg": false
}
```

{% endsamplefile %}
{% endsample %}

{% warning %}

**注意**：基于 JavaScript 的配置也支持 `.htmlnanorc.js` 和 `htmlnano.config.js`，但应尽可能避免，因为这会降低 Parcel 缓存的有效性。请改用基于 JSON 的配置格式。

{% endwarning %}

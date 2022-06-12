---
layout: layout.njk
title: SVG
eleventyNavigation:
  key: languages-svg
  title: <img src="/assets/lang-icons/svg.svg" alt=""/> SVG
  order: 3
---

SVG 是一种基于 XML 的基于矢量的 2D 图形格式，支持交互性和动画。 Parcel 支持将 SVG 作为单独的文件、嵌入在 HTML 中或作为 JSX 导入到 JavaScript 文件中。

## Dependencies

Parcel 检测 SVG 中对其他文件的大多数引用（例如 `<script>`、`<image>` 和 `<use>`）并处理它们。这些引用被重写，以便它们链接到正确的输出文件。

文件名相对于当前 SVG 文件进行解析，但您也可以使用 [absolute](/features/dependency-resolution/#absolute-specifiers) 和 [tilde](/features/dependency-resolution/#tilde-specifiers) 说明符.详见[依赖解析 Dependency resolution](/features/dependency-resolution/)。

### Stylesheets

外部样式表可以通过 SVG 文档中的 `xml-stylesheet` 处理指令来引用。您可以引用 CSS 文件或任何其他可编译为 CSS 的文件，例如 [SASS](/languages/sass/)、[Less](/languages/less/) 或 [Stylus](/languages/stylus)。

{% sample %}
{% samplefile "example.svg" %}

```xml/0
<?xml-stylesheet href="style.css" ?>
<svg viewBox="0 0 240 20" xmlns="http://www.w3.org/2000/svg">
  <text>Red text</text>
</svg>
```

{% endsamplefile %}
{% samplefile "style.css" %}

```css
text {
  fill: red;
}
```

{% endsamplefile %}
{% endsample %}

有关 Parcel 如何处理 CSS 的详细信息，请参阅 [CSS](/languages/css/) 文档。

### Scripts

`<script>` 元素可用于从 SVG 引用脚本文件。您可以引用 JavaScript 文件或任何其他编译为 JavaScript 的文件，例如 [TypeScript](/languages/typescript/)、[JSX](/languages/javascript/#jsx) 或 [CoffeeScript](/languages/coffeescript /)。

`type="module"` 属性应该用于指示文件是 [ES 模块](/languages/javascript/#es-modules) 或 [CommonJS](/languages/javascript/#commonjs) 文件。如果省略，则将引用的文件视为经典脚本。有关这方面的更多信息，请参阅 [Classic scripts](/languages/javascript/#classic-scripts)。 SVG 中尚不支持 ES 模块，因此 Parcel 将所有 JavaScript 编译为经典脚本，即使是作为模块编写的。

{% note %}

**注意**：SVG 对 `<script>` 元素使用 `href` 属性而不是 `src` 属性。

{% endnote %}

{% sample %}
{% samplefile "example.svg" %}

```xml/2
<svg viewBox="0 0 240 80" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="50" fill="red" />
  <script type="module" href="interactions.js" />
</svg>
```

{% endsamplefile %}
{% samplefile "interactions.js" %}

```javascript
let circle = document.querySelector("circle");
circle.addEventListener("click", () => {
  circle.setAttribute("fill", "blue");
});
```

{% endsamplefile %}
{% endsample %}

有关 `<script>` 元素的更多信息，请参阅 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/script)，以及 [JavaScript](/languages/javascript/) 文档了解 Parcel 如何处理 JavaScript 的详细信息。

### Images

可以使用 [`<image>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/image) 元素将光栅图像或其他 SVG 嵌入到 SVG 文件中。 Parcel 识别 `href` 和 `xlink:href` 属性。

```xml
<image href="image.jpg" width="100" height="50" />
```

Parcel 的图像转换器还可以通过使用 [Query parameters](/features/dependency-resolution/#query-parameters) 来调整图像大小和转换图像。

```xml
<image href="image.jpg?as=webp" width="100" height="50" />
```

{% note %}

**注意**：通过 `<image>` 元素引用的 SVG 不会加载外部资源，例如样式表、字体和其他图像，并且脚本和交互性被禁用。

{% endnote %}

有关 Parcel 如何处理图像的详细信息，请参阅 [Image](/recipes/image/) 文档。

### Links

SVG 文件可以使用 [`<a>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/a) 元素链接到其他网页或文件。 Parcel 支持 `href` 和 `xlink:href` 属性。

```xml
<a href="circle.html">
  <circle cx="50" cy="40" r="35" />
</a>
```

默认情况下，从 SVG 文件引用的其他资产将在其编译文件名中包含 [content hash](/features/production/#content-hashing)，而由 `<a>` 元素引用的文件不会。这是因为这些 URL 通常是人类可读的，并且随着时间的推移需要有一个稳定的名称。 Bundle 命名可以被[Namer plugins](/plugin-system/namer/)覆盖。

### 外部参考 External references

Parcel 通过许多其他元素的 `href` 和 `xlink:href` 属性支持外部引用。有关详细信息，请参阅 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/href)。

```xml/0,2
<use href="fox.svg#path" stroke="red" />
<text>
  <textPath href="fox.svg#path">
    Quick brown fox jumps over the lazy dog.
  </textPath>
</text>
```

通过`url()`函数在表示属性(如`fill`, `stroke`, `clip-path`)中引用的外部资源，以及其他许多其他属性也受到支持。

```xml
<circle
  cx="50" cy="40" r="35"
  fill="url(external.svg#gradient)" />
```

## 内联脚本和样式

带有文本内容的 `<script>` 和 `<style>` 标签也像独立文件一样被处理，并且生成的包被插入回 SVG 文件中。使用如上所述的 `type="module"` 属性来启用从内联脚本导入其他模块。

{% sample %}
{% samplefile "example.svg" %}

```xml/3,6-8
<svg viewBox="0 0 240 80" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="50" />
  <style>
    @import './style.scss';
  </style>
  <script type="module">
    import {setup} from './interactions.ts';
    let circle = document.querySelector('circle');
    setup(circle);
  </script>
</svg>
```

{% endsamplefile %}
{% samplefile "style.scss" %}

```scss
$fill: blue;

circle {
  fill: $fill;
}
```

{% endsamplefile %}
{% samplefile "interactions.ts" %}

```typescript
export function setup(element: SVGElement) {
  element.addEventListener("click", () => {
    element.setAttribute("fill", "red");
  });
}
```

{% endsamplefile %}
{% endsample %}

通过 `@import` 引用的 CSS 文件和通过 `import` 引用的 JavaScript 将被捆绑到编译后的 SVG 文件中。有关如何引用外部文件的信息，请参阅 [Stylesheets](#stylesheets) 和 [Scripts](#scripts)。

## 内联`style`属性

[`style`](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/style) 属性可用于任何 SVG 元素来定义 CSS 样式。 Parcel 将处理内联 CSS，并将结果重新插入到 `style` 属性中。这包括以下引用的 URL，以及为您的目标浏览器转换现代 CSS。

```xml
<circle
  cx="50" cy="40" r="35"
  style="fill: url(external.svg#gradient)" />
```

## HTML 中的 SVG

HTML 中的 SVG 既可以作为外部文件引用，也可以直接嵌入到 HTML 文档中。

### 外部 SVG

可以通过多种方式从 HTML 引用 SVG 文件。最简单的是使用 `<img>` 元素，并使用 `src` 属性引用 SVG 文件。 Parcel 将遵循参考并处理 SVG 及其所有依赖项。

```html
<img src="logo.svg" alt="logo" />
```

如果您的 SVG 是静态的，这种方法非常有效。如果 SVG 引用了外部资源，例如其他 SVG、图像、字体、样式表或脚本，或者包含任何交互性，它将无法工作。您也无法通过 HTML 页面中的 CSS 更改 SVG 的样式或使用 JavaScript 操作 SVG 的 DOM，并且用户无法选择 SVG 中的任何文本。

[`<object>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/object) 元素可用于在 HTML 中嵌入外部 SVG 并启用外部引用、脚本、交互性和文本选择。使用 `data` 属性来引用 SVG 文件。

```html
<object data="interactive.svg" title="Interactive SVG"></object>
```

这也允许您通过 `<object>` 元素上的 `getSVGDocument()` 方法访问 SVG DOM。

```javascript
let object = document.querySelector("object");
let svg = object.getSVGDocument();
let circle = svg.querySelector("circle");
circle.setAttribute("fill", "red");
```

但是，使用 `<object>` 元素嵌入的 SVG 无法通过 HTML 页面上的 CSS 设置样式。

### 内联 SVG

SVG 可以直接内联到 HTML 中，而不是作为单独的文件引用。这允许 HTML 页面上的 CSS 设置 SVG 元素的样式。 Parcel 在嵌入的 SVG 中支持外部引用，就像 SVG 位于单独的文件中时一样。

```html/3-5
<!DOCTYPE html>
<html>
  <body>
    <svg width="100" height="100">
      <circle cx="50" cy="50" r="50" />
    </svg>
    <style>
      circle {
        fill: blue;
      }

      circle:hover {
        fill: green;
      }
    </style>
  </body>
</html>
```

## CSS 中的 SVG

可以使用 `url()` 函数从 CSS 文件中引用 SVG。与 `<img>` 元素一样，背景图像中的 SVG 不支持样式表等外部资源，并且禁用了脚本和交互性。

```css
.logo {
  background: url("logo.svg");
}
```

您还可以使用数据 URL 将小型 SVG 嵌入到 CSS 文件中。使用 `data-url:` 方案来执行此操作，Parcel 将构建 SVG 并将结果内联到已编译的 CSS 中。有关详细信息，请参阅 [Bundle inlining](/features/bundle-inlining/)。

```css
.logo {
  background: url("data-url:logo.svg");
}
```

## SVG in JavaScript

SVG 文件既可以作为 JavaScript 的外部 URL 引用，也可以作为字符串内联，或者转换为 JSX 以在 React 等框架中呈现。

### URL references

Parcel 支持使用`URL`构造函数引用 SVG 文件。这个示例使用结果使用 JSX 渲染一个`<img>`元素。这与上面的[External SVG](#external-svg)中描述的方法相同。如果 SVG 是交互式的或者有外部资源，则可以使用`<object>`元素。

```jsx/0
const logo = new URL('logo.svg', import.meta.url);

export function Logo() {
  return <img src={logo} alt="logo" />;
}
```

有关更多详细信息，请参阅 JavaScript 文档中的 [URL dependencies](/languages/javascript/#url-dependencies)。

### 内联为字符串

SVG 可以通过使用 `bundle-text:` 方案导入，在 JavaScript 中作为字符串内联。

```javascript/0
import svg from 'bundle-text:./logo.svg';

let logo = document.createElement('div');
logo.innerHTML = svg;
document.body.appendChild(logo);
```

有关详细信息，请参阅 [Bundle inlining](/features/bundle-inlining/)。

### 作为 React 组件导入

`@parcel/transformer-svg-react` 插件可用于将 SVG 文件作为 React 组件导入。这使用 [SVGR](https://react-svgr.com) 将 SVG 文件转换为 JSX。它还使用 [SVGO](https://github.com/svg/svgo) 来优化 SVG 以减小文件大小。

此插件不包含在默认 Parcel 配置中，因此您需要安装它并将其添加到您的 `.parcelrc` 中。

```shell
yarn add @parcel/transformer-svg-react --dev
```

您可以配置 `.parcelrc` 以将所有 SVG 转换为 JSX，或者使用命名管道创建可以从 JavaScript 导入语句引用的 URL 方案。这种方法允许从 JavaScript 引用的 SVG 文件转换为 JSX，但在其他地方引用的 SVG 保存为 SVG 文件。在将 SVG 转换为 JSX 之前，首先使用 `"..."` 语法运行默认的 SVG 转换器。

{% sample %}
{% samplefile ".parcelrc" %}

```json/3
{
  "extends": "@parcel/config-default",
  "transformers": {
    "jsx:*.svg": ["...", "@parcel/transformer-svg-react"]
  }
}
```

{% endsamplefile %}
{% samplefile "example.jsx" %}

```jsx/0
import Icon from "jsx:./icon.svg";

export const App = () => <Icon />;
```

{% endsamplefile %}
{% endsample %}

## Production

在生产模式下，Parcel 包括优化以减少代码的文件大小。有关其工作原理的更多详细信息，请参阅 [Production](/features/production/)。

### 缩小 Minification

在生产模式下，Parcel 会自动缩小您的代码以减小打包的文件大小。默认情况下，Parcel 使用 [SVGO](https://github.com/svg/svgo) 来执行 SVG 缩小。

要配置 SVGO，您可以在项目根目录中创建一个 `svgo.config.json` 文件。要查看 SVGO 的所有可用配置选项，请参阅 [官方文档](https://github.com/svg/svgo#configuration)。

{% sample %}
{% samplefile "svgo.config.json" %}

```json
{
  "plugins": [
    {
      "name": "preset-default",
      "params": {
        "overrides": {
          "inlineStyles": false
        }
      }
    }
  ]
}
```

{% endsamplefile %}
{% endsample %}

{% warning %}

**注意**：基于 JavaScript 的配置也支持 `svgo.config.js`，但应尽可能避免，因为它会降低 Parcel 缓存的有效性。请改用基于 JSON 的配置格式。

{% endwarning %}

---
layout: layout.njk
title: CSS
eleventyNavigation:
  key: languages-css
  title: <img src="/assets/lang-icons/postcss.svg" alt=""/> CSS
  order: 2
---

Parcel 包括开箱即用的 CSS 支持。要添加 CSS 文件，请在 HTML 文件中使用 `<link>` 标记引用它：

```html
<link rel="stylesheet" href="index.css" />
```

或从 JavaScript 文件中导入：

```javascript
import "./index.css";
```

## Dependencies

CSS 资源可以包含由 `@import` 语法引用的依赖项，以及通过 `url()` 函数对图像、字体等的引用。

### `@import`

[`@import`](https://developer.mozilla.org/en-US/docs/Web/CSS/@import) at-rule 可用于将另一个 CSS 文件内联到与包含文件相同的 CSS 包中文件。这意味着在运行时不需要单独的网络请求来加载依赖项。

```css
@import "other.css";
```

引用的文件应该是 [relative](/features/dependency-resolution/#relative-specifiers) 到包含的 CSS 文件。您还可以使用 [absolute](/features/dependency-resolution/#absolute-specifiers) 和 [tilde](/features/dependency-resolution/#tilde-specifiers) 说明符。要从 npm 导入 CSS 文件，请使用 `npm:` [scheme](/features/dependency-resolution/#url-schemes)。

```css
@import "npm:bootstrap/bootstrap.css";
```

启用 `@parcel/resolver-glob` 插件后，您还可以使用 glob 一次导入多个 CSS 文件。有关详细信息，请参阅 [Glob 说明符](/features/dependency-resolution/#glob-specifiers)。

```css
@import "./components/*.css";
```

### `url()`

[`url()`](<https://developer.mozilla.org/en-US/docs/Web/CSS/url()>) 函数可用于引用文件，例如背景图像或字体。引用的文件将由 Parcel 处理，并且 URL 引用将被重写以指向输出文件名。

```css
body {
  background: url(images/background.png);
}
```

引用的文件应该是 [relative](/features/dependency-resolution/#relative-specifiers) 到包含的 CSS 文件。您还可以使用 [absolute](/features/dependency-resolution/#absolute-specifiers) 和 [tilde](/features/dependency-resolution/#tilde-specifiers) 说明符。 `data-url:` 方案也可用于将文件内联为数据 URL。有关详细信息，请参阅 [Bundle inlining](/features/bundle-inlining/)。

```css
.logo {
  background: url("data-url:./logo.png");
}
```

{% warning %}

**注意**：只有 [绝对路径 absolute paths](/features/dependency-resolution/#absolute-specifiers) 可以在 CSS 自定义属性中使用，而不是相对路径。这是因为自定义属性中的 `url()` 引用是从使用 `var()` 的位置解析的，而不是定义自定义属性的位置。这意味着自定义属性可以根据使用的文件解析为不同的 URL。要解决这种歧义，请在自定义属性中引用 URL 时使用绝对路径。

{% sample %}
{% samplefile "/src/index.css" %}

```css
body {
  /* ❌ relative paths are not allowed in custom properties. */
  --logo: url(images/logo.png);
  /* ✅ use absolute paths instead. */
  --logo: url(/src/images/logo.png);
}
```

{% endsamplefile %}
{% samplefile "/src/home/header.css" %}

```css
.logo {
  background: var(--logo);
}
```

{% endsamplefile %}
{% endsample %}

在上面的示例中，相对路径 `images/logo.png` 将解析为 `/src/home/images/logo.png` 而不是您所期望的 `/src/images/logo.png`，因为它是在 `/src/home/header.css` 中引用。无论在哪个文件 `var(--logo)` 中使用，绝对路径`/src/images/logo.png` 始终解析。

{% endwarning %}

## CSS modules

默认情况下，从 JavaScript 导入的 CSS 是全局的。如果两个 CSS 文件定义了相同的类名、id、自定义属性、`@keyframes` 等，它们可能会发生冲突并相互覆盖。为了解决这个问题，Parcel 支持 [CSS modules](https://github.com/css-modules/css-modules)。

CSS modules 将每个文件中定义的类视为唯一的。每个类名或标识符都被重命名以包含唯一的哈希，并且映射被导出到 JavaScript 以允许引用它们。

要使用 CSS modules，请创建一个带有 `.module.css` 扩展名的文件，然后从带有 [namespace import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#import_an_entire_modules_contents)的 JavaScript 文件中导入它。然后，您可以访问 CSS 文件中定义的每个类作为模块的导出。

```javascript
import * as classes from "./styles.module.css";

document.body.className = classes.body;
```

```css
.body {
  background: skyblue;
}
```

`.body` 类将被重命名为独特的东西，以避免选择器与其他 CSS 文件发生冲突。

CSS modules 也适用于编译成 CSS 的其他语言，例如 SASS、Less 或 Stylus。使用相应的文件扩展名命名您的文件，例如 `.module.scss`、`.module.less` 或 `.module.styl`。

### Tree shaking

使用 CSS 模块还具有在代码中明确对特定类名的依赖的好处。这可以自动删除未使用的 CSS 类。

![Example of tree shaking CSS modules](/blog/beta3/tree-shaking-css-modules.jpg)

如您在上面的示例中所见，仅使用了 `.button` 类，因此从编译的 CSS 文件中删除了未使用的 `.cta` 类。

这也适用于其他未使用的 CSS 规则，例如 `@keyframes` 和 `@counter-style`，以及 CSS 自定义属性（当启用 [`dashedIdents`](#local-css-variables) 选项时）。

{% warning %}

**注意**：仅当您使用 [命名空间 namespace](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#import_an_entire_modules_contents) 或[named](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#import_a_single_export_from_a_module) 导入。Tree shaking 不适用于 [default imports](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#importing_defaults)。

```javascript
import styles from "./styles.module.css";
```

should be replaced with:

```javascript
import * as styles from "./styles.module.css";
```

{% endwarning %}

### 本地 CSS 变量

默认情况下，类名、id 选择器以及 `@keyframes`、`@counter-style` 和 CSS 网格线和区域的名称都在定义它们的模块范围内。 CSS 变量和其他 [`< dashed-ident>`](https://www.w3.org/TR/css-values-4/#dashed-idents) 名称也可以使用项目根目录`package.json 中的`dashedIdents` 配置选项启用`。

{% sample %}
{% samplefile "package.json" %}

```json/3
{
  "@parcel/transformer-css": {
    "cssModules": {
      "dashedIdents": true
    }
  }
}
```

{% endsamplefile %}
{% endsample %}

启用后，CSS 变量将被重命名，因此它们不会与其他文件中定义的变量名冲突。引用变量使用标准的 `var()` 语法，Parcel 将更新该语法以匹配本地范围的变量名称。

您还可以使用 `from` 关键字引用其他文件中定义的变量：

{% sample %}
{% samplefile "style.module.css" %}

```css/1
.button {
  background: var(--accent-color from "./vars.module.css");
}
```

{% endsamplefile %}

{% samplefile "vars.module.css" %}

```css
:root {
  --accent-color: hotpink;
}
```

{% endsamplefile %}
{% endsample %}

全局变量可以使用 `from global` 语法来引用，但是，它们目前必须在非 CSS 模块文件中定义。

{% sample %}
{% samplefile "style.module.css" %}

```css/1
@import "vars.css";

.button {
  color: var(--color from global);
}
```

{% endsamplefile %}
{% samplefile "vars.css" %}

```css
:root {
  --color: purple;
}
```

{% endsamplefile %}
{% endsample %}

### 自定义命名模式

默认情况下，Parcel 会将文件名的哈希值添加到 CSS 文件中的每个类名和标识符之前。您可以使用项目根目录 `package.json` 中的 `"pattern"` 选项配置此命名模式。这接受一个带有占位符的字符串，将由 Parcel 填充，允许您添加自定义前缀或调整范围类的命名约定。

{% sample %}
{% samplefile "package.json" %}

```json/3
{
  "@parcel/transformer-css": {
    "cssModules": {
      "pattern": "my-company-[name]-[hash]-[local]"
    }
  }
}
```

{% endsamplefile %}
{% endsample %}

当前支持以下占位符：

- `[name]` - 文件的基本名称，不带扩展名。
- `[hash]` - 完整文件路径的哈希。
- `[local]` - 原始类名或标识符。

{% warning %}

**注意**：由于浏览器自动添加后缀，CSS 网格线名称可能不明确，它会为每个网格模板区域生成以 `-start` 和 `-end` 结尾的线名。使用 CSS 网格时，您的 `"pattern"` 配置必须以 `[local]` 占位符结尾，这样这些引用才能正常工作。

{% sample %}
{% samplefile "grid.module.css" %}

```css/5
.grid {
  grid-template-areas: "nav main";
}

.nav {
  grid-column-start: nav-start;
}
```

{% endsamplefile %}
{% samplefile "package.json" %}

```json5/5
{
  "@parcel/transformer-css": {
    "cssModules": {
      // ❌ [local] must be at the end so that
      // auto-generated grid line names work
      "pattern": "[local]-[hash]"
      // ✅ do this instead
      "pattern": "[hash]-[local]"
    }
  }
}
```

{% endsamplefile %}
{% endsample %}

{% endwarning %}

### 全局启用 CSS 模块

默认情况下，CSS 模块仅对名称以 `.module.css` 结尾的文件启用。默认情况下，所有其他 CSS 文件都被视为全局 CSS。但是，可以通过在项目根目录 `package.json` 中配置 `@parcel/transformer-css` 来覆盖所有源文件（即不在 `node_modules` 中）的 CSS 模块。

{% sample %}
{% samplefile "package.json" %}

```json/2
{
  "@parcel/transformer-css": {
    "cssModules": true
  }
}
```

{% endsamplefile %}
{% endsample %}

当使用带有其他选项的配置对象时，请改用`"global"`选项。

```json5/3
{
  "@parcel/transformer-css": {
    "cssModules": {
      "global": true,
      // ...
    }
  }
}
```

{% warning %}

**注意**：在之前的 Parcel 版本中，`postcss-modules` 用于实现 CSS 模块支持。在项目的 PostCSS 配置文件中全局启用 CSS 模块。如果您如上所述启用 CSS 模块，则此插件现在可以从您的 PostCSS 配置中删除。

如果这是您使用的唯一 PostCSS 插件，您可以完全删除您的 PostCSS 配置。这可以显着提高构建性能。如果你没有使用任何 `postcss-modules` 配置选项，你可能会看到一个警告。

{% endwarning %}

## 转译 Transpilation

Parcel 包括对转换现代 CSS 语法以支持开箱即用的旧浏览器的支持，包括供应商前缀和语法降低。此外，支持 [PostCSS](https://postcss.org) 以启用自定义 CSS 转换。

### 浏览器目标 Browser targets

默认情况下，Parcel 不会为旧浏览器执行任何 CSS 语法转换。这意味着如果您使用现代语法或不使用供应商前缀编写代码，这就是 Parcel 将输出的内容。您可以使用 package.json 中的 `browserslist` 字段声明应用支持的浏览器。声明此字段时，Parcel 将相应地转译您的代码，以确保与您支持的浏览器兼容。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "browserslist": "> 0.5%, last 2 versions, not dead"
}
```

{% endsamplefile %}
{% endsample %}

See the [Targets](/features/targets/) docs for more details on how to configure this.

### 浏览器前缀

根据您配置的浏览器目标，Parcel 会自动为许多 CSS 功能添加浏览器前缀的后备。例如，当使用 [`image-set()`](<https://developer.mozilla.org/en-US/docs/Web/CSS/image/image-set()>) 函数时，Parcel 会输出一个回退 `-webkit-image-set()` 值，因为 Chrome 还不支持无前缀值。

```css
.logo {
  background: image-set(url(logo.png) 2x, url(logo.png) 1x);
}
```

编译为：

```css
.logo {
  background: -webkit-image-set(url(logo.png) 2x, url(logo.png) 1x);
  background: image-set("logo.png" 2x, "logo.png");
}
```

此外，如果您的 CSS 源代码（或更可能是库）包含不必要的浏览器前缀，Parcel CSS 将自动删除它们以减少包大小。例如，在为现代浏览器编译时，`transition` 属性的前缀版本将被删除，因为所有浏览器都支持无前缀版本。

```css
.button {
  -webkit-transition: background 200ms;
  -moz-transition: background 200ms;
  transition: background 200ms;
}
```

becomes:

```css
.button {
  transition: background 0.2s;
}
```

### 语法降低

Parcel 会自动将许多现代 CSS 语法功能编译为目标浏览器支持的更兼容的输出。

支持以下功能：

- [Color Level 5](https://drafts.csswg.org/css-color-5/)
  - [`color-mix()`](https://drafts.csswg.org/css-color-5/#color-mix) function
- [Color Level 4](https://drafts.csswg.org/css-color-4/)
  - [`lab()`](<https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/lab()>), [`lch()`](<https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/lch()>), `oklab()`, and `oklch()` colors
  - [`color()`](<https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color()>) function supporting predefined color spaces such as `display-p3` and `xyz`
  - [`hwb()`](<https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/hwb()>) function
  - Space separated components in `rgb()` and `hsl()` functions
  - Hex colors with alpha, e.g. `#rgba` and `#rrggbbaa`
- [Logical properties](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Logical_Properties), e.g. `margin-inline-start`
- [Media query range syntax](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries#syntax_improvements_in_level_4), e.g. `@media (width <= 100px)` or `@media (100px < width < 500px)`
- Alignment shorthands, e.g. [`place-items`](https://developer.mozilla.org/en-US/docs/Web/CSS/place-items) and [`place-content`](https://developer.mozilla.org/en-US/docs/Web/CSS/place-content)
- [`clamp()`](<https://developer.mozilla.org/en-US/docs/Web/CSS/clamp()>) function
- [Double position gradient stops](https://css-tricks.com/while-you-werent-looking-css-gradients-got-better/) (e.g. `red 40% 80%`)
- Two-value [`overflow`](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow) shorthand
- Multi-value [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) property (e.g. `inline flex`)

### 语法草案 Draft syntax

Parcel 还可以配置为编译几个尚未在任何浏览器中本地提供的草案规范。因为这些是草稿并且语法仍然可以更改，所以必须在您的项目中手动启用它们。

#### 嵌套 Nesting

[CSS 嵌套](https://drafts.csswg.org/css-nesting/) 草案规范允许嵌套样式规则，子规则的选择器以某种方式扩展父选择器。 CSS 预处理器（如 SASS）非常普遍地支持这一点，但有了这个规范，它最终将在浏览器中得到原生支持。 Parcel 将此语法编译为当今所有浏览器都支持的非嵌套样式规则。

因为嵌套是草稿，所以默认情况下不启用。要使用它，请通过在项目根 `package.json` 文件中配置 `@parcel/transformer-css` 来启用它。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "@parcel/transformer-css": {
    "drafts": {
      "nesting": true
    }
  }
}
```

{% endsamplefile %}
{% endsample %}

一旦启用，项目中的任何 CSS 文件都可以使用直接嵌套样式规则或 `@nest`规则。

[Directly nested style rules](https://drafts.csswg.org/css-nesting/#direct) 必须以`&`嵌套选择器为前缀。这表明父选择器将被替换的位置。例如：

```css
.foo {
  color: blue;
  & > .bar {
    color: red;
  }
}
```

相当于：

```css
.foo {
  color: blue;
}
.foo > .bar {
  color: red;
}
```

[@nest rule](https://drafts.csswg.org/css-nesting/#at-nest) 允许在父选择器被替换的地方而不是在开始处进行嵌套。

```css
.foo {
  color: red;
  @nest .parent & {
    color: blue;
  }
}
```

相当于：

```css
.foo {
  color: red;
}
.parent .foo {
  color: blue;
}
```

[Conditional rules](https://drafts.csswg.org/css-nesting/#conditionals)如`@media`也可以嵌套在样式规则中，无需重复选择器。例如：

```css
.foo {
  display: grid;

  @media (orientation: landscape) {
    grid-auto-flow: column;
  }
}
```

相当于：

```css
.foo {
  display: grid;
}

@media (orientation: landscape) {
  .foo {
    grid-auto-flow: column;
  }
}
```

#### 自定义媒体查询 Custom media queries

媒体查询级别 5 草案规范中包含对 [custom media queries](https://drafts.csswg.org/mediaqueries-5/#custom-mq) 的支持。这允许您定义在 CSS 文件中的多个位置重复使用的媒体查询。启用此功能后，Parcel CSS 将提前执行 ​​ 此替换。

例如：

```css
@custom-media --modern (color), (hover);

@media (--modern) and (width > 1024px) {
  .a {
    color: green;
  }
}
```

相当于：

```css
@media ((color) or (hover)) and (width > 1024px) {
  .a {
    color: green;
  }
}
```

因为自定义媒体查询是草稿，所以默认情况下不启用它们。要使用它们，请通过在项目根 `package.json` 文件中配置 `@parcel/transformer-css` 来启用 `customMedia` 功能。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "@parcel/transformer-css": {
    "drafts": {
      "customMedia": true
    }
  }
}
```

{% endsamplefile %}
{% endsample %}

### 伪类替换

Parcel 支持用可以使用 JavaScript 应用的普通 CSS 类替换 CSS 伪类，例如 `:focus-visible`。这使得为 ​​ 旧版浏览器填充这些伪类成为可能。

伪类映射可以在您的项目根 `package.json` 文件中配置：

{% sample %}
{% samplefile "package.json" %}

```json
{
  "@parcel/transformer-css": {
    "pseudoClasses": {
      "focusVisible": "focus-visible"
    }
  }
}
```

{% endsamplefile %}
{% endsample %}

上述配置将导致所有选择器中的 `:focus-visible` 伪类被替换为 `.focus-visible` 类。这使您能够使用 JavaScript [polyfill](https://github.com/WICG/focus-visible)，它将适当地应用 `.focus-visible` 类。

如下伪类可以配置如上图：

- `hover` – 对应于 `:hover` 伪类
- `active` – 对应于 `:active` 伪类
- `focus` – 对应于 `:focus` 伪类
- `focusVisible` – 对应于 `:focus-visible` 伪类
- `focusWithin` – 对应于`:focus-within` 伪类

## PostCSS

[PostCSS](http://postcss.org/) 是一个用插件转换 CSS 的工具。虽然 Parcel 支持许多常见 PostCSS 插件的等效功能，例如 [autoprefixer](https://github.com/postcss/autoprefixer) 和 [postcss-preset-env](https://github.com/csstools/postcss-preset -env) 如上所述开箱即用，PostCSS 对于更多自定义 CSS 转换（例如非标准语法添加）很有用。它也被流行的 CSS 框架使用，例如 [Tailwind](https://tailwindcss.com)。

您可以通过使用以下名称之一创建配置文件来将 PostCSS 与 Parcel 一起使用：`.postcssrc`、`.postcssrc.json`、`.postcssrc.js` 或 `postcss.config.js`。

首先，将您希望使用的 postcss 插件安装到您的应用中：

```shell
yarn add tailwindcss --dev
```

然后，创建一个`.postcssrc`。插件在 `plugins` 对象中指定为键，而选项使用对象值定义。如果插件没有选项，只需将其设置为 `true`。

如果您的插件需要额外的配置，请同时创建这些文件。例如，使用 Tailwind，您需要一个 `tailwind.config.js`。

{% sample %}
{% samplefile ".postcssrc" %}

```json
{
  "plugins": {
    "tailwindcss": true
  }
}
```

{% endsamplefile %}

{% samplefile "tailwind.config.js" %}

```js
module.exports = {
  content: ["./src/*.{html,js}"],
  theme: {
    extend: {},
  },
  variants: {},
  plugins: [],
};
```

{% endsamplefile %}
{% endsample %}

### 默认插件

当在 `package.json` 中指定 `browserslist` 时，Parcel 会自动包含 `autoprefixer` 和 `postcss-preset-env` 的等效项。这些 [在 Rust 中实现](https://github.com/parcel-bundler/parcel-css) 比 PostCSS 快得多。如果这些是您项目中唯一需要的转换，那么您可能根本不需要 PostCSS。

如果您有一个现有项目，其 PostCSS 配置仅包含上述插件，您可以将其完全删除。如果您正在使用其他插件，则可以删除 `autoprefixer` 和 `postcss-preset-env`，同时仅保留自定义插件。这可以显着提高构建性能，因为 Parcel 的内置转译器比 PostCSS 快得多。

有关 Parcel 的内置转译支持的更多详细信息，请参阅 [above](#transpilation)。

### postcss-import

默认情况下，Parcel 使用 PostCSS 独立转换每个 CSS 文件。然而，一些 PostCSS 插件（例如`postcss-custom-properties`）可能需要访问来自其他`@import`ed CSS 资产的声明。

在这些情况下，您可以使用 [`postcss-import`](https://github.com/postcss/postcss-import) 在整个包上一次运行 PostCSS。 [`postcss-url`](https://github.com/postcss/postcss-url) 也应该用于确保在导入的文件被内联时正确解析 `url()` 引用。

{% sample %}
{% samplefile ".postcssrc" %}

```json
{
  "plugins": {
    "postcss-import": true,
    "postcss-url": true,
    "postcss-custom-properties": true
  }
}
```

{% endsamplefile %}
{% samplefile "app.css" %}

```css
@import "./config/index.css";

html {
  background-color: var(--varColor);
}

.icon {
  width: 50px;
  height: 50px;
  background-image: var(--varIcon);
}
```

{% endsamplefile %}
{% samplefile "config/index.css" %}

```css
:root {
  --varColor: red;
  --varIcon: url("../icon.svg");
}
```

{% endsamplefile %}
{% endsample %}

## Production

在生产模式下，Parcel 包括优化以减少代码的文件大小。有关其工作原理的更多详细信息，请参阅 [Production](/features/production/)。

### 缩小 Minification

在生产模式下，Parcel 会自动缩小您的代码以减小捆绑包的文件大小。默认情况下，Parcel 使用 [@parcel/css](https://github.com/parcel-bundler/parcel-css) 来执行 CSS 缩小。

{% warning %}

**注意**：在之前的版本中，Parcel 使用 [cssnano](http://cssnano.co/) 进行缩小。如果您的项目包含 cssnano 配置文件，例如 `.cssnanorc` 或 `cssnano.config.json`，您可能会看到升级 Parcel 后不再应用它的警告。

在大多数情况下，您可以简单地删除 cssnano 配置文件并允许 Parcel 处理缩小。但是，如果您确实依赖此配置中的某些设置并希望继续使用 cssnano 而不是 `@parcel/css` 进行缩小，则可以将 Parcel 配置为使用 `@parcel/optimizer-cssnano`。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "optimizers": {
    "*.css": ["@parcel/optimizer-cssnano"]
  }
}
```

{% endsamplefile %}
{% endsample %}

{% endwarning %}

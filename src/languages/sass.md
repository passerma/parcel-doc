---
layout: layout.njk
title: Sass
eleventyNavigation:
  key: languages-sass
  title: <img src="/assets/lang-icons/sass.svg" alt=""/> Sass
  order: 6
---

Parcel 使用 `@parcel/transformer-sass` 插件自动支持 [Sass](https://sass-lang.com/) 文件。当检测到 `.sass` 或 `.scss` 文件时，它会自动安装到您的项目中。

编译后的 Sass 文件也以与 [CSS](/languages/css/) 相同的方式处理，这意味着它是为您的浏览器目标编译的，并且任何 [PostCSS](/languages/css/#postcss) 插件也适用。 [CSS 模块](/languages/css/#css-modules) 也可以通过使用 `.module.scss` 扩展名命名您的文件来使用。

## 示例用法

在 HTML 文件中引用 SCSS 文件：

```html
<link rel="stylesheet" href="style.scss" />
```

在 JavaScript 或 TypeScript 中将 Sass/SCSS 文件作为 CSS 模块导入：

```js
import * as classes from "./style.module.scss";

document.body.className = classes.body;
```

使用 Parcel CLI 直接编译 Sass/SCSS：

```
parcel build style.scss
```

## 配置

要配置 Sass，请创建一个 `.sassrc` 或 `.sassrc.json` 文件。有关您可以在这些配置文件中定义的所有选项的列表，您可以查看官方 [Sass 文档](https://sass-lang.com/documentation/js-api#options)

{% warning %}

**注意**：基于 JavaScript 的配置也支持 `.sassrc.js`，但应尽可能避免，因为它会降低 Parcel 缓存的有效性。请改用基于 JSON 的配置格式（例如 `.sassrc.json`）。

{% endwarning %}

---
layout: layout.njk
title: Stylus
eleventyNavigation:
  key: languages-stylus
  title: <img src="/assets/lang-icons/stylus.svg" class="dark-invert" alt=""/> Stylus
  order: 7
---

Parcel 使用 `@parcel/transformer-stylus` 插件自动支持 [Stylus](https://stylus-lang.com/) 文件。当检测到 `.styl` 文件时，它会自动安装到您的项目中。

编译后的 Stylus 文件也以与 [CSS](/languages/css/) 相同的方式处理，这意味着它是为您的浏览器目标编译的，并且还应用了任何 [PostCSS](/languages/css/#postcss) 插件。 [CSS 模块](/languages/css/#css-modules) 也可以通过使用 `.module.styl` 扩展名命名您的文件来使用。

## 示例用法

在 HTML 文件中引用 Stylus 文件：

```html
<link rel="stylesheet" href="style.styl" />
```

在 JavaScript 或 TypeScript 中将 Stylus 文件作为 CSS 模块导入：

```js
import * as classes './style.module.styl';

document.body.className = classes.body;
```

使用 Parcel CLI 直接编译 Stylus

```
parcel build style.styl
```

## 配置

要配置 Stylus，请创建一个 `.stylusrc` 文件。要查看配置手写笔的可用选项，请参阅官方 [Stylus 文档](https://stylus-lang.com/docs/js.html)。

{% warning %}

**注意**：基于 JavaScript 的配置也支持 `.stylusrc.js`，但应尽可能避免，因为它会降低 Parcel 缓存的有效性。请改用基于 JSON 的配置格式（例如 `.stylusrc`）。

{% endwarning %}

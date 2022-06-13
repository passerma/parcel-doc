---
layout: layout.njk
title: Less
eleventyNavigation:
  key: languages-less
  title: <img src="/assets/lang-icons/less.svg" class="dark-invert" alt=""/> Less
  order: 8
---

Parcel 使用 `@parcel/transformer-less` 插件自动支持 [Less](https://lesscss.org/) 文件。当检测到 `.less` 文件时，它会自动安装到您的项目中。

编译的 Less 文件的处理方式也与 [CSS](/languages/css/) 相同，这意味着它是为您的浏览器目标编译的，并且任何 [PostCSS](/languages/css/#postcss) 插件也适用。 [CSS 模块](/languages/css/#css-modules) 也可以通过使用 `.module.less` 扩展名命名您的文件来使用。

## 示例用法

在 HTML 文件中引用 Less 文件：

```html
<link rel="stylesheet" href="style.less" />
```

在 JavaScript 或 TypeScript 中将 Less 文件作为 CSS 模块导入：

```js
import * as classes './style.module.less';

document.body.className = classes.body;
```

使用 Parcel CLI 直接编译 Less

```
parcel build style.less
```

## 配置

要配置 Less，请创建一个 `.lessrc` 文件。要查看配置 Less 的可用选项，请参阅官方 [Less 文档](http://lesscss.org/usage/#less-options)。

{% warning %}

**注意**：基于 JavaScript 的配置也支持 `.lessrc.js`，但应尽可能避免，因为它会降低 Parcel 缓存的有效性。请改用基于 JSON 的配置格式（例如 `.lessrc`）。

{% endwarning %}

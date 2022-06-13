---
layout: layout.njk
title: SugarSS
eleventyNavigation:
  key: languages-sugarss
  title: <img src="/assets/lang-icons/sugarss.svg" alt=""/> SugarSS
  order: 9
---

[SugarSS](https://github.com/postcss/sugarss) 是 [PostCSS](https://github.com/postcss/postcss) 的基于缩进的 CSS 语法。 Parcel 使用 `@parcel/transformer-sugarss` 插件自动支持 SugarSS。当检测到 `.sss` 文件时，它会自动安装到您的项目中。

编译后的 SugarSS 文件的处理方式也与 [CSS](/languages/css/) 相同，这意味着它是针对您的浏览器目标编译的，并且还应用了任何 [PostCSS](/languages/css/#postcss) 插件。 [CSS 模块](/languages/css/#css-modules) 也可以通过使用 `.module.sss` 扩展名命名您的文件来使用。

## 示例用法

在 HTML 文件中引用 SugarSS 文件：

```html
<link rel="stylesheet" href="style.sss" />
```

在 JavaScript 或 TypeScript 中将 SugarSS 文件作为 CSS 模块导入：

```js
import * as classes './style.module.sss';

document.body.className = classes.body;
```

使用 Parcel CLI 直接编译 SugarSS

```
parcel build style.sss
```

---
layout: layout.njk
title: Pug
eleventyNavigation:
  key: languages-pug
  title: <img src="/assets/lang-icons/pug.svg" alt=""/> Pug
  order: 15
---

[Pug](https://pugjs.org) 是一种可编译为 HTML 的模板语言。 Parcel 使用 `@parcel/transformer-pug` 插件自动支持 Pug。当检测到 `.pug` 文件时，它会自动安装到您的项目中。

Pug 被编译为 HTML 并按照 [HTML 文档](/languages/html/) 中的描述进行处理。

## 示例用法

```pug
doctype html
html(lang="en")
  head
    link(rel="stylesheet", href="style.css")
  body
    h1 Hello Pug!
    p.
      Pug is a terse and simple templating language with a
      strong focus on performance and powerful features.
    script(type="module", src="index.js")
```

Pug 可以像 HTML 一样用作 Parcel 的入口：

```shell
parcel index.pug
```

Pug 也可以在任何允许 URL 的地方引用，例如在 HTML 文件中，或从 JS 文件中。要将编译后的 HTML 内联到 JavaScript 文件中，请使用 `bundle-text:` 方案。有关详细信息，请参阅 [Bundle inlining](/features/bundle-inlining/)。

```js
import html from "bundle-text:./index.pug";

document.body.innerHTML = html;
```

## 配置

可以使用 `.pugrc`、`.pugrc.js` 或 `pug.config.js` 文件配置 Pug。有关可用选项的详细信息，请参阅 [Pug API 参考](https://pugjs.org/api/reference.html)。

{% warning %}

**注意：** `.pugrc.js` 和 `pug.config.js` 支持基于 JavaScript 的配置，但应尽可能避免，因为它们会降低 Parcel 缓存的有效性。请改用基于 JSON 的配置格式（例如 `.pugrc`）。

{% endwarning %}

### Locals

您可以在 Pug 配置中定义一个 `locals` 对象，这将在渲染时提供给您的 Pug 模板。

{% sample %}
{% samplefile "index.pug" %}

```pug
h1 Hello, #{name}!
```

{% endsamplefile %}
{% samplefile ".pugrc" %}

```json
{
  "locals": {
    "name": "Puggy"
  }
}
```

{% endsamplefile %}
{% endsample %}

---
layout: layout.njk
title: CoffeeScript
eleventyNavigation:
  key: languages-coffee
  title: <img src="/assets/lang-icons/coffeescript.svg" class="dark-invert" alt=""/> CoffeeScript
  order: 5
---

[CoffeeScript](https://coffeescript.org) 是一种转换为 JavaScript 的语言，它允许您使用更短的语法和其他功能，如 [存在运算符 existential operator](https://coffeescript.org/#existential-operator)、[较短的数组拼接语法 shorter array-splicing syntax](https://coffeescript.org/#slices)、[块正则表达式 block regular expressions](https://coffeescript.org/#regexes) 等等。

Parcel 使用 `@parcel/transformer-coffeescript` 插件自动支持 CoffeeScript。当检测到 `.coffee` 文件时，它会自动安装到您的项目中。

CoffeeScript 编译为 JavaScript 并按照 [JavaScript 文档](/languages/javascript/) 中的说明进行处理。

## 示例用法

{% sample %}
{% samplefile "index.html" %}

```html
<script type="module" src="app.coffee"></script>
```

{% endsamplefile %}
{% samplefile "app.coffee" %}

```coffeescript
console.log 'Hello world!'
```

{% endsamplefile %}
{% endsample %}

### URL 依赖项

在 JavaScript 文件中，可以使用 `URL` 构造函数结合 `import.meta.url` 创建 [URL 依赖项](/languages/javascript/#url-dependencies)。这可用于引用图像、[workers](/languages/javascript/#workers)、[service workers](/languages/javascript/#service-workers) 等 URL。

CoffeeScript 目前不支持 `import.meta`。相反，您可以使用带有 `file:` 前缀的 CommonJS `__filename` 变量将其转换为 URL。例如，以下是在 CoffeeScript 中创建工作者的方法：

```coffeescript
new Worker new URL('worker.js', 'file:' + __filename),
  type: 'module'
```

其他类型的依赖项（例如图像）也是如此：

```coffeescript
img = document.createElement 'img'
img.src = new URL 'hero.jpg', 'file:' + __filename
document.body.appendChild img
```

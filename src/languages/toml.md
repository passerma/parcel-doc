---
layout: layout.njk
title: TOML
eleventyNavigation:
  key: languages-toml
  title: <img src="/assets/lang-icons/toml.svg" alt=""/> TOML
  order: 12
---

Parcel 支持使用 `@parcel/transformer-toml` 插件从 JavaScript 导入 TOML 文件。 当检测到 `.toml` 文件时，它会自动安装到您的项目中。

## 示例用法

{% sample %}
{% samplefile "app.js" %}

```js
import data from "./data.toml";
console.log(data.hello[0]);
// => "world"
```

{% endsamplefile %}
{% samplefile "data.toml" %}

```toml
hello = [
  "world",
  "computer"
]
```

{% endsamplefile %}
{% endsample %}

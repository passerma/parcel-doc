---
layout: layout.njk
title: JSON
eleventyNavigation:
  key: languages-json
  title: <img src="/assets/lang-icons/json.svg" class="dark-invert" alt=""/> JSON
  order: 11
---

Parcel 支持将 JSON 和 [JSON5](https://json5.org) 文件导入到 JavaScript 中。

## 示例用法

{% sample %}
{% samplefile "app.js" %}

```js
import data from "./data.json";
console.log(data.hello[0]);
// => "world"
```

{% endsamplefile %}
{% samplefile "data.json" %}

```json
{
  "hello": ["world", "computer"]
}
```

{% endsamplefile %}
{% endsample %}

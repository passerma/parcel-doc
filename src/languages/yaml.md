---
layout: layout.njk
title: YAML
eleventyNavigation:
  key: languages-yaml
  title: <img src="/assets/lang-icons/yaml.svg" class="dark-invert" alt=""/> YAML
  order: 13
---

Parcel 支持使用 `@parcel/transformer-yaml` 插件从 JavaScript 导入 YAML 文件。 当检测到 `.yaml` 文件时，它会自动安装到您的项目中。

## 示例用法

{% sample %}
{% samplefile "app.js" %}

```js
import data from "./data.yaml";
console.log(data.hello[0]);
// => "world"
```

{% endsamplefile %}
{% samplefile "data.yaml" %}

```yaml
hello:
  - world
  - computer
```

{% endsamplefile %}
{% endsample %}

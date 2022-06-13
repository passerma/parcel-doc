---
layout: layout.njk
title: MDX
eleventyNavigation:
  key: languages-mdx
  title: <img src="/assets/lang-icons/mdx.svg" alt=""/> MDX
  order: 16
---

[MDX](https://mdxjs.com) 是 [Markdown](https://daringfireball.net/projects/markdown/) 的变体，编译为 JSX，支持在 Markdown 文档中嵌入交互组件。 Parcel 使用 `@parcel/transformer-mdx` 插件自动支持 MDX。当检测到 `.mdx` 文件时，它会自动安装到您的项目中。

## 示例用法

首先，安装`@mdx-js/react`。这是将 MDX 文件呈现为 React 组件所必需的。

```shell
yarn add @mdx-js/react@^1
```

然后，您可以将 `.mdx` 文件导入 JavaScript 并使用 React 渲染它：

{% sample %}
{% samplefile "app.js" %}

```js
import Hello from "./hello.mdx";

export function App() {
  return <Hello />;
}
```

{% endsamplefile %}
{% samplefile "hello.mdx" %}

```md
# Hello, MDX!

This is a pretty cool MDX file.
```

{% endsamplefile %}
{% endsample %}

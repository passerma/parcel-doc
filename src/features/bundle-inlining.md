---
layout: layout.njk
title: 打包内联
eleventyNavigation:
  key: features-bundle-inlining
  title: 🪆 打包内联(Bundle inlining)
  order: 4
---

Parcel 包含多种将一个包的已编译内容内联到另一个包中的方法。

## 将包内联为文本

`bundle-text:`方案可用于将打包的内容作为纯文本内联。Parcel 将正常编译解析的文件，包括打包所有依赖项，然后将结果作为字符串内联到父包中。

这可以以多种方式使用。例如，您可以内联已编译的 CSS 包并使用结果在运行时注入样式标签。这在您需要控制样式标签插入位置的情况下可能很有用，例如插入到 shadowRoot 中。

```javascript
import cssText from "bundle-text:./test.css";

// inject <style> tag
let style = document.createElement("style");
style.textContent = cssText;
shadowRoot.appendChild(style);
```

## 内联作为数据 URL

`data-url:`方案允许将打包内联为数据 URL。解析的文件将被编译，包括所有依赖项，并转换为数据 URL。如果文件是二进制格式，它将被编码为 base 64，否则为 URI。

这可能有用的一个示例是在 CSS 文件中内联小图像。

```css
.foo {
  background: url(data-url:./background.png);
}
```

## 底层实现

`bundle-text:`和`data-url:`使用[Named pipelines](/features/plugins/#named-pipelines)在默认 Parcel 配置中实现。`@parcel/transformer-inline-string`[Transformer](/plugin-system/transformer/)插件将编译后的资产标记为内联，这告诉 Parcel 不要将包写入磁盘，而是将其内联到父包中。为了实现数据 URL，`@parcel/optimizer-data-url` [Optimizer](/plugin-system/optimizer/)插件用于将编译的包转换为数据 URL。

在 Parcel 配置中，它如下所示。每个`"..."`管道中的 告诉 Parcel 先运行与文件匹配的常用转换器，然后运行`@parcel/transformer-inline-string`。

{% sample %}
{% samplefile "@parcel/config-default" %}

```json
{
  "transformers": {
    "bundle-text:*": ["...", "@parcel/transformer-inline-string"],
    "data-url:*": ["...", "@parcel/transformer-inline-string"]
  },
  "optimizers": {
    "data-url:*": ["...", "@parcel/optimizer-data-url"]
  }
}
```

{% endsamplefile %}
{% endsample %}

您可以创建自己的命名管道来自定义内联，但您可以重用上述插件或创建自定义插件。有关详细信息，请参阅[Parcel configuration](/features/plugins/)。

另一个可能有用的 Parcel 插件是`@parcel/transformer-inline`。像`@parcel/transformer-inline-string`一样，它将资产标记为内联，但结果未编码为字符串。这意味着如果内联包包含代码，它将在父包中执行，而不是向用户返回字符串。如果您有一个以某种方式包装打包并需要在运行时对其进行解码的自定义插件，这可能会很有用。

例如，您可能希望将文件内联为[ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)或其他一些自定义编码。这可以使用自定义优化器插件来实现，该插件对包的输出进行后处理。

```javascript
import { Optimizer } from "@parcel/plugin";
import { blobToBuffer } from "@parcel/utils";

export default new Optimizer({
  async optimize({ contents }) {
    let buffer = await blobToBuffer(contents);
    return {
      contents: `new Uint8Array(${JSON.stringify(Array.from(buffer))}).buffer`,
    };
  },
});
```

现在您可以使用新插件定义命名管道，并将编译后的文件作为数组缓冲区导入。

有关编写自定义插件的更多详细信息，请参阅[插件系统 Plugin system](/plugin-system/overview/)和[parcel 配置 Parcel Configuration](/features/plugins/)。

## 内联为 blob URL

您可能希望将包的内容内联为[blob URL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL)，它可以传递给浏览器中的许多 Web API。该`@parcel/optimizer-blob-url`插件可用于执行此操作，组合`@parcel/transformer-inline`。默认情况下不包含这些命名管道，因此您需要定义在`.parcelrc`。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "transformers": {
    "blob-url:*": ["...", "@parcel/transformer-inline"]
  },
  "optimizers": {
    "blob-url:*": ["...", "@parcel/optimizer-blob-url"]
  }
}
```

{% endsamplefile %}
{% endsample %}

## 内联而不转换

在 JavaScript 中，可以在不首先通过 Parcel 转换器运行文件的情况下内联文件的内容。这可以使用`fs`Parcel 静态分析的 Node 模块来完成。它可以内联为多种不同编码的字符串，也可以内联为[Buffer](https://nodejs.org/api/buffer.html)。有关更多详细信息，请参阅[Node emulation](/features/node-emulation/)。

```javascript
import fs from "fs";

const sourceCode = fs.readFileSync(__dirname + "/foo.js", "utf8");
```

在上面的例子中，`sourceCode`变量将是`foo.js`未经编译的内容，即原始源代码而不是打包后的结果。

## 与其他工具集成

由于包内联是 Parcel 特有的功能，因此您需要配置其他工具（例如 TypeScript 或 Flow）来支持它。有关如何执行此操作的详细信息，请参阅依赖项解析文档中的[配置其他工具部分 Configuring other tools](/features/dependency-resolution/#configuring-other-tools)。

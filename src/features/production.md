---
layout: layout.njk
title: 生产环境
eleventyNavigation:
  key: features-production
  title: 🚀 生产环境(Production)
  order: 6
---

Parcel 的生产环境会自动捆绑和优化您的生产应用程序。它可以使用以下命令`parcel build`运行：

```shell
parcel build src/index.html
```

## 尺寸优化

Parcel 包含许多旨在减少包大小的优化，包括自动缩小、tree shaking，图像优化等。

### 缩小

Parcel 包括开箱即用的 JavaScript、CSS、HTML 和 SVG 缩小器。缩小通过删除空格、将变量重命名为更短的名称以及许多其他优化来减小输出包的文件大小。

默认情况下，使用`parcel build`命令时会启用缩小。如果需要，您可以使用`--no-optimize`CLI 标志来禁用缩小和其他优化。

Parcel 使用[terser](https://github.com/fabiosantoscode/terser)来缩小 JavaScript，[@parcel/css](https://github.com/parcel-bundler/parcel-css)用于 CSS，[htmlnano](https://github.com/posthtml/htmlnano)用于 HTML，[svgo](https://github.com/svg/svgo)用于 SVG。如果需要，您可以使用`.terserrc`、`.htmlnanorc`或`svgo.config.json`config 文件配置。有关更多详细信息，请参阅[JavaScript](/languages/javascript/), [CSS](/languages/css/), [HTML](/languages/html), 和 [SVG](/languages/svg/)。

### Tree shaking

在生产构建中，Parcel 静态分析每个模块的导入和导出，并删除所有未使用的内容。这称为"tree shaking"或"dead code elimination"。静态和动态`import()`,、CommonJS 和 ES 模块甚至跨语言与 CSS 模块都支持 tree shaking。

Parcel 还尽可能将模块连接到单个作用域中，而不是将每个模块包装在单独的函数中。这称为“scope hoisting”。这有助于使缩小更有效，并通过使模块之间的引用静态而不是动态对象查找来提高运行时性能。

有关 tree shaking 更有效的提示，请参阅[Scope hoisting](/features/scope-hoisting/)。

### 开发分支移除

`parcel build`自动将`NODE_ENV`环境变量设置为`production`。此环境变量通常在库中用于启用仅开发调试功能，可以在生产构建中剥离这些功能以减少包大小。Parcel 内联此环境变量并优化比较以删除死分支。

您也可以在自己的代码中利用此功能。例如，您可以使用 if 语句来检查`NODE_ENV`环境变量。

```javascript
if (process.env.NODE_ENV !== "production") {
  // Only runs in development and will be stripped in production builds.
}
```

有关环境变量内联的更多详细信息，请参阅[节点仿真 Node emulation docs](/features/node-emulation/)。

### 图像优化

Parcel 支持调整图像大小、转换和优化图像。在 HTML、CSS 或 JavaScript 中引用图像时，您可以使用查询参数来指定图像应转换为的格式和大小。您可以从同一源图像请求多种尺寸或格式，这有助于有效地支持不同类型的设备或浏览器。

```html
<picture>
  <source
    type="image/webp"
    srcset="image.jpg?as=webp&width=400, image.jpg?as=webp&width=800 2x"
  />
  <source
    type="image/jpeg"
    srcset="image.jpg?width=400, image.jpg?width=800 2x"
  />
  <img src="image.jpg?width=400" width="400" />
</picture>
```

调整图像大小和转换图像在开发和生产模式下都会发生，因此您也可以使用正确的图像尺寸和格式进行测试。有关更多详细信息，请参阅[图像转换 Image transformer](/recipes/image/)文档。

arcel 还包括在生产模式下默认对 JPEG 和 PNG 进行无损图像优化，这可以在不影响图像质量的情况下减小图像的大小。这不需要使用任何查询参数或配置。但是，由于优化是无损的，因此与使用查询参数`quality`或使用 WebP 或 AVIF 等现代格式相比，可能减少的大小可能更少。

### 差分打包

Parcel 会自动生成`<script type="module">`现代 JavaScript 语法，并`<script nomodule>`在必要时为旧版浏览器提供备用。通过避免转换类、异步/等待等功能，这为大多数用户减少了包大小。有关更多详细信息，请参阅 Targets 文档中的[差异捆绑 Differential bundling](/features/targets/#differential-bundling)。

### 压缩

Parcel 支持使用[Gzip](https://en.wikipedia.org/wiki/Gzip)和[Brotli](https://en.wikipedia.org/wiki/Brotli)压缩包。虽然许多服务器即时压缩数据，但其他服务器要求您提前上传预压缩的有效负载。这也可能允许更好的压缩，这对于每个网络请求来说都太慢了。

因为不是每个人都需要它，所以默认情况下不启用压缩。要启用它，请将`@parcel/compressor-gzip`或`@parcel/compressor-brotli`添加到您的`.parcelrc`。

```shell
yarn add @parcel/compressor-gzip @parcel/compressor-brotli --dev
```

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "compressors": {
    "*.{html,css,js,svg,map}": [
      "...",
      "@parcel/compressor-gzip",
      "@parcel/compressor-brotli"
    ]
  }
}
```

{% endsamplefile %}
{% endsample %}

现在，您将在原始未压缩包旁边获得一个`.gz`和`.br`文件。如果基于文本的文件类型比上面示例中列出的要多，则需要相应地扩展 glob。

如果您不需要未压缩的包，您也可以从上面的示例中删除`"..."`以*仅*输出压缩文件。例如，要仅输出一个`.gz`文件，您可以使用以下配置：

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "compressors": {
    "*.{html,css,js,svg,map}": ["@parcel/compressor-gzip"]
  }
}
```

{% endsamplefile %}
{% endsample %}

## 缓存优化

Parcel 包含与浏览器和 CDN 缓存相关的多项优化，包括内容散列、打包清单和共享打包。

### 内容散列

Parcel 会自动在所有输出文件的名称中包含内容哈希，从而实现长期浏览器缓存。每当包的内容发生变化时，文件名中包含的哈希值就会更新，从而触发 CDN 和浏览器缓存的失效。

默认情况下，所有捆绑包都包含内容哈希，但条目和某些需要名称稳定的依赖类型除外。例如，服务工作者需要一个稳定的文件名才能正常工作，并且 HTML 中的`<a>`标签引用用户可读的 URL。

您还可以使用`--no-content-hash`CLI 标志禁用内容散列。请注意，名称仍将包含哈希，但不会在每次构建时更改。您可以使用[Namer](/plugin-system/namer/)插件完全自定义包命名。

### 级联失效 Cascading invalidation

Parcel 在每个条目包中使用清单来避免在许多情况下[cascading invalidation](https://philipwalton.com/articles/cascading-cache-invalidation/)问题。此清单包括稳定包 ID 到最终内容散列文件名的映射。当一个包需要引用另一个包时，它使用包 id 而不是内容散列名称。这意味着当一个包更新时，只有那个包和条目需要在浏览器缓存中失效，中间包不会改变。这提高了跨部署的缓存命中率。

### 共享打包

在生产构建中，Parcel 会自动优化应用程序中的打包图，以减少重复并提高可缓存性。当您的应用程序的多个部分依赖于相同的公共模块时，它们会自动去重到一个单独的包中。这允许常用的依赖项与您的应用程序代码并行加载，并由浏览器单独缓存。

例如，如果您的应用程序中的多个页面依赖于`react`和`lodash`，它们可能会被移动到一个单独的包中，而不是在每个页面中重复。这样，当用户从一个页面导航到另一个页面时，他们只需要下载该页面的附加代码，而不是重新下载那些已经缓存的库。

有关如何配置的更多详细信息，请参阅[代码拆分 Code splitting](/features/code-splitting/)。

## 分析打包大小

Parcel 包含一些工具来帮助您分析打包的大小。

### 详细报告

默认情况下，Parcel 在为生产构建时会在终端中输出打包报告。它包括每个输出包的大小和构建时间。要查看有关组成每个包的文件的更多详细信息，您可以使用`--detailed-report`CLI 选项。默认情况下，每个包中最多显示 10 个文件，按大小排序。您也可以传递一个数字来增加它，例如`--detailed-report 20`。

### 打包分析器

`@parcel/reporter-bundle-analyzer`插件可用于生成包含树形图的 HTML 文件，该树形图直观地显示每个包中每个资产的相对大小。您可以使用`--reporter`CLI 选项运行它。

```shell
parcel build src/index.html --reporter @parcel/reporter-bundle-analyzer
```

这会在您的项目根目录中生成一个文件夹`parcel-bundle-reports`，其中包含每个目标的 HTML 文件：

<div style="border: 1px solid black">

![A screenshot of the bundle analyzer output](/assets/bundle-analyzer.png)

</div>

如果您想在每个构建中自动运行打包分析器，您也可以将`"reporters"`添加到`.parcelrc`文件中。

### Bundle Buddy

`@parcel/reporter-bundle-buddy`插件可用于生成与[Bundle Buddy](https://bundle-buddy.com)兼容的报告。您可以使用`--reporter`CLI 选项运行它。

```shell
parcel build src/index.html --reporter @parcel/reporter-bundle-buddy
```

现在将`dist`录中的文件上传到[Bundle Buddy website](https://bundle-buddy.com/parcel).

<div style="border: 1px solid black">

![A screenshot of the Bundle Buddy website with a loaded project](/assets/bundle-buddy.png)

</div>

---
layout: layout.njk
title: 图片 Image
eleventyNavigation:
  key: recipes-image
  title: <img src="/assets/lang-icons/image.svg" alt=""/> 图片 Image
  order: 2
---

Parcel 内置了对调整大小、转换和优化图像的支持。可以从 HTML、CSS、JavaScript 或任何其他文件类型引用图像。

## 调整图像大小和转换图像

Parcel 包含一个开箱即用的图像转换器，它允许您调整图像大小、将它们转换为不同的格式或调整质量以减小文件大小。这可以在引用图像时使用查询参数或使用[configuration](#configuration)文件来完成。

图像转换器依赖于[Sharp](https://sharp.pixelplumbing.com/)图像转换库，它会在需要时作为开发依赖项自动安装到您的项目中。

您可以使用的查询参数是：

- `width` – 调整图像大小的宽度
- `height` – 调整图像大小的高度
- `quality` – 您想要的图像质量百分比，例如`?quality=75`
- `as` – 将图像转换为的文件格式，例如：`?as=webp`

### 图像格式

支持以下图像格式，通过`as`查询参数作为输入和输出：

- `jpeg` / `jpg` - [JPEG](https://en.wikipedia.org/wiki/JPEG)是一种被广泛支持的有损图像格式。它通常用于照片，并提供相当好的压缩，但不支持透明度或无损压缩。
- `png` - [Portable Network Graphics](https://en.wikipedia.org/wiki/Portable_Network_Graphics) (PNG)是一种无损图像格式。PNG 通常比 JPEG 或其他有损图像格式大得多，但支持透明度并为精细细节提供更高的质量。
- `webp` – [WebP](https://en.wikipedia.org/wiki/WebP)持有损和无损压缩以及动画和透明度。它在所有现代浏览器中都[支持 supported](https://caniuse.com/webp)，并提供更好的压缩效果，与 JPEG 和 PNG 具有相同的质量。
- `avif` – [AVIF](https://jakearchibald.com/2020/avif-has-landed/)是一种基于 AV1 视频编解码器的新型有损图像格式，与 JPEG 和 WebP 相比，它提供了显着的压缩和质量改进。目前最新版本的 Chrome 和 Firefox 都[支持 supported](https://caniuse.com/avif)。

以下格式也支持作为输入，但浏览器通常不支持：`tiff`, `heic` / `heif`, 和 `raw`。

如果您设置[setup a custom libvips build](https://github.com/lovell/sharp/issues/2437)，也支持 GIF ，但是，由于文件很大，不鼓励使用 GIF。[请改用视频格式 Use a video format instead](https://web.dev/replace-gifs-with-videos/)。

有关选择正确图像格式的更多指导，请参阅[web.dev](https://web.dev/choose-the-right-image-format/)上的指南。

### JavaScript

要从 JavaScript 引用图像，请使用`URL`构造函数。有关更多详细信息，请参阅 JavaScript 文档中的[URL dependencies](/languages/javascript/#url-dependencies)。

{% sample %}
{% samplefile "main.js" %}

```js
const imageUrl = new URL("image.jpeg?as=webp&width=250", import.meta.url);
```

{% endsamplefile %}
{% endsample %}

### HTML

要从 HTML 引用图像，请使用`<img>`或`<picture>`元素。可以使用不同的查询参数多次引用同一图像，以创建不同格式或大小的多个版本。有关更多详细信息，请参阅[HTML docs](/languages/html/#images)。

{% sample %}
{% samplefile "index.html" %}

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>HTML Example</title>
  </head>
  <body>
    <picture>
      <source srcset="image.jpeg?as=avif&width=800" type="image/avif" />
      <source srcset="image.jpeg?as=webp&width=800" type="image/webp" />
      <source srcset="image.jpeg?width=800" type="image/jpeg" />
      <img src="image.jpeg?width=200" alt="test image" />
    </picture>
  </body>
</html>
```

{% endsamplefile %}
{% endsample %}

## 配置 Configuration

除了查询参数，Parcel 还支持使用配置文件来定义适用于项目中所有图像的选项。例如，您可以以特定质量设置重新编码所有图像以减小文件大小，或为每种输出格式定义更高级的选项。

要设置项目中所有图像的质量，请在项目中创建一个`sharp.config.json`文件并定义`quality`字段。这将重新编码每个图像，而不仅仅是查询参数引用的图像。

{% sample %}
{% samplefile "sharp.config.json" %}

```json
{
  "quality": 80
}
```

{% endsamplefile %}
{% endsample %}

您还可以为每种格式定义更高级的选项。所有格式中定义了选项的图像都 sharp.config.json 将被重新编码。[在此处](https://sharp.pixelplumbing.com/api-output#jpeg)查看支持选项的完整列表。

{% sample %}
{% samplefile "sharp.config.json" %}

```json
{
  "jpeg": {
    "quality": 75,
    "chromaSubsampling": "4:4:4"
  },
  "webp": {
    "nearLossless": true
  },
  "png": {
    "palette": true
  }
}
```

{% endsamplefile %}
{% endsample %}

## 图像优化 Image optimization

在生产模式下，Parcel 还默认为 jpeg 和 PNGs 提供无损图像优化，这样可以在不影响图像质量的情况下减小图像的尺寸。这不需要使用任何查询参数或配置。但是，由于优化是无损耗的，因此可能的大小缩减可能比使用`quality`查询参数或使用诸如 WebP 或 AVIF 之类的现代格式要小。

## 禁用图像优化

要在生产模式下禁用 JPEG 和 PNG 的默认图像优化，请将以下内容添加到您的 .parcelrc 配置文件中：

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "optimizers": {
    "*.{jpg,jpeg,png}": []
  }
}
```

{% endsamplefile %}
{% endsample %}

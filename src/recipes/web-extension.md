---
layout: layout.njk
title: Web 扩展 Web Extension
eleventyNavigation:
  key: recipes-webext
  title: <img class="dark-invert" src="/assets/lang-icons/webext.svg" alt=""/> Web 扩展 Web Extension
  order: 7
---

[Web 扩展 Web Extensions](https://developer.chrome.com/docs/extensions/) 是一组 API，用于构建可跨多种浏览器运行的浏览器扩展。 Parcel 支持使用 `@parcel/config-webextension` 构建 Web 扩展。

## 入门

首先，将 `@parcel/config-webextension` 安装到您的项目中：

```shell
yarn add @parcel/config-webextension --dev
```

接下来，您需要一个 [manifest.json](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json) 文件，它是你的分机。有关如何设置的详细信息，请参阅 [本指南](https://developer.chrome.com/docs/extensions/mv3/getstarted/)。支持清单 V2 和 V3。您可以在 Web 扩展代码中使用 [TypeScript](/languages/typescript)、[Vue](/languages/vue) 以及 Parcel 支持的任何其他语言。

{% sample %}
{% samplefile "manifest.json" %}

```json
{
  "manifest_version": 3,
  "name": "Sample Web Extension",
  "version": "0.0.1",
  "background": {
    "service_worker": "background.ts",
    "type": "module"
  },
  "content_scripts": [
    {
      "matches": ["*://github.com/parcel-bundler/*"],
      "js": ["parcel-content-script.ts"]
    }
  ]
}
```

{% endsamplefile %}
{% endsample %}

要构建您的扩展程序，请使用您的 `manifest.json` 作为入口运行 Parcel，并使用 `@parcel/config-webextension` 作为配置：

```shell
parcel build manifest.json --config @parcel/config-webextension
```

你也可以在你的项目中创建一个扩展 `@parcel/config-webextension` 的 `.parcelrc` 文件。这样您就不需要每次都将 `--config` 选项传递给 Parcel CLI。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-webextension"
}
```

{% endsamplefile %}
{% endsample %}

## HMR

由于 MV3 中的[内容安全政策限制 restrictions on Content Security Policy](https://developer.chrome.com/docs/extensions/mv3/intro/mv3-migration/#content-security-policy)，不支持 HMR，但更新您的代码将导致扩展重新加载。对于 MV2，默认情况下完全支持 HMR。使用内容脚本重新加载页面将在两个版本中重新加载扩展。

为了获得最佳的开发人员体验，请使用 `--host localhost` 进行开发构建（这有时是重新加载内容脚本所必需的）。您可以复制以下配置：

{% sample %}
{% samplefile "package.json" %}

```json
{
  "scripts": {
    "start": "parcel watch src/manifest.json --host localhost --config @parcel/config-webextension",
    "build": "parcel build src/manifest.json --config @parcel/config-webextension"
  }
}
```

{% endsamplefile %}
{% endsample %}

运行 `yarn start` 或 `npm start` 将启动开发服务器。Source maps 和 HMR 将适用于后台脚本、弹出页面和选项页面。对于 MV2，HMR 通常也适用于内容脚本。

要将扩展添加到您的浏览器，请解压加载 Parcel 的输出文件夹。例如，在 Chrome 中，在 `chrome://extensions` 页面中[单击“Load Unpacked”](https://developer.chrome.com/extensions/getstarted#manifest) 并选择 `path/to/project/dist `。

运行 `yarn build` 或 `npm run build` 将为您提供最终的 Web 扩展包，准备发布。压缩输出目录后，您应该能够将文件上传到您选择的平台，例如 Chrome 网上应用店。

## 特别注意事项

### 意外消息 Unexpected messages

在开发模式下，每当重新加载内容脚本页面时，您的后台脚本都会收到内容为 `{ __parcel_hmr_reload__: true }` 的消息事件。 Parcel 将在必要时自动使用它来刷新扩展。因此，您需要确保后台脚本收到的任何消息在处理它们之前不具有 `__parcel_hmr_reload__` 属性。

### Styling

在内容脚本中导入的任何样式都将被注入到该内容脚本的“css”属性中，因此将应用于整个页面。通常这是您想要的，但如果不是，您始终可以使用 [CSS modules](/languages/css#css-modules) 来阻止样式应用于原始站点。

此外，内容脚本 CSS 会解析指向它们被注入的站点的链接，因此您将无法引用本地资产。你应该 [inline your bundles](</languages/css#url()>) 来解决这个问题。

{% sample %}
{% samplefile "content-script.css" %}

```css
.my-class {
  /* Equivalent to: https://injected-site.com/custom-bg.png */
  /* This is probably not what you want! */
  background-image: url(./custom-bg.png);
}

.my-other-class {
  /* This will use the local file custom-bg.png */
  background-image: url(data-url:./custom-bg.png);
}
```

{% endsamplefile %}
{% endsample %}

最后，当从内容脚本的`import()`中添加或删除链接的 CSS 时，热重装可能无法工作，而同步`import`没有这样的问题。这是一个已知的限制，将在未来的版本中予以修正。

### `web_accessible_resources`

你在内容脚本中使用的任何资源都会自动添加到`web_accessible_resources`中，所以你通常不需要在`web_accessible_resources`中指定任何内容。例如，下面的内容脚本可以正常工作:

{% sample %}
{% samplefile "content-script.js" %}

```js
import myImage from "url:./image.png";

const injectedImage = document.createElement("img");
injectedImage.src = myImage;
document.body.appendChild(injectedImage);
```

{% endsamplefile %}
{% endsample %}

但是，如果您确实希望其他扩展程序或网站可以访问您的扩展程序中的资源，您可以在 `web_accessible_resources` 中指定文件路径或 glob。请注意，Parcel 将 `web_accessible_resources` 中的条目视为 Unix glob（例如，`examples/*.png` 将检索示例文件夹中的每个 PNG，而 `examples/**.png` 将递归执行此操作）。这与 Chrome 中的 globbing 不同，后者始终是递归的。

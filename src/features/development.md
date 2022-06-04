---
layout: layout.njk
title: 开发
eleventyNavigation:
  key: features-development
  title: 🧑‍💻 开发
  order: 1
---

Parcel 包括一个开箱即用的开发服务器，支持热重载、HTTPS、API 代理等。

## Dev server

Parcel 的内置开发服务器会在您运行默认命令 `parcel` 时自动启动，该命令是 `parcel serve` 的缩写。 默认情况下，他启动在 [http://localhost:1234](http://localhost:1234). 如果端口 `1234` 已在使用中，则将使用备用端口。Parcel 启动后，开发服务器正在监听的位置将打印到终端。

开发服务器支持多个选项，您可以通过 CLI 选项指定：

- `-p`, `--port` – 覆盖默认端口。 `PORT` 环境变量也可以用来设置端口。
- `--host` – 默认情况下，开发服务器接受所有接口上的连接。您可以覆盖它以指定仅应接受来自某些主机的连接。
- `--open` – Parcel 启动后自动在默认浏览器中打开。您还可以传递浏览器名称来打开不同的浏览器，例如 `--open safari`.

## 热重载

当您更改代码时，Parcel 会自动重建更改的文件并在浏览器中更新您的应用程序。默认情况下，Parcel 会完全重新加载页面，但在某些情况下，它可能会执行热模块更换 (HMR)。HMR 通过在运行时更新浏览器中的模块来改善开发体验，而无需刷新整个页面。这意味着当您更改代码中的小事情时，可以保留应用程序状态。

CSS 更改通过 HMR 自动应用，无需重新加载页面。在使用内置 HMR 支持的框架时也是如此，例如 React（通过快速刷新）和 Vue。

如果您不使用框架，则可以使用 `module.hot` API 选择加入 HMR。这将防止页面重新加载，而是就地应用更新。`module.hot` 仅在开发中可用，因此您需要在使用之前检查它是否存在。

```javascript
if (module.hot) {
  module.hot.accept();
}
```

HMR 的工作原理是替换模块的代码，然后重新评估它以及它的所有父模块。如果你需要自定义这个过程，你可以使用 `module.hot.accept` 和 `module.hot.dispose` 方法来挂钩它。这些使您可以在新版本的模块中保存和恢复状态。

`module.hot.dispose` 接受一个回调，该回调在该模块即将被替换时调用。使用它来保存任何状态以在提供的对象中恢复模块的新版本 `data` ，或清理将在新版本中重新创建的计时器等内容。

`module.hot.accept` 接受一个回调函数，该函数在该模块或其任何依赖项更新时执行。您可以使用它来使用存储在 `module.hot.data`.

```javascript
if (module.hot) {
  module.hot.dispose(function (data) {
    // module is about to be replaced.
    // You can save data that should be accessible to the new asset in `data`
    data.updated = Date.now();
  });

  module.hot.accept(function (getParents) {
    // module or one of its dependencies was just updated.
    // data stored in `dispose` is available in `module.hot.data`
    let { updated } = module.hot.data;
  });
}
```

## 开发目标

使用开发服务器时，一次只能构建一个目标。默认情况下，Parcel 使用支持现代浏览器的开发目标。这意味着旧浏览器的现代 JavaScript 语法转换被禁用。

如果您需要在旧版浏览器中进行测试，您可以提供 `--target` CLI 选项来选择要构建的目标。例如，要构建 package.json 中定义的“legacy”目标，请使用 `--target legacy`。如果您没有定义任何显式目标，并且 `browserslist` 在您的 package.json 中只有一个，您可以将隐式默认目标与 `--target default`。这将导致您的源代码被编译，就像它在生产中一样。

有关详细信息，请参阅 [Targets](/features/targets/) 文档。

## 懒加载模式

在开发中，在开发服务器启动之前等待整个应用程序构建可能会令人沮丧。在处理具有许多页面的大型应用程序时尤其如此。如果您只开发一项功能，则无需等待所有其他功能构建完成，除非您导航到它们。

您可以使用 `--lazy` CLI 标志告诉 Parcel 将构建文件推迟到浏览器中请求它们之前，这可以显着减少开发构建时间。服务器启动很快，当您第一次导航到某个页面时，Parcel 仅构建该页面所需的文件。当您导航到另一个页面时，该页面将按需构建。如果您导航回之前构建的页面，它会立即加载。

```shell
parcel 'pages/*.html' --lazy
```

这也适用于动态 `import()`，而不仅仅是单独的某个入口。因此，如果您的页面具有动态加载的功能，则该功能在被激活之前不会被构建。当它被请求时，Parcel 也会急切地构建所有依赖项，而无需等待它们被请求。

## 缓存

Parcel 将它构建的所有内容缓存到磁盘。如果您重新启动开发服务器，Parcel 将仅重建自上次运行以来已更改的文件。Parcel 会自动跟踪构建中涉及的所有文件、配置、插件和开发依赖项，并在发生更改时精细地使缓存无效。例如，如果您更改配置文件，所有依赖该配置的源文件都将重新构建。

默认情况下，缓存存储在项目内的文件夹 `.parcel-cache` 中。您应该将此文件夹添加到您的 `.gitignore` （或其他的配置）中，以便它不会在您的存储库中提交。您还可以使用 `--cache-dir` CLI 选项覆盖缓存的位置。

也可以使用标志 `--no-cache` 禁用缓存。请注意，这只会禁用从缓存中读取.parcel-cache-仍将创建一个文件夹。

## HTTPS

有时，您可能需要在开发过程中使用 HTTPS。例如，您可能需要使用某个主机名进行身份验证 cookie，或调试混合内容问题。Parcel 的开发服务器支持开箱即用的 HTTPS。您可以使用自动生成的证书，也可以提供自己的证书。

要使用自动生成的自签名证书，请使用 `--https` CLI 标志。首次加载页面时，您可能需要在浏览器中手动信任此证书。

```shell
parcel src/index.html --https
```

要使用自定义证书，您需要使用 `--cert` 和 `--key` CLI 选项分别指定证书文件和私钥。

```shell
parcel src/index.html --cert certificate.cert --key private.key
```

## API 代理

为了在开发 Web 应用程序时更好地模拟实际生产环境，您可以在 `.proxyrc.json` 或 `.proxyrc` 或 `.proxyrc.js` 文件中指定应该代理到另一台服务器（例如您的真实 API 服务器或本地测试服务器）的路径。

### `.proxyrc` / `.proxyrc.json`

在此 JSON 文件中，您指定一个对象，其中每个键都是匹配 URL 的模式，值是 [`http-proxy-middleware`对象](https://github.com/chimurai/http-proxy-middleware#options):

{% sample %}
{% samplefile ".proxyrc" %}

```js
{
  "/api": {
    "target": "http://localhost:8000/",
    "pathRewrite": {
      "^/api": ""
    }
  }
}

```

{% endsamplefile %}
{% endsample %}

此示例将使 `http://localhost:1234/api/endpoint` 被代理到 `http://localhost:8000/endpoint`.

### `.proxyrc.js`

对于更复杂的配置， `.proxyrc.js` 文件允许您附加任何与 [connect](https://github.com/senchalabs/connect)兼容的中间件。首先，确保您的项目安装了 `http-proxy-middleware`。此示例具有与上述 `.proxyrc` 相同的功能 。

{% sample %}
{% samplefile ".proxyrc.js" %}

```js
const { createProxyMiddleware } = require("http-proxy-middleware");

module.exports = function (app) {
  app.use(
    createProxyMiddleware("/api", {
      target: "http://localhost:8000/",
      pathRewrite: {
        "^/api": "",
      },
    })
  );
};
```

{% endsamplefile %}
{% endsample %}

### 文件监听

为了支持最佳的缓存和开发体验，Parcel 使用了一个用 C++ 编写的非常快的观察器，它集成了每个操作系统的低级文件观察功能。使用这个观察者 Parcel 观察项目根目录中的每个文件（包括所有 `node_modules`）。根据这些文件中的事件和元数据，Parcel 确定需要重建哪些文件。

#### 文件监听的一些问题

##### 安全写入（safe write）

一些文本编辑器和 IDE 具有称为“安全写入（safe write）”的功能，通过获取文件副本并在保存时重命名它来防止数据丢失。但是，此功能可以防止自动检测文件更新。

要禁用安全写入，请使用以下提供的选项：

- Sublime Text 3：添加 `atomic_save: "false"` 到您的用户偏好中。
- IntelliJ：在首选项中使用搜索来查找“safe write”并禁用它。
- Vim: 添加 `:set backupcopy=yes` 到您的设置中。
- WebStorm: 在 Preferences > Appearance & Behavior > System Settings 中取消选中 `Use "safe write"`
- vis: add 添加 `:set savemethod inplace` 到您的设置中。

##### Linux: 设备上没有剩余空间

根据项目的大小和操作系统的观察者限制，在 Linux 上运行 Parcel 时可能会弹出此错误。要解决此问题，请将 `sysctl` 的配置更改为 `fs.inotify` 让 `max_user_watches` 具有更高的值.

您可以通过添加或更改 `/etc/sysctl.conf`: 文件的以下行来执行此操作

```
fs.inotify.max_queued_events = 16384
fs.inotify.max_user_instances = 128
fs.inotify.max_user_watches = 16384
```

如果此错误仍然存 ​​ 在，您可以尝试进一步增加值。

##### 使用 Dropbox、Google Drive 或其他云存储解决方案

最好不要将 Parcel 项目放在使用 Dropbox 或 Google Drive 等同步到云的文件夹中。这些解决方案会创建大量文件系统事件，这些事件可能会干扰我们的观察程序并导致不必要的重建。

## 自动安装

当您使用默认不包含的语言或插件时，Parcel 会自动为您将必要的依赖项安装到您的项目中。例如，如果您包含一个.sass 文件，Parcel 将安装该 `@parcel/transformer-sass` 插件。发生这种情况时，您将在终端中看到一条消息，并且新的依赖项将添加到您的 `package.json` 的 `devDependencies` 中。

Parcel 会根据锁定文件自动检测您在项目中使用的包管理器。例如，如果 `yarn.lock` 存在，则将用 Yarn 安装包。如果未找到锁定文件，则根据系统上安装的内容选择包管理器。当前支持以下包管理器，按优先级顺序列出：

- [Yarn](https://yarnpkg.com)
- [Pnpm](https://pnpm.io)
- [Npm](https://www.npmjs.com)

默认情况下，自动安装仅在开发期间发生。在生产构建期间，如果缺少依赖项，则构建将失败。您还可以在开发期间使用 `--no-autoinstall` CLI 标志禁用自动安装。

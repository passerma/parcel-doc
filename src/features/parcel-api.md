---
layout: layout.njk
title: Parcel API
eleventyNavigation:
  key: features-parcel-api
  title: 📚 Parcel API
  order: 10
---

Parcel API 可用于以编程方式运行构建或监视项目的更改。它与 Parcel CLI 使用的 API 相同。当您需要更大的灵活性或需要将 Parcel 集成到另一个构建系统时使用 API。

## `Parcel` constructor

Parcel API 可以通过`@parcel/core`包使用。您还需要一个默认配置，例如`@parcel/config-default`。

```shell
yarn add @parcel/core @parcel/config-default
```

接下来，将此包导入您的程序并调用`Parcel` constructor。它接受一个[`InitialParcelOptions`](/plugin-system/api/#InitialParcelOptions)对象，该对象包含 Parcel CLI 使用的所有选项以及更多选项。

有两个必需参数：

- `entries` – 描述要构建的入口点的字符串或字符串数 ​​ 组。请参阅[Entries](/features/targets/#entries)。
- `config`或`defaultConfig` – Parcel 配置要使用的文件路径或包说明符。如果`config`设置，则始终使用提供的配置并`.parcelrc`忽略项目。如果`defaultConfig`设置，则项目`.parcelrc`优先，并`defaultConfig`用作后备。如果给出了相对路径或包说明符，则相对于项目根进行解析。

{% sample %}
{% samplefile "build.mjs" %}

```javascript
import { Parcel } from "@parcel/core";

let bundler = new Parcel({
  entries: "a.js",
  defaultConfig: "@parcel/config-default",
});
```

{% endsamplefile %}
{% endsample %}

### 目标 Targets

默认情况下，Parcel 进行开发构建，但这可以通过将`mode`选项设置为`production`，从而启用范围提升、缩小等。请参阅[生产 Production](/features/production/)。

如果项目中未配置[Targets](/features/targets/)，您还可以使用`defaultTargetOptions`设置目标值。例如，要覆盖默认浏览器目标，请使用`engines`选项。

{% sample %}
{% samplefile "build.mjs" %}

```javascript
import { Parcel } from "@parcel/core";

let bundler = new Parcel({
  entries: "a.js",
  defaultConfig: "@parcel/config-default",
  mode: "production",
  defaultTargetOptions: {
    engines: {
      browsers: ["last 1 Chrome version"],
    },
  },
});
```

{% endsamplefile %}
{% endsample %}

当设置为数组时，`targets`选项可用于指定要构建的项目目标（如`package.json`中所述）。例如，仅构建项目的`modern`目标：

{% sample %}
{% samplefile "build.mjs" %}

```javascript
import { Parcel } from "@parcel/core";

let bundler = new Parcel({
  entries: "a.js",
  defaultConfig: "@parcel/config-default",
  targets: ["modern"],
});
```

{% endsamplefile %}
{% endsample %}

或者，`targets`可以设置为一个对象，该对象将覆盖项目中定义的任何目标。有关可用选项的更多详细信息，请参阅[Targets](/features/targets/)。

{% sample %}
{% samplefile "build.mjs" %}

```javascript
import { Parcel } from "@parcel/core";

let bundler = new Parcel({
  entries: "a.js",
  defaultConfig: "@parcel/config-default",
  mode: "production",
  targets: {
    modern: {
      engines: {
        browsers: ["last 1 Chrome version"],
      },
    },
    legacy: {
      engines: {
        browsers: ["IE 11"],
      },
    },
  },
});
```

{% endsamplefile %}
{% endsample %}

### 环境变量

环境变量如`NODE_ENV`可以使用`env`选项进行设置。这允许在不重写`process.env`上的值的情况下设置变量。

{% sample %}
{% samplefile "build.mjs" %}

```javascript
import { Parcel } from "@parcel/core";

let bundler = new Parcel({
  entries: "a.js",
  defaultConfig: "@parcel/config-default",
  mode: "production",
  env: {
    NODE_ENV: "production",
  },
});
```

{% endsamplefile %}
{% endsample %}

### Reporters

默认情况下，当您使用该 API 时，Parcel 不会向 CLI 写入任何输出。这意味着它不会报告状态信息，也不会为诊断提供漂亮的格式。可以使用`additionalReporters`除了在`.parcelrc`。`@parcel/reporter-cli`插件提供 CLI 使用的默认报告器，但您也可以使用任何其他报告器插件。

{% sample %}
{% samplefile "build.mjs" %}

```javascript
import { Parcel } from "@parcel/core";
import { fileURLToPath } from "url";

let bundler = new Parcel({
  entries: "a.js",
  defaultConfig: "@parcel/config-default",
  additionalReporters: [
    {
      packageName: "@parcel/reporter-cli",
      resolveFrom: fileURLToPath(import.meta.url),
    },
  ],
});
```

{% endsamplefile %}
{% endsample %}

## 打包 Building

构建`Parcel`实例后，您可以使用它来构建项目或监视更改。要构建一次，请使用`run`API。这将返回一个 Promise，如果构建成功，它将通过[`BuildSuccessEvent`](/plugin-system/reporter/#BuildSuccessEvent)包含[`BundleGraph`](/plugin-system/bundler/#BundleGraph)和其他一些信息的信息来解决，或者如果构建失败，则以错误拒绝。构建错误有一个或多个[`Diagnostic`](/plugin-system/logging/#Diagnostic)附加到它们的对象，其中包含错误的详细信息。

{% sample %}
{% samplefile "build.mjs" %}

```javascript
import { Parcel } from "@parcel/core";

let bundler = new Parcel({
  entries: "a.js",
  defaultConfig: "@parcel/config-default",
});

try {
  let { bundleGraph, buildTime } = await bundler.run();
  let bundles = bundleGraph.getBundles();
  console.log(`✨ Built ${bundles.length} bundles in ${buildTime}ms!`);
} catch (err) {
  console.log(err.diagnostics);
}
```

{% endsamplefile %}
{% endsample %}

## 监听 Watching

要监视一个项目的更改并获得每次重建的通知，请使用`watch`API。每当生成成功或失败时，传递一个要调用的回调函数。回调接收一个错误参数和一个事件对象。Error 参数仅用于监视期间的致命错误。正常的构建失败由[`BuildFailureEvent`](/plugin-system/reporter/#BuildFailureEvent)表示，其中包括一个[`Diagnostic`](/plugin-system/logging/#Diagnostic)对象列表。

`watch`也会返回一个订阅对象，当你想停止观看时，应该调用`unsubscribe`方法。

{% sample %}
{% samplefile "build.mjs" %}

```javascript
import { Parcel } from "@parcel/core";

let bundler = new Parcel({
  entries: "a.js",
  defaultConfig: "@parcel/config-default",
});

let subscription = await bundler.watch((err, event) => {
  if (err) {
    // fatal error
    throw err;
  }

  if (event.type === "buildSuccess") {
    let bundles = event.bundleGraph.getBundles();
    console.log(`✨ Built ${bundles.length} bundles in ${event.buildTime}ms!`);
  } else if (event.type === "buildFailure") {
    console.log(event.diagnostics);
  }
});

// some time later...
await subscription.unsubscribe();
```

{% endsamplefile %}
{% endsample %}

## Dev server

开发服务器包含在默认的 Parcel 配置中。可以通过将`serveOptions`传递给`Parcel`constructor 构造函数并在观看模式下运行 Parcel 来启用它。可以通过设置`hmrOptions`启用热重新加载。一个`port`是唯一需要的选项，但其他如 HTTPS 选项也可以设置。

{% sample %}
{% samplefile "build.mjs" %}

```javascript
import { Parcel } from "@parcel/core";

let bundler = new Parcel({
  entries: "a.js",
  defaultConfig: "@parcel/config-default",
  serveOptions: {
    port: 3000,
  },
  hmrOptions: {
    port: 3000,
  },
});

await bundler.watch();
```

{% endsamplefile %}
{% endsample %}

## 文件系统 File system

Parcel 在整个核心和大多数插件中使用一个抽象的文件系统。默认情况下，它依赖于 Node.js 的 API，但 Parcel 也有一个 `MemoryFS`的实现。你可以使用`inputFS`选项覆盖文件系统 Parcel 读取源文件，`outputFS`选项覆盖文件系统 Parcel 将输出(包括缓存)写入。

`MemoryFS`位于`@parcel/fs`包中。构造它需要传递一个`WorkerFarm`实例，以便可以从多个线程读取和写入文件系统。您需要使用从@parcel/core`导出的`createWorkerFarm`函数创建一个工人农场，并将其传递给`MemoryFS`和`Parcel`constructors构造函数。完成后，确保调用工作农场的`end`以允许进程退出。

此示例将其输出写入内存中的文件系统，并记录每个构建包的内容。

{% sample %}
{% samplefile "build.mjs" %}

```javascript
import { Parcel, createWorkerFarm } from "@parcel/core";
import { MemoryFS } from "@parcel/fs";

let workerFarm = createWorkerFarm();
let outputFS = new MemoryFS(workerFarm);

let bundler = new Parcel({
  entries: "a.js",
  defaultConfig: "@parcel/config-default",
  workerFarm,
  outputFS,
});

try {
  let { bundleGraph } = await bundler.run();

  for (let bundle of bundleGraph.getBundles()) {
    console.log(bundle.filePath);
    console.log(await outputFS.readFile(bundle.filePath, "utf8"));
  }
} finally {
  await workerFarm.end();
}
```

{% endsamplefile %}
{% endsample %}

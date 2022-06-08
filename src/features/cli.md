---
layout: layout.njk
title: CLI
eleventyNavigation:
  key: features-cli
  title: ⌨️ CLI
  order: 9
---

`parcel`是使用 Parcel 最常用的方式。它支持三种不同的命令`serve`, `watch`, 和 `build`.

## `parcel [serve] <entries>`

`serve`命令启动一个开发服务器，它会在您更改文件时自动重建您的应用程序，并支持[热重载 hot reloading](/features/development/#hot-reloading)。它接受一个或多个文件路径或 glob 作为[entries](#entries)。`serve`是默认命令，因此也可以通过将条目直接传递给`parcel`。

```bash
parcel src/index.html
```

{% warning %}

**注意**: 如果你已经指定了多个 HTML 入口点，但是没有一个具有输出路径`/index.html`，dev 服务器将用 404 响应`http://localhost:1234/`，，因为 Parcel 不知道哪个 HTML 包是索引。

在这种情况下，直接加载文件，例如`http://localhost:1234/a.html`和`http://localhost:1234/b.html`.

{% endwarning %}

有关详细信息，请参阅[Development](/features/development/)。

## `parcel watch <entries>`

`watch`命令类似于`serve`,，但不启动开发服务器（仅 HMR 服务器）。但是，它会在您进行更改时自动重建您的应用程序，并支持[热重载 hot reloading](/features/development/#hot-reloading)。如果您正在构建库、后端或拥有自己的开发 (HTTP) 服务器，请使用此选项`watch`。有关如何指定入口的信息，请参见[below](#entries)。

```bash
parcel watch src/index.html
```

## `parcel build <entries>`

`build`命令执行单个生产构建并退出。这默认启用[scope hoisting](/features/scope-hoisting)和其他生产优化。有关如何指定入口的信息，请参见[below](#entries)。

```bash
parcel build src/index.html
```

有关详细信息，请参阅[生产 Production](/features/production/)。

## 入口 Entries

所有 Parcel 命令都接受一个或多个入口。入口可以是相对或绝对路径，也可以是全局路径。它们也可能是包含`source`字段的`package.json`的目录。如果入口被完全省略，则使用当前工作目录中`package.json`中的`source`字段。有关详细信息，请参阅目标文档中的[入口 Entries](/features/targets/#entries)。

{% warning %}

**注意**: 请务必将 glob 用单引号括起来，以确保它们不会被您的 shell 解析并直接传递给 Parcel。这确保 Parcel 可以自动拾取与 glob 匹配的新创建的文件，而无需重新启动。

{% endwarning %}

```bash
# 单文件
parcel src/index.html

# 多文件
parcel src/a.html src/b.html

# Glob (quotes required)
parcel 'src/*.html'

# Directory with package.json#source
parcel packages/frontend

# Multiple packages with a glob
parcel 'packages/*'

# Current directory with package.json#source
parcel
```

## 参数 Parameters

所有 Parcel 命令都支持这些参数。

| 格式                                         | 描述                                                                                                                                   |
| -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- | --- |
| `--target [name]`                            | 指定要构建的目标。可以多次指定。请参阅[Targets](/features/targets/).                                                                   |
| `--dist-dir <dir>`                           | 目标未指定时要写入的输出目录。<br>Package.json 的`targets`中[`distDir`](/features/targets/#distdir) 选项的默认值。                     |
| `--public-url <url>`                         | 绝对 url 的路径前缀。<br> Package.json 的`targets`中[`publicUrl`](/features/targets/#publicurl)选项的默认值。                          |
| `--no-source-maps`                           | 禁用源映射，<br>覆盖 package.json 的`targets`中[`sourceMap`](/features/targets/#sourcemap)的选项。                                     |
| `--config <path>`                            | 指定要使用的 Parcel 配置。<br>可以是文件路径或包名。默认为`@parcel/config-default`。请参阅[Parcel configuration](/features/plugins/)。 |
| `--reporter <package name>`                  | 除了`.parcelrc`中指定的外，还可以运行指定的 reporter 插件。可以多次指定。                                                              |
| `--log-level (none/error/warn/info/verbose)` | 设置日志级别。                                                                                                                         |
| `--cache-dir <path>`                         | 设置缓存目录。默认为`.parcel-cache`。请参阅[Caching](/features/development/#caching)。                                                 |
| `--no-cache`                                 | 禁用从文件系统缓存中读取。请参阅[Caching](/features/development/#caching)。                                                            |     |
| `--profile`                                  | 配置构建（可以生成火焰图）。                                                                                                           |
| `-V, --version`                              | 输出版本号。                                                                                                                           |

### 特定于`serve`和`watch`的参数

| 格式                | 描述                                                                                                                   |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `-p, --port <port>` | 开发服务器和 HMR 的端口（默认端口是`process.env.PORT`或 1234)。请参阅[Dev server](/features/development/#dev-server)。 |
| `--host <host>`     | 设置要监听的主机，默认监听所有接口                                                                                     |
| `--https`           | 通过[HTTPS](/features/development/#https)运行开发服务器和 HMR 服务器。                                                 |
| `--cert <path>`     | 要使用的证书的路径。请参阅[HTTPS](/features/development/#https)。                                                      |
| `--key <path>`      | 要使用的私钥的路径。请参阅[HTTPS](/features/development/#https)。                                                      |
| `--no-hmr`          | 禁用[热重载 hot reloading](/features/development/#hot-reloading)。                                                     |
| `--hmr-port <port>` | HMR 服务器的端口（默认为开发服务器的端口）。请参阅[Hot reloading](/features/development/#hot-reloading).               |
| `--hmr-host <host>` | HMR 服务器的主机（默认为开发服务器的主机）。请参阅[Hot reloading](/features/development/#hot-reloading).               |
| `--no-autoinstall`  | 禁用[auto install](/features/development/#auto-install)。                                                              |
| `--watch-for-stdin` | 标准输入关闭后停止 Parcel。                                                                                            |

### 特定于`serve`的参数

| 格式               | 描述                                                                                                  |
| ------------------ | ----------------------------------------------------------------------------------------------------- |
| `--open [browser]` | 自动在浏览器中打开条目。默认为默认浏览器。请参阅[开发 Dev server](/features/development/#dev-server). |
| `--lazy`           | 仅构建开发服务器请求的捆绑包。请参阅[懒加载 Lazy mode](/features/development/#lazy-mode).             |

### 特定于`build`的参数

| 格式                        | 描述                                                                                                                                                             |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| `--no-optimize`             | 禁用优化，例如缩小。<br> 覆盖 package.json 的选项`targets`属性[`optimize`](/features/targets/#optimize)。请参阅[生产 Production](/features/production/)。        |
| `--no-scope-hoist`          | 禁用 scope hoisting. <br>覆盖 package.json 的选项`targets`属性[`scopeHoist`](/features/targets/#scopehoist)。请参阅[Scope hoisting](/features/scope-hoisting/)。 |
| `--no-content-hash`         | 禁用输出文件名的内容散列。<br> 打包名称可能仍包含哈希，但它们不会在每次构建时更改。请参阅[Content hashing](/features/production/#content-hashing)。              |     |
| `--detailed-report [depth]` | 在 CLI 报告中显示每个捆绑包中最大的 10 个（使用`depth`可配置的数量）资源。见[Detailed report](/features/production/#detailed-report)。                           |

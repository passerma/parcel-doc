---
layout: layout.njk
title: 目标(Targets)
eleventyNavigation:
  key: features-targets
  title: 🎯 目标(Targets)
  order: 5
---

Parcel 可以同时以多种不同的方式编译您的源代码。这些被称为**目标**。例如，您可以有一个针对较新浏览器的“现代”目标和一个针对旧浏览器的“旧版”目标。

## 入口 Entries

“入口 Entries”是 Parcel 在构建源代码时开始的文件。它们可以在 CLI 上指定，或者使用 package.json 中的`source`字段。

### `$ parcel <entries>`

可以在 CLI 上为任何 Parcel 命令指定一个或多个入口文件。

```shell
$ parcel src/a.html src/b.html
```

入口可以指定为 glob 以一次匹配多个文件。请务必将 glob 用单引号括起来，以确保 glob 不会被您的 shell 解析并直接传递给 Parcel。这确保 Parcel 可以自动拾取与 glob 匹配的新创建的文件，而无需重新启动。

```shell
$ parcel './src/*.html'
```

入口也可以是目录，在这种情况下，必须在`package.json`中包含`source`字段的文件，详情见下文。

### `package.json#source`

package.json 中的`source`字段可以指定一个或多个入口文件。

```json
{
  "source": "src/index.html"
}
```

```json
{
  "source": ["src/a.html", "src/b.html"]
}
```

### `package.json#targets.*.source`

package.json 中声明的任何目标中的`source`字段可以指定一个或多个特定于该目标的入口文件。例如，您可以同时构建前端和后端，或者您的桌面和移动应用程序。有关配置目标的详细信息，请参见下文。

```json
{
  "targets": {
    "frontend": {
      "source": "app/index.html"
    },
    "backend": {
      "source": "api/index.js"
    }
  }
}
```

## 目标 Targets

Parcel 遵循每个已解析目标中的依赖关系，为一个或多个目标构建源代码。目标指定输出目录或文件路径，以及有关如何编译代码的信息。

默认情况下，Parcel 包含一个输出到`dist`件夹的隐式目标。这可以使用`--dist-dir`CLI 选项覆盖。

```shell
$ parcel build src/index.html --dist-dir output
```

也可以在 package.json 中使用`targets`字段指定输出目录。这将覆盖`--dist-dir`CLI 选项。

```json
{
  "targets": {
    "default": {
      "distDir": "./output"
    }
  }
}
```

### 环境

除了输出位置之外，目标还指定了有关代码将在其中运行的“环境”的信息。它们告诉 Parcel 要构建的环境类型（例如浏览器或 Node.js），以及每个引擎的版本支持。这会影响 Parcel 编译代码的方式，包括要转换的语法。

#### `package.json#browserslist`

对于浏览器目标，ackage.json 中的`browserslist`字段可用于指定您支持的浏览器。您可以通过使用统计或特定浏览器的版本范围进行查询。有关更多信息，请参阅[browserslist 文档](https://github.com/browserslist/browserslist#full-list)。

```json
{
  "browserslist": "> 0.5%, last 2 versions, not dead"
}
```

#### `package.json#engines`

对于 Node.js 和其他目标，package.json 中的`engines`字段可用于指定您支持的版本。使用 semver 范围指定引擎。

```json
{
  "engines": {
    "node": ">= 12"
  }
}
```

### 隐式环境

当一个文件依赖于另一个文件时，环境是从其父文件继承的。但是您对资产的依赖方式可能会改变环境的某些属性。例如，当依赖于 Service Worker 时，环境会自动更改为 Service Worker 上下文，以便适当地编译代码。

```javascript
navigator.serviceWorker.register(new URL("service-worker.js", import.meta.url));
```

### 差分打包

“差分打包”是指为不同的目标发布多个版本的代码，并允许浏览器选择最优化的版本进行下载。当您在 HTML 文件中使用`<script type="module">`元素时，并且环境指定的某些浏览器本身不支持 ES 模块时，Parcel 也会自动生成`<script nomodule>`回退。

```html
<script type="module" src="app.js"></script>
```

编译为：

```html
<script type="module" src="app.c9a6fe.js"></script>
<script nomodule src="app.f7d631.js"></script>
```

这允许支持 ES 模块的现代浏览器下载更小的包，而使用后备仍然支持旧版浏览器。这可以通过避免转换现代 JavaScript 语法（如类、箭头函数、异步/等待等）来显着减少包大小并缩短加载时间。

这会根据您的浏览器目标自动发生，正如您在 package.json 中的`"browserslist"`中的字段中所声明的那样。如果没有`browserslist`或者所有浏览器目标都原生支持 ES 模块，则`nomodule`不会生成回退。

## 多个目标 Multiple targets

您可能有多个目标，以便同时为多个不同的环境构建源代码。例如，您可以为应用程序设置“现代”和“传统”目标，或者为库设置 ES 模块和 CommonJS 目标（[见下文](#library-targets)）。

使用 package.json 中的字段配置`targets`字段。每个目标都有一个名称，指定为`target`字段下的键，以及一个关联的配置对象。例如，`engines`每个目标中的字段可用于自定义编译它的环境。

```json
{
  "targets": {
    "modern": {
      "engines": {
        "browsers": "Chrome 80"
      }
    },
    "legacy": {
      "engines": {
        "browsers": "> 0.5%, last 2 versions, not dead"
      }
    }
  }
}
```

当指定多个目标时，默认情况下将写入输出`dist/${targetName}`（例如`dist/modern`，`dist/legacy`在上面的示例中）。这可以使用`distDir`每个目标中的字段进行自定义。或者，如果目标只有一个入口，则可以使用与目标名称对应的顶级 package.json 字段为输出指定准确的文件名。

```json
{
  "modern": "dist/modern.js",
  "legacy": "dist/legacy.js",
  "targets": {
    "modern": {
      "engines": {
        "browsers": "Chrome 80"
      }
    },
    "legacy": {
      "engines": {
        "browsers": "> 0.5%, last 2 versions, not dead"
      }
    }
  }
}
```

## 库目标 Library targets

Parcel 包含一些用于构建库的内置目标。其中包括`main`, `module`, `browser`, 和 `types`字段。

```json
{
  "name": "my-library",
  "version": "1.0.0",
  "source": "src/index.js",
  "main": "dist/main.js",
  "module": "dist/module.js",
  "types": "dist/types.d.ts"
}
```

默认情况下，库目标不打包`node_modules`依赖项。此外，库默认禁用缩小。可以使用目标字段中的适当选项重写这些内容(见下文)。Scope hoisting 不能被禁用为库的目标。

库目标会根据目标自动输出原生 ES 模块或 CommonJS。

- **`main`** – 默认情况下，输出 CommonJS。如果`.mjs`使用了扩展名，或者`"type": "module"`指定了字段，则改为输出 ES 模块。
- **`module`** – 输出一个 ES 模块。
- **`browser`** – 特定于浏览器的`main`字段覆盖。输出 CommonJS。

`main`和`module`如果还有可用的目标，或者如果指定了但未指定浏览器目标，则默认为 Node 环境,`browser`编译`engines.node`。否则，它们默认为浏览器环境编译。这可以使用`context`目标配置中的选项覆盖（见下文）。

要使 Parcel 忽略这些字段之一，请在`targets`字段中指定`false`。

```json
{
  "main": "unrelated.js",
  "targets": {
    "main": false
  }
}
```

有关[Building a library with Parcel](/getting-started/library/)构建库的介绍，请参阅使用 Parcel 构建库。

## 目标选项

### `context`

```javascript
"node" |
  "browser" |
  "web-worker" |
  "service-worker" |
  "worklet" |
  "electron-main" |
  "electron-renderer";
```

`context`属性定义了要构建的环境类型。这告诉 Parcel 有哪些特定于环境的 API 可用，例如 DOM、Node 文件系统 API 等。

对于内置库目标（例如`main`和`module`)），`context`会自动推断 。有关更多详细信息，请参阅上面的[Library targets](#library-targets)。

### `engines`

覆盖此目标的顶级`package.json#engines`和`browserslist`段中定义的引擎。目标中的`engines.browsers`字段可以像`browserslist`。有关详细信息，请参阅上面的[环境 Environments](#environments)和[多目标 Multiple targets](#multiple-targets)。

### `outputFormat`

```javascript
"global" | "esmodule" | "commonjs";
```

定义要输出的模块类型。

- `global` – 可以在浏览器的`<script>`标签中加载的经典脚本。库目标不支持。
- `esmodule` – 一个使用`import`和`export`语句的 ES 模块。可以`<script type="module">`在浏览器的标签中加载，或者由 Node.js 或其他捆绑程序加载。
- `commonjs` – 一个使用`require`和`module.exports`的 CommonJS 模块。可以由 Node.js 或其他捆绑程序加载。

对于内置库目标（例如`main`和`module`），`outputFormat`会自动推断 。在目标的顶级 package.json 字段中定义的文件扩展名也可能影响输出格式。有关更多详细信息，请参阅上面的[库目标 Library targets](#library-targets)。

### `scopeHoist`

启用或禁用 scope hoisting。默认情况下，为生产构建启用 scope hoisting。`--no-scope-hoist`CLI 标志可用于在运行`parcel build`时禁用 scope hoisting。Scope hoisting 也可以通过在目标配置中设置`scopeHoist`选项来禁用。

### `isLibrary`

当设置为`true`,时，目标被视为将发布到 npm 并由另一个工具使用的库，而不是直接在浏览器或其他目标环境中使用。当为`true`，该`outputFormat`选项必须是`esmodule`或`commonjs`并且`scopeHoist`不能设置为`false`。

对于内置库目标（例如`main`和`module`），它会自动设置为`true`。有关更多详细信息，请参阅上面的[库目标 Library targets](#library-targets)。

### `optimize`

E 启用或禁用优化（例如缩小）。确切的行为由插件决定。默认情况下，在生产构建期间启用优化 (`parcel build`)，库目标除外。这可以使用`--no-optimize`CLI 标志或目标配置中的选项`optimize`覆盖。

### `includeNodeModules`

确定是打包`node_modules`还是将它们视为外部。用于浏览器目标默认是`true`，库目标默认是`false`。可能的值为：

- **`false`** – 不包括`node_modules`。
- **an array** – 要包含的包名称列表。在以下示例中，*仅*打包`react`。所有其他文件`node_modules`都被排除在外。

  ```json
  {
    "targets": {
      "main": {
        "includeNodeModules": ["react"]
      }
    }
  }
  ```

- **an object** – 包名到布尔值的映射。如果未列出软件包，则将其包括在内。在以下示例中，`node_modules` _除了_ react 之外的所有内容都捆绑在一起。

  ```json
  {
    "targets": {
      "main": {
        "includeNodeModules": {
          "react": false
        }
      }
    }
  }
  ```

### `sourceMap`

启用或禁用 sourceMap，并设置源映射选项。默认情况下，源映射已启用。这可以使用`--no-source-maps`CLI 标志覆盖，或者通过目标配置中的选项将`sourceMap`设置为`false`。

`sourceMap`选项还接受具有以下选项的对象。

- `inline` – 是否将源映射作为数据 URL 内联到包中，而不是作为单独的输出文件链接到它。
- `inlineSources` – 是否将原始源代码内联到源映射中，而不是从`sourceRoot`，在为生产构建浏览器目标时，默认情况下设置为`true`。
- `sourceRoot` – 加载原始源代码的 URL。这是在使用内置 Parcel 开发服务器时在开发中自动设置的。否则，它默认为从项目根目录到包的相对路径。

### `source`

覆盖目标根目录的 package.json 中的`source`字段。这允许每个目标具有不同的条目。有关更多详细信息，请参阅[package.json#targets.\*.source](#package.json%23targets.*.source)。

### `distDir`

设置将写入此目标中的已编译包的位置。默认情况下，`dist`如果只给出一个目标，或者`dist/${targetName}`多个目标。有关详细信息，请参阅[目标 Targets](#targets)。

### `publicUrl`

设置在运行时将加载此捆绑包的基本 URL。打包的相对路径`distDir`将自动附加。如果捆绑包是从与您的网站相同的域加载的，则`publicUrl`可以是完全限定的 URL（例如，也可以`https://some-cdn.com/`绝对路径（例如`/public`)。

默认情况下`publicUrl`是`/`。如果您的 HTML 文件和其他资产部署到同一位置，这是一个很好的默认设置。如果您将资产部署到不同的位置，您可能需要设置`publicUrl`。 也可以使用`--public-url`CLI 选项设置公共 URL。

在大多数情况下，使用从父捆绑包到子捆绑包的相对路径加载捆绑包。这允许将部署移动到新位置而无需重新构建（例如，将暂存构建提升到生产环境）。当`publicUrl`相对路径不可能时使用（例如在 HTML 中）。

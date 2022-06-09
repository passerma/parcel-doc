---
layout: layout.njk
title: Plugins
eleventyNavigation:
  key: features-plugins
  title: 🔌 Plugins
  order: 11
---

对于许多零配置的项目来说，Parcel 就是开箱即用。但是如果你想要更多的控制，或者需要扩展或覆盖 Parcel 的默认设置，在你的项目中的文件可以通过创建一个`.parcelrc`文件。

Parcel 的设计非常模块化。Parcel core 本身几乎不是特定于构建 JavaScript 或 web 页面的——所有的行为都是通过插件指定的。对于构建的每个阶段都有特定的插件类型，所以你可以定制几乎所有的东西。

## `.parcelrc`

Parcel 配置在`.parcelrc`文件中指定。它是用[JSON5](https://json5.org)编写的，它类似于 JSON，但支持注释、不带引号的键、尾随逗号和其他功能。

### 扩展配置

Parcel 的默认配置在`@parcel/config-default`。大多数时候，您会希望在自己的 Parcel 配置中扩展它。为此，请在`.parcelrc`中使用`extends`。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default"
}
```

{% endsamplefile %}
{% endsample %}

您还可以通过传递数组来扩展多个配置。配置按指定的顺序合并。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": ["@parcel/config-default", "@company/parcel-config"]
}
```

{% endsamplefile %}
{% endsample %}

您还可以使用相对路径引用项目中的另一个配置。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "../.parcelrc"
}
```

{% endsamplefile %}
{% endsample %}

扩展配置也可以扩展其他配置，形成配置链。

### Glob maps

许多字段在`.parcelrc`像`transformers`或`packagers`一样，使用对象作为全局映射到插件名称。这使您可以通过文件扩展名、文件路径甚至特定的文件名配置 Parcel 的行为。Globs 相对于包含`.parcelrc`的目录进行匹配。

Glob 映射中字段的顺序定义了当针对某个文件名进行测试时它们的优先级。这使您可以为项目中的某些文件配置不同的行为，例如特定目录中的文件。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "transformers": {
    "icons/*.svg": ["highest-priority"],
    "*.svg": ["lowest-priority"]
  }
}
```

{% endsamplefile %}
{% endsample %}

在这里，如果我们试图找到文件的转换`icons/home.svg`me.svg，我们将沿着 glob 向下工作，直到找到匹配项，`icons/*.svg`。我们永远达不到`*.svg`。

一旦检查了当前配置中的所有 glob，Parcel 就会回退到任何扩展配置中定义的 glob。

### Pipelines

许多`.parcelrc`类似于`transformers`, `optimizers`和`reporters`接受一系列的插件。这些被称为**pipelines**。

如果您希望定义一个高优先级的管道来扩展低优先级的管道，而不是重写它，那么您可以使用特殊的`"..."`语法来完成这项工作。将其添加到管道中，以嵌入下一个优先级管道。您可以在管道的开始、结束甚至中间插入它，这样就可以完全控制管道的扩展方式。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "transformers": {
    "icons/*.svg": ["@company/parcel-transformer-svg-icons", "..."],
    "*.svg": ["@parcel/transformer-svg"]
  }
}
```

{% endsamplefile %}
{% endsample %}

在上面的示例中，在处理`icons/home.svg`时，我们首先运行`@company/parcel-transformer-svg-icons`，然后运行`@parcel/transformer-svg`。

这也适用于已经扩展的信任。如果使用了`"..."`，并且在当前配置中没有定义低优先级的管道，Parcel 将返回到扩展配置中定义的管道 pipelines。

由于默认配置中包含了`@parcel/transformer-svg`，上面的例子可以这样重写:

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "transformers": {
    "icons/*.svg": ["@company/parcel-transformer-svg-icons", "..."]
  }
}
```

{% endsamplefile %}
{% endsample %}

### Named pipelines

除了基于 glob 的管道外，Parcel 还支持 **named pipelines**，这使您能够以多种方式导入相同的文件。命名管道在`.parcelrc`中定义。就像普通的管道一样，但是在开始的时候包含了一个 URL 方案。

例如，默认情况下，导入图片通常会将 URL 返回到外部文件，但是您可以使用默认 Parcel 配置中定义的`data-url:`named pipeline 将其内联为数据 URL。有关详细信息，请参阅[Bundle inlining](/features/bundle-inlining/)。

{% sample %}
{% samplefile "src/example.css" %}

```css
.logo {
  background: url(data-url:./logo.png);
}
```

{% endsamplefile %}
{% endsample %}

您还可以定义自己的命名管道。例如，您可以定义一个`arraybuffer:`named pipeline，它允许您将文件导入为[ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)。`*`glob 可以匹配本例中的任何文件，但也可以使用更特定的 glob。使用`"..."`语法允许 Parcel 按照通常的方式处理文件，然后运行`parcel-transformer-arraybuffer`插件将其转换为 ArrayBuffer。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "transformers": {
    "arraybuffer:*": ["...", "parcel-transformer-arraybuffer"]
  }
}
```

{% endsamplefile %}
{% samplefile "src/example.js" %}

```javascript
import buffer from "arraybuffer:./file.png";
```

{% endsamplefile %}
{% endsample %}

转换器和优化器管道支持命名管道。对于变压器，管道在引用资产的依赖项中指定。对于优化器，它是从绑定包的入口资产继承的。

## Plugins

Parcel 支持许多不同类型的插件，它们作为构建的一部分执行特定的任务。插件被引用在你的`.parcelrc`。使用他们的 NPM 包名称。

### Transformers

<a href="/plugin-browser/?type=%22transformer%22&page=0&filter=%22%22&includeOfficial=false" class="npm text-sm text-gray-800 bg-red-50 hover:bg-red-100 transition group rounded px-2 py-1"><img src="/assets/npm.svg" class="h-3 inline pr-2 cursor-pointer">查找 Transformer plugins</a>

[Transformer](/plugin-system/transformer/)转换单个资产来编译它，发现依赖关系，或将其转换为不同的格式。它们使用`.parcelrc`中的[glob map](#glob-maps)进行配置。多个变压器可以在同一资产上串行运行[pipelines](#pipelines)和[named pipelines](#named-pipelines)从而允许在同一项目中以多种不同方式编译同一文件。`"..."`语法可以用来扩展文件的默认 transformers。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "transformers": {
    "*.svg": ["...", "@parcel/transformer-svg-react"]
  }
}
```

{% endsamplefile %}
{% endsample %}

在编译资源时，其文件类型可能会更改。例如，在编译 TypeScript 时，资产的类型从`ts`或`tsx`改为`js`。当这种情况发生时，Parcel 会重新评估资源应该如何进一步处理，并通过匹配的`.js`的文件。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "transformers": {
    "*.{ts,tsx}": ["@parcel/transformer-typescript-tsc"]
  }
}
```

{% endsamplefile %}
{% endsample %}

{% warning %}

一旦 transformer 更改了资源类型，使其不再与当前管道匹配，该资源将被放入不同的管道中。如果没有与新资源类型匹配的管道，那么转换就完成了。当前管道中稍后定义的 Transformers 将不运行。

{% endwarning %}

### Resolvers

<a href="/plugin-browser/?type=%22resolver%22&page=0&filter=%22%22&includeOfficial=false" class="npm text-sm text-gray-800 bg-red-50 hover:bg-red-100 transition group rounded px-2 py-1"><img src="/assets/npm.svg" class="h-3 inline pr-2 cursor-pointer">查询 Resolver plugins</a>

[Resolver](/plugin-system/resolver/)插件负责将依赖说明符转换为完整的文件路径，并由转换器处理。请参阅[Dependency resolution](/features/dependency-resolution/)以了解其工作原理的详细信息。解析器配置使用插件名数组在`.parcelrc`。解析通过插件列表进行，直到其中一个返回结果。

可以使用`"..."`语法用于扩展默认解析器。这允许您覆盖某些依赖项的解决方案，但对其他依赖项回退到默认值。通常，您需要在运行默认解析器之前添加自定义解析器。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "resolvers": ["@company/parcel-resolver", "..."]
}
```

{% endsamplefile %}
{% endsample %}

If `"..."` is omitted, your resolver must be able to handle all dependencies or resolution will fail.

### Bundler (实验性)

<a href="/plugin-browser/?type=%22bundler%22&page=0&filter=%22%22&includeOfficial=false" class="npm text-sm text-gray-800 bg-red-50 hover:bg-red-100 transition group rounded px-2 py-1"><img src="/assets/npm.svg" class="h-3 inline pr-2 cursor-pointer">Find Bundler plugins</a>

[Bundler](/plugin-system/bundler/)插件负责将资源组合成捆绑包。捆绑器可以通过在`.parcelrc`。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "bundler": "@company/parcel-bundler"
}
```

{% endsamplefile %}
{% endsample %}

{% warning %}

**注意**: Bundler 插件是实验性的，可能会发生变化，即使是在次要更新之间也是如此。

{% endwarning %}

### Runtimes (experimental)

<a href="/plugin-browser/?type=%22runtime%22&page=0&filter=%22%22&includeOfficial=false" class="npm text-sm text-gray-800 bg-red-50 hover:bg-red-100 transition group rounded px-2 py-1"><img src="/assets/npm.svg" class="h-3 inline pr-2 cursor-pointer">查询 Runtime plugins</a>

[Runtime](/plugin-system/runtime/)插件允许您将资源注入到包中。它们可以使用在`.parcelrc`。此列表中的所有运行时插件都在每个捆绑包上运行。该`"..."`语法可用于扩展默认运行时。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "runtimes": ["@company/parcel-runtime", "..."]
}
```

{% endsamplefile %}
{% endsample %}

如果`"..."`省略，则不会运行默认运行时。这可能会破坏事情，因为许多 Parcel 功能依赖于默认运行时。

{% warning %}

**注意**: Runtime 插件是实验性的，可能会发生变化，即使在小更新之间也是如此。

{% endwarning %}

### 命名器 Namers

<a href="/plugin-browser/?type=%22namer%22&page=0&filter=%22%22&includeOfficial=false" class="npm text-sm text-gray-800 bg-red-50 hover:bg-red-100 transition group rounded px-2 py-1"><img src="/assets/npm.svg" class="h-3 inline pr-2 cursor-pointer">查询 Namer plugins</a>

[Namer](/plugin-system/namer/)插件确定捆绑的输出文件名。它们是使用在`.parcelrc`。命名通过命名器列表进行，直到其中一个返回结果。

`"..."`语法可用于扩展默认命名器。这允许您覆盖某些捆绑包的命名，但其他捆绑包回退到默认值。通常，您需要在运行默认名称之前添加自定义名称。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "namers": ["@company/parcel-namer", "..."]
}
```

{% endsamplefile %}
{% endsample %}

如果`"..."`省略，您的命名器必须能够处理所有包的命名，否则构建将失败。

### Packagers

<a href="/plugin-browser/?type=%22packager%22&page=0&filter=%22%22&includeOfficial=false" class="npm text-sm text-gray-800 bg-red-50 hover:bg-red-100 transition group rounded px-2 py-1"><img src="/assets/npm.svg" class="h-3 inline pr-2 cursor-pointer">查询 Packager plugins</a>

[Packager](/plugin-system/packager/)插件负责将捆绑包中的所有资产组合到一个输出文件中。它们使用`.parcelrc`中的[glob map](#glob-maps)进行配置。根据包的输出文件名匹配 Globs。可以配置一个单独的打包程序插件来运行每个包。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "packagers": {
    "*.{jpg,png}": "@company/parcel-packager-image-sprite"
  }
}
```

{% endsamplefile %}
{% endsample %}

### Optimizers

<a href="/plugin-browser/?type=%22optimizer%22&page=0&filter=%22%22&includeOfficial=false" class="npm text-sm text-gray-800 bg-red-50 hover:bg-red-100 transition group rounded px-2 py-1"><img src="/assets/npm.svg" class="h-3 inline pr-2 cursor-pointer">查询 Optimizer plugins</a>

[Optimizer](/plugin-system/optimizer/)插件类似于 transformers，但它们接受的是一个捆绑而不是单一资源。它们使用`.parcelrc`中的[glob map](#glob-maps)进行配置。多个优化器可以使用[pipelines](#pipelines)和[named pipelines](#named-pipelines)串行地在同一个 bundle 上运行，从而允许在同一个项目中以多种不同的方式编译同一个 bundle。可以使用`"..."`语法扩展包的默认优化器。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "optimizers": {
    "*.js": ["@parcel/optimizer-esbuild"]
  }
}
```

{% endsamplefile %}
{% endsample %}

### 压缩器 Compressors

<a href="/plugin-browser/?type=%22compressor%22&page=0&filter=%22%22&includeOfficial=false" class="npm text-sm text-gray-800 bg-red-50 hover:bg-red-100 transition group rounded px-2 py-1"><img src="/assets/npm.svg" class="h-3 inline pr-2 cursor-pointer">查询 Compressor plugins</a>

[Compressor](/plugin-system/compressor/)插件用于将最后一个包写入磁盘，可以以某种方式对其进行压缩或编码(例如 Gzip)。它们使用`.parcelrc`中的[glob map](#glob-maps)进行配置。 多个 Compressors 可以使用[pipelines](#pipelines)在同一捆绑上运行。每个压缩器插件都会生成一个并行编写的附加文件，例如`bundle.js`, `bundle.js.gz`和`bundle.js.br`。可以使用"..."`语法扩展包的默认压缩器。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "compressors": {
    "*.{js,html,css}": [
      "...",
      "@parcel/compressor-gzip",
      "@parcel/compressor-brotli"
    ]
  }
}
```

{% endsamplefile %}
{% endsample %}

### Reporters

<a href="/plugin-browser/?type=%22reporter%22&page=0&filter=%22%22&includeOfficial=false" class="npm text-sm text-gray-800 bg-red-50 hover:bg-red-100 transition group rounded px-2 py-1"><img src="/assets/npm.svg" class="h-3 inline pr-2 cursor-pointer">查询 Reporter plugins</a>

[Reporter](/plugin-system/reporter/)插件在整个构建过程中接收 Parcel 发生的事件。例如，记者可以将状态信息写入 stdout，运行 dev 服务器，或者在构建结束时生成捆绑分析报告。记录程序使用`.parcelrc`中的包名数组进行配置。此列表中的所有记录器都针对每个生成事件运行。可以使用`"..."`语法扩展默认记录器。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "reporters": ["...", "@parcel/reporter-bundle-analyzer"]
}
```

{% endsamplefile %}
{% endsample %}

您不经常使用的 Reporters 也可以使用选项`--reporter`在[CLI](/features/cli/)上指定，或者使用`additionalReporters`选项通过[API](/features/parcel-api/) 指定。Reporters 在`.parcelrc`中指定一直在运行。

### 本地插件

Parcel 插件是 NPM 包。这意味着他们有一个`package.json`声明他们兼容的 Parcel 版本，以及他们可能拥有的任何依赖项。他们还必须遵循命名系统以确保清晰。

通常，Parcel 插件会发布到 NPM 注册表或内部公司注册表（例如 Artifactory）。这鼓励插件与社区或公司内的项目共享，以避免重复。

但是，在开发插件时，直接在项目中运行它而不先发布它会很有用。有几种方法可以做到这一点。

#### Yarn and NPM workspaces

一种方法是通过[Yarn Workspaces](https://classic.yarnpkg.com/en/docs/workspaces/)或[NPM Workspaces](https://docs.npmjs.com/cli/v7/using-npm/workspaces)。使用 monorepo 设置。这使您可以像依赖已发布的包一样依赖仓库中的其他包。为此，请设置如下项目结构：

```
project
├── .parcelrc
├── package.json
└── packages
    ├── app
    │   └── package.json
    └── parcel-transformer-foo
        ├── package.json
        └── src
            └── FooTransformer.js
```

在您的根`package.json`中，使用`workspaces`字段来引用您的包。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "name": "my-project",
  "private": true,
  "workspaces": ["packages/*"]
}
```

{% endsamplefile %}
{% endsample %}

然后，您可以在`.parcelrc`中的`parcel-transformer-foo`像引用已发布的包一样引用。每当您更新插件的代码时，Parcel 都会重新构建您的项目。

您还可以选择将您的应用程序保留在根目录中（例如，在`src`文件夹中）而不是在`packages/app`。

#### `link:` 协议

Yarn 支持使用`link:`协议定义依赖项以将本地目录引用为包。例如，您可以设置这样的项目结构：

```
project
├── .parcelrc
├── package.json
├── src
│   └── index.html
└── parcel-transformer-foo
    ├── package.json
    └── src
        └── FooTransformer.js
```

`parcel-transformer-foo`在你的根 package.json 中，你可以使用`link:`协议定义对包的依赖。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "name": "my-project",
  "dependencies": {
    "parcel": "^2.0.0",
    "parcel-transformer-foo": "link:./parcel-transformer-foo"
  }
}
```

{% endsamplefile %}
{% endsample %}

然后，在你的`.parcelrc`中的`parcel-transformer-foo`可以像引用已发布的包一样引用。每当您更新插件的代码时，Parcel 都会重新构建您的项目。

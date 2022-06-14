---
layout: layout.njk
title: 插件系统概述
eleventyNavigation:
  key: plugin-system-overview
  title: 概述
  order: 1
summary: 插件系统的高级概述
---

<figure>
  <a href="/assets/diagram-plugin-system.opt.png" target="_blank">
    <img class="img-plugin-diagram" alt="A diagram of the Parcel plugin system" src="/assets/diagram-plugin-system.opt.png"/>
  </a>
</figure>

## Parcel 架构

即使您没有做任何复杂的事情，如果您要使用 Parcel，花一些时间并了解它的工作原理是很有意义的。

### Phases 的阶段

概括地说，Parcel 经历了几个阶段：

- Resolving - 解决
- Transforming - 转型
- Bundling - 捆绑
- Naming - 命名
- Packaging - 打包
- Optimizing - 优化
- Compressing - 压缩

**resolving** 和 **transforming** 阶段并行工作以构建所有资产的图表。

资产图在 **bundling** 阶段被转换为捆绑包。每个包的输出文件名在**Naming**阶段确定。

然后，**Packaging**、**Optimizing**和**Compressing**阶段协同工作以并行生成每个包的最终内容。

**packaging**阶段将每个捆绑包中的资产合并到输出文件中。

**optimizing**阶段转换每个包的内容。完成后，Parcel 确定每个包的内容哈希，这些哈希将应用于最终输出文件名。

最后，**Compressing**阶段为每个输出文件生成一个或多个编码，因为它们被写入文件系统。

### 资产图 Asset Graph

在解析和转换阶段，Parcel 会发现您的应用程序或程序中的所有资产。每个资产都可以对 Parcel 将引入的其他资产有自己的依赖关系。

表示所有这些资产及其相互依赖关系的数据结构称为“资产图”。

### 捆绑图

一旦 Parcel 构建了整个 Asset Graph，它就会开始将其变成“捆绑包”。这些捆绑包是放在一个文件中的资产分组。捆绑包将（通常）仅包含相同语言的资产。

某些资产被视为您应用程序的“入口”点，并将作为单独的捆绑包保留。例如，如果您的 `index.html` 文件链接到 `about.html` 文件，它们不会被合并在一起。

### 插件类型的完整列表

- [Transformer](/plugin-system/transformer): Converts an asset (into another asset) <br>
  _Example: convert Typescript to JavaScript (per file)_
- [Resolver](/plugin-system/resolver): Turns dependency requests into absolute paths (or exclude them) <br>
  _Example: add your own syntax for imports, e.g. `import "^/foo"`_
- [Bundler](/plugin-system/bundler): Turns an asset graph into a bundle graph <br>
  _Example: create a bundler that does vendoring (splitting app and node_modules code)_
- [Namer](/plugin-system/namer): Generates a filename (or filepath) for a bundle <br>
  _Example: place output bundles in a hierarchical file structure, omit hashes in bundle names_
- [Runtime](/plugin-system/runtime): Programmatically inserts (synthetic) assets into bundles" <br>
  _Example: add analytics to every bundle_
- [Packager](/plugin-system/packager): Turns a group of assets (bundle) into a bundle file" <br>
  _Example: concatenate all input CSS files into a CSS bundle_
- [Optimizer](/plugin-system/optimizer): Applies modifications to the finished bundle (similar to a transformer) <br>
  _Example: run a minifier or convert into a data-url for inline usage_
- [Compressor](/plugin-system/compressor): Compresses or encodes bundles in one or more ways <br>
  _Example: compress a bundle with Gzip_
- [Validator](/plugin-system/validator): Analyzes assets and emit warnings and errors <br>
  _Example: do type-checking (TypeScript, Flow)_
- [Config](/features/plugins/): A reuseable '.parcelrc' package <br>
  _Example: provide a tailor-made parcel config for your boilerplate_ <br>
- [Reporter](/plugin-system/reporter): Listens to events of the build <br>
  _Example: generate a bundle report, run a dev server_

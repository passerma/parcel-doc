---
layout: layout.njk
title: 使用 Parcel 构建JavaScript库
description: 一个入门指南，介绍如何用 Parcel构建一个javascript库，包括一个 ES 模块、 CommonJS 和 TypeScript 定义的输出。
eleventyNavigation:
  key: getting-started-library
  title: 📔 JavaScript库
  order: 2
---

## 安装

在开始之前，您需要安装 Node 和 Yarn 或 npm，并为您的项目创建一个目录。然后，使用 Yarn 安装 Parcel：

```shell
yarn add --dev parcel
```

或者在使用 npm 运行时：

```shell
npm install --save-dev parcel
```

## 项目设置

现在已经安装了 Parcel，让 `package.json` 为我们的库设置一个源文件。我们将使用 `source` 字段来声明源文件，并创建一个 `main` [target](/features/targets/) 字段构建输出文件。这将被使用我们库的其他工具（例如其他程序或 Node.js）使用。

{% sample %}
{% samplefile "package.json" %}

```json/3-4
{
  "name": "my-library",
  "version": "1.0.0",
  "source": "src/index.js",
  "main": "dist/main.js",
  "devDependencies": {
    "parcel": "latest"
  }
}
```

{% endsamplefile %}
{% endsample %}

上面示例的 `src/index.js` 用作我们库的源代码，所以接下来让我们创建该文件。在此示例中，我们使用的是 JavaScript 文件，但我们也可以在此处引用 TypeScript 文件或任何其他编译为 JavaScript 的语言。

{% sample %}
{% samplefile "src/index.js" %}

```javascript
export function add(a, b) {
  return a + b;
}
```

{% endsamplefile %}
{% endsample %}

现在，我们的库导出了一个名为 `add` 的函数，它将两个参数相加并返回结果。由于这是使用关键字以 ES 模块语法编写的，因此 Parcel 将按照该字段 `export` 的默认值将我们的代码编译为 CommonJS 模块。输出到 `main` 文件中。

要构建我们的库，在项目目录中运行 `npx parcel build` 。Parcel 将构建您的源代码并在 `main` 字段的 `dist/main.js` 的内容中输出一个 JavaScript 文件。

## 打包脚本

到目前为止，我们一直在 `parcel` 直接运行 CLI，但在您的 `package.json` 文件中创建一些脚本以简化此操作会很有用。我们还将设置一个 `watch` 脚本，该脚本将监视您的源文件的更改并自动重建，因此您在进行更改时无需手动运行 `build` 脚本。

{% sample %}
{% samplefile "package.json" %}

```json/5-8
{
  "name": "my-library",
  "version": "1.0.0",
  "source": "src/index.js",
  "main": "dist/main.js",
  "scripts": {
    "watch": "parcel watch",
    "build": "parcel build"
  },
  "devDependencies": {
    "parcel": "latest"
  }
}
```

{% endsamplefile %}
{% endsample %}

现在您可以运行 `yarn build` 以构建您的项目以进行发布和使用 `yarn watch` 进行开发。

## CommonJS 和 ES 模块

Parcel 接受 CommonJS 和 ES 模块作为输入，并且可以输出这些格式中的一种或多种，​​ 具体取决于 `package.json`。 要添加 ES 模块目标，请将 `module` 字段添加到您的 `package.json`.

{% sample %}
{% samplefile "package.json" %}

```json/5
{
  "name": "my-library",
  "version": "1.0.0",
  "source": "src/index.js",
  "main": "dist/main.js",
  "module": "dist/module.js",
  "devDependencies": {
    "parcel": "latest"
  }
}
```

{% endsamplefile %}
{% endsample %}

现在 Parcel 将 `dist/main.js` 作为 CommonJS 模块和 ES 模块输出。使用您的库的时候将选择它们支持的任何一个。

您还可以使用文件扩展名来指示要输出的模块类型。 `.mjs` 扩展会产生一个 ES 模块，而扩展 `.cjs` 会产生一个 CommonJS 模块。这会覆盖该 `main` 字段的默认行为。`"type": "module"` 字段也可以在 package.json 中设置，以将 `main` 字段也视为 ES 模块。有关更多详细信息，请参阅 [Node.js docs](https://nodejs.org/dist/latest-v16.x/docs/api/packages.html#packages_determining_module_system)。

## TypeScript

Parcel 还支持构建用 TypeScript 编写的库。 `source` 字段可以指向您的 `.ts` 或 `.tsx` 文件，Parcel 会自动将 JavaScript 输出到您的目标中。您还可以使用 `types` 字段在 package.json 中的指向一个 `.d.ts` 文件，Parcel 将在编译的 JavaScript 同目录生成一个类型文件。这让 VSCode 之类的编辑器可以为库的用户提供自动完成功能。

{% sample %}
{% samplefile "package.json" %}

```json/6
{
  "name": "my-library",
  "version": "1.0.0",
  "source": "src/index.ts",
  "main": "dist/main.js",
  "module": "dist/module.js",
  "types": "dist/types.d.ts",
  "devDependencies": {
    "parcel": "latest"
  }
}
```

{% endsamplefile %}
{% endsample %}

现在 Parcel 将输出一个 `dist/types.d.ts` 包含我们库的类型定义以及已编译代码的文件。

## 下一步

现在您已经设置了您的项目，您可以了解 Parcel 的一些更高级的功能。查看有关 [Targets](/features/targets/)的文档，并查看语言部分，以获得使用流行的 Web 框架和工具的更深入的指南。

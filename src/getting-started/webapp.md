---
layout: layout.njk
title: 使用 Parcel 构建 Web 应用程序
description: 一个入门指南，逐步了解如何使用Parcel设置一个项目。
eleventyNavigation:
  key: getting-started-webapp
  title: 🌐 web应用程序
  order: 1
---

## 安装

在开始之前，您需要安装 Node 和 Yarn 或 npm，并为您的项目创建一个目录。然后，使用 Yarn 将 Parcel 安装到您的应用程序中：

```shell
yarn add --dev parcel
```

或者在使用 npm 运行时：

```shell
npm install --save-dev parcel
```

## 项目设置

现在已经安装了 Parcel，让我们为我们的应用程序创建一些源文件。Parcel 接受任何类型的文件作为入口点，但 HTML 文件是一个很好的起点。Parcel 将从那里遵循您的所有依赖项来构建您的应用程序。

{% sample %}
{% samplefile "src/index.html" %}

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>My First Parcel App</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
  </body>
</html>
```

{% endsamplefile %}
{% endsample %}

Parcel 内置了一个开发服务器，它会在您进行更改时自动重建您的应用程序。要启动它，请运行 parcel 指向您的入口文件的 CLI：

```shell
yarn parcel src/index.html
```

或者在使用 npm 运行时：

```shell
npx parcel src/index.html
```

现在在浏览器中打开 [http://localhost:1234/](http://localhost:1234/) 以查看您在上面创建的 HTML 文件。

接下来，您可以开始将依赖项添加到您的 HTML 文件，例如 JavaScript 或 CSS 文件。您可以创建一个 `styles.css` ，在 `index.html` 使用 `<link>` 标签引用，或者一个 `app.js` 文件使用 `<script>` 标签引入。

{% sample %}
{% samplefile "src/styles.css" %}

```css
h1 {
  color: hotpink;
  font-family: cursive;
}
```

{% endsamplefile %}
{% samplefile "src/app.js" %}

```javascript
console.log("Hello world!");
```

{% endsamplefile %}
{% samplefile "src/index.html" %}

```html/5-6
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8"/>
    <title>My First Parcel App</title>
    <link rel="stylesheet" href="styles.css" />
    <script type="module" src="app.js"></script>
  </head>
  <body>
    <h1>Hello, World!</h1>
  </body>
</html>
```

{% endsamplefile %}
{% endsample %}

当您进行更改时，您应该会在浏览器中看到您的应用程序自动更新，甚至无需刷新页面！

在这个例子中，我们展示了如何使用普通的 HTML、CSS 和 JavaScript，但 Parcel 也可以与许多常见的 Web 框架和语言一起使用，比如开箱即用的 [React](/recipes/react/) 和 [TypeScript](/languages/typescript/) 。查看文档的语言部分以了解更多信息。

## 打包脚本

到目前为止，我们一直在 `parcel` 直接运行 CLI，但在您的 `package.json` 文件中创建一些脚本以简化此操作会很有用。我们还将设置一个脚本来使用该命令构建您的应用程序以进行 [生产环境](/features/production/) 发布。使用 `parcel build` 命令。最后，您还可以使用 `source` 在一个地方声明您的 [入口文件](/features/targets/#entries)，这样您就不需要在每个 `parcel` 命令中重复它们。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "name": "my-project",
  "source": "src/index.html",
  "scripts": {
    "start": "parcel",
    "build": "parcel build"
  },
  "devDependencies": {
    "parcel": "latest"
  }
}
```

{% endsamplefile %}
{% endsample %}

现在您可以运行 `yarn build` 以构建您生产环境的项目，使用 `yarn start` 启动开发环境。

## 声明浏览器目标

默认情况下，Parcel 不执行任何代码转换。这意味着如果您使用现代语言功能编写代码，这就是 Parcel 将输出的内容。您可以使用 `browserslist` 字段声明您的应用支持的浏览器。声明此字段时，Parcel 将相应地转译您的代码，以确保与您支持的浏览器兼容。

{% sample %}
{% samplefile "package.json" %}

```json/3
{
  "name": "my-project",
  "source": "src/index.html",
  "browserslist": "> 0.5%, last 2 versions, not dead",
  "scripts": {
    "start": "parcel",
    "build": "parcel build"
  },
  "devDependencies": {
    "parcel": "latest"
  }
}
```

{% endsamplefile %}
{% endsample %}

您可以在 [Targets](/features/targets/) 页面上了解有关目标的更多信息，以及 Parcel 对差异捆绑的自动支持。

## 下一步

现在您已经设置了您的项目，您可以了解 Parcel 的一些更高级的功能。查看有关 [开发环境](/features/development/) 和 [生产环境](/features/production/) 的文档，并查看语言部分，以获得使用流行 Web 框架和工具的更深入的指南。

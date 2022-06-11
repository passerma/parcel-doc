---
layout: layout.njk
title: React
eleventyNavigation:
  key: recipes-react
  title: <img src="/assets/lang-icons/react.svg" alt=""/> React
  order: 3
---

Parcel 非常适合构建单页或多页 React 应用程序。它包括一流的快速刷新开发体验，并支持 JSX、TypeScript、Flow 和许多开箱即用的样式方法。

## 入门

首先，安装`react`和`react-dom`进入您的项目：

```shell
yarn add react react-dom
```

大多数 Parcel 应用程序都以 HTML 文件开头。Parcel 从那里遵循依赖项（例如`<script>`标签）来构建您的应用程序。

{% sample %}
{% samplefile "src/index.html" %}

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>My Parcel App</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="index.js"></script>
  </body>
</html>
```

{% endsamplefile %}
{% samplefile "src/index.js" %}

```jsx
import ReactDOM from "react-dom";
import { App } from "./App";

const app = document.getElementById("app");
ReactDOM.render(<App />, app);
```

{% endsamplefile %}
{% samplefile "src/App.js" %}

```jsx
export function App() {
  return <h1>Hello world!</h1>;
}
```

{% endsamplefile %}
{% endsample %}

正如你所看到的，我们已经从 HTML 文件中的`<script>`元素中引用了`index.js`。这个导入的`react-dom`并用它将我们的`App`组件渲染到我们页面中的`<div id="app">`元素中。

有关开始使用新项目的更多详细信息，请参阅[Building a web app with Parcel](/getting-started/webapp/)。

## JSX

Parcel 在检测到您正在使用 React 时会自动支持 JSX。如果您使用的是 React 17 或更高版本，它还会自动启用[modern JSX transform](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)，这意味着您甚至不需要导入 React 即可使 JSX 工作，正如您 App.js 在上面的示例中所见。

要了解有关 JSX 的更多信息，请参阅 React 文档中的[Introducing JSX](https://reactjs.org/docs/introducing-jsx.html)，[JSX In Depth](https://reactjs.org/docs/jsx-in-depth.html)以及 Parcel 的 JavaScript 文档[JSX](/languages/javascript/#jsx)部分，以获取有关如何配置其处理方式的一些详细信息的详细信息。

## 快速刷新

Parcel 对[React Fast Refresh](https://reactnative.dev/docs/fast-refresh)有一流的支持，在编辑代码时不需要重新加载页面就可以得到快速反馈。在大多数情况下，它可以在编辑代码时保留组件状态，即使您出错。参见[Hot reloading](/features/development#hot-reloading)文档了解详细的工作原理。

### 提示

- **避免类组件** – 快速刷新仅适用于函数组件（和 Hooks）。
- **仅导出 React 组件** – 如果一个文件混合了 React 组件和其他类型的值，则其状态将在其更改时重置。要保留状态，请仅导出 React 组件并尽可能将其他导出移动到不同的文件。
- **避免未命名的默认导出** – 使用默认导出箭头函数声明组件将导致状态在更改时被重置。使用命名函数，或将箭头函数分配给变量。
- **将入口组件保存在它们自己的文件中** – 入口组件应该与调用`ReactDOM.render`的文件分开放置，否则每次更改都会重新挂载它们。

有关更多提示，请参阅官方[React Fast Refresh docs](https://reactnative.dev/docs/fast-refresh)。

## TypeScript

[TypeScript](https://www.typescriptlang.org/)支持开箱即用。您可以从您的 HTML 页面引用`.ts`或`.tsx`文件，Parcel 将按照您的预期编译它。

要为 React 添加 TypeScript 定义，请将以下包安装到您的项目中：

```shell
yarn add @types/react @types/react-dom --dev
```

有关将 TypeScript 与 Parcel 一起使用的更多详细信息，请参阅[TypeScript](/languages/typescript/)。

## Flow

安装时自动支持[Flow](https://flow.org/)。要将其添加到现有项目，首先安装`flow-bin`为依赖项：

```shell
yarn add flow-bin --dev
```

然后，使用`// @flow`您要输入检查的文件顶部的指令。这也向 Parcel 发出信号，哪些文件可以具有在为浏览器编译时应该剥离的 Flow 类型。

有关将 Flow 与 Parcel 一起使用的更多详细信息，请参阅[Flow](/languages/javascript/#flow)。

## Styling

Parcel 支持多种使用 React 编写的样式应用程序的不同方式。

### CSS

您可以将 CSS 文件导入 JavaScript 或 TypeScript 文件以将其与组件一起加载。

{% sample %}
{% samplefile "Button.js" %}

```jsx/0
import './Button.css';

export function Button({ children }) {
  return (
    <button className="button">
      {children}
    </button>
  );
}
```

{% endsamplefile %}
{% samplefile "Button.css" %}

```css
.button {
  background: hotpink;
}
```

{% endsamplefile %}
{% endsample %}

您还可以使用`<link rel="stylesheet">`HTML 文件中的标准元素加载 CSS，但从组件中引用 CSS 有助于明确哪些组件依赖于哪些 CSS。这也有助于代码拆分，因为只会加载您呈现的组件所需的 CSS。

Parcel 还支持 CSS 语言，如[SASS](/languages/sass/), [Less](/languages/less/), 和[Stylus](/languages/stylus/)。有关 Parcel 如何处理，请参阅 CSS。[CSS](/languages/css/)。

### CSS modules

默认情况下，从 JavaScript 导入的 CSS 是全局的。如果两个 CSS 文件定义了相同的类名，它们可能会发生冲突并相互覆盖。为了解决这个问题，Parcel 支持[CSS modules](https://github.com/css-modules/css-modules)。

CSS modules 将每个文件中定义的类视为唯一的。每个类名都被重命名以包含唯一的哈希，并且映射被导出到 JavaScript 以允许引用这些重命名的类名。

要使用 CSS modules，请创建一个带有`.module.css`扩展名的文件，然后从带有[namespace import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#import_an_entire_modules_contents)的 JavaScript 文件中导入它。然后，您可以在 JSX 中渲染元素时使用 CSS 模块的导出。

{% sample %}
{% samplefile "Button.js" %}

```jsx/0,4
import * as classes './Button.module.css';

export function Button({ children }) {
  return (
    <button className={classes.button}>
      {children}
    </button>
  );
}
```

{% endsamplefile %}
{% samplefile "Button.module.css" %}

```css
.button {
  background: hotpink;
}
```

{% endsamplefile %}
{% endsample %}

请参阅[CSS modules](/languages/css/#css-modules)以了解有关 Parcel 如何处理 CSS 模块的更多信息。

### CSS-in-JS

CSS-in-JS 库，如[Styled Components](https://styled-components.com), [Emotion](https://emotion.sh/docs/introduction)和许多其他库都可以很好地与 Parcel 配合使用。有些可能需要构建配置，例如[Babel](/languages/javascript/#babel)插件。要启用它，请在您的项目中创建一个 Babel 配置，Parcel 会自动选择它。

例如，要使用 Emotion，请安装 Babel 插件并在您的项目中创建一个：`.babelrc`：

```shell
yarn add @emotion/babel-plugin --dev
yarn add @emotion/react
```

{% sample %}
{% samplefile ".babelrc" %}

```json
{
  "plugins": ["@emotion/babel-plugin"]
}
```

{% endsamplefile %}
{% endsample %}

您还需要在`tsconfig.json`或`jsxImportSource`中设置`jsxImportSource`选项，这样就可以使用 Emotion 的 JSX 杂注来代替默认值。这使得 css 的道具能够正常工作。

{% sample %}
{% samplefile "jsconfig.json" %}

```json
{
  "compilerOptions": {
    "jsxImportSource": "@emotion/react"
  }
}
```

{% endsamplefile %}
{% endsample %}

现在，您可以使用 CSS-in-JS 渲染元素：

{% sample %}
{% samplefile "Button.js" %}

```jsx
import { css } from "@emotion/react";

export function Button({ children }) {
  return (
    <button
      css={css`
        background: hotpink;
        &:hover {
          background: purple;
        }
      `}
    >
      {children}
    </button>
  );
}
```

{% endsamplefile %}
{% endsample %}

### Tailwind CSS

[Tailwind CSS](https://tailwindcss.com)是一个流行的实用程序优先 CSS 框架。它使用[PostCSS](/languages/css/#postcss)构建一个 CSS 文件，其中仅包含您在代码中使用的类。

要使用它，首先，安装必要的依赖项：

```shell
yarn add tailwindcss postcss autoprefixer --dev
```

接下来，创建 PostCSS 和 Tailwind 所需的配置文件。此示例将使用 Tailwind 的[JIT mode](https://tailwindcss.com/docs/just-in-time-mode)通过仅编译您使用的类来加速构建。确保修改传递给`content`选项的 glob，使其与您将使用 Tailwind 类的所有源文件匹配。

{% sample %}
{% samplefile ".postcssrc" %}

```json
{
  "plugins": {
    "tailwindcss": {}
  }
}
```

{% endsamplefile %}
{% samplefile "tailwind.config.js" %}

```javascript
module.exports = {
  content: ["./src/*.{html,js}"],
  theme: {
    extend: {},
  },
  variants: {},
  plugins: [],
};
```

{% endsamplefile %}
{% endsample %}

最后，你可以从任何匹配`tailwind.config.js`中列出的`content`glob 文件中引用 Tailwind 类。

{% sample %}
{% samplefile "Button.js" %}

```jsx
export function Button({ children }) {
  return (
    <button className="p-2 rounded bg-blue-500 hover:bg-blue-600 transition">
      {children}
    </button>
  );
}
```

{% endsamplefile %}
{% endsample %}

## 图片 Images

您可以使用`URL`构造函数引用 JSX 的外部图像。Parcel 还支持使用[query parameters](/features/dependency-resolution/#query-parameters)调整图像大小并将其转换为不同的格式。它还处理图像优化，并在输出文件名中包含一个[content hash](/features/production/#content-hashing)，用于长期的浏览器缓存。

{% sample %}
{% samplefile "Logo.js" %}

```jsx/0,3
const logo = new URL('logo.svg', import.meta.url);

export function Logo() {
  return <img src={logo} alt="logo" />;
}
```

{% endsamplefile %}
{% endsample %}

请参阅 JavaScript 文档中的[URL dependencies](/languages/javascript/#url-dependencies)了解更多关于这种语法的细节，参阅[Image](/recipes/image/)文档了解更多关于 Parcel 如何处理图像的信息。

### SVG

可以如上所述引用外部 SVG 文件。您还可以将 SVG 作为 React 组件导入，这些组件可以直接在 JSX 中呈现。

首先，安装 `@parcel/transformer-svg-react` 插件并将其添加到您的 `.parcelrc` 中：

```shell
yarn add @parcel/transformer-svg-react --dev
```

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

现在，您可以从组件文件中导入 SVG 并像任何其他组件一样渲染它们。

{% sample %}
{% samplefile "AddButton.js" %}

```jsx
import AddIcon from "./AddIcon.svg";

export function AddButton() {
  return (
    <button aria-label="Add">
      <AddIcon />
    </button>
  );
}
```

{% endsamplefile %}
{% endsample %}

上面的示例展示了如何将每个 SVG 文件转换为 JSX，但在某些情况下您可能希望更具选择性。有关详细信息，请参阅 SVG 文档中的 [Importing as a React component](/languages/svg/#importing-as-a-react-component)。

有关 Parcel 如何转换和优化 SVG 文件的更多信息，请参阅 [SVG](/languages/svg/) 文档。

## 代码拆分 Code splitting

代码拆分通过延迟加载应用程序的各个部分来帮助减少初始页面加载大小。这可以通过使用动态 `import()` 语法以及 [`React.lazy`](https://reactjs.org/docs/code-splitting.html#reactlazy) 来完成。

此示例在用户单击按钮时延迟加载“配置文件”组件。当它看到动态的 `import()` 时，Parcel 将 `Profile` 组件移动到与 `Home` 组件分开的包中，并按需加载它。`React.lazy` 处理将其转换为组件，而 `Suspense` 处理在加载时呈现回退。

{% sample %}
{% samplefile "Home.js" %}

```jsx
import React, { Suspense } from "react";

const Profile = React.lazy(() => import("./Profile"));

export function Home() {
  let [showProfile, setShowProfile] = React.useState(false);

  return (
    <main>
      <h1>Home</h1>
      <button onClick={() => setShowProfile(true)}>Show Profile</button>
      {showProfile && (
        <Suspense fallback={<div>Loading...</div>}>
          <Profile />
        </Suspense>
      )}
    </main>
  );
}
```

{% endsamplefile %}
{% samplefile "Profile.js" %}

```jsx
export default function Profile() {
  return <h2>Profile</h2>;
}
```

{% endsamplefile %}
{% endsample %}

有关 Parcel 中代码拆分的更多详细信息，请参阅 [代码拆分 Code Splitting](/features/code-splitting/) 文档，以及 React 中的 [代码拆分 Code Splitting](https://reactjs.org/docs/code-splitting.html)更多关于 `Suspense` 和 `React.lazy` 的文档。

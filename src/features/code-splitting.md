---
layout: layout.njk
title: 代码拆分
eleventyNavigation:
  key: features-code-splitting
  title: ✂️ 代码拆分
  order: 2
---

Parcel 支持开箱即用的零配置代码拆分。这允许您将应用程序代码拆分为可以按需加载的单独包，从而减小初始包大小并加快加载时间。

代码拆分是通过使用动态 `import()` 语法来控制的，它的工作方式与普通 `import` 语句类似，但返回一个 Promise。这意味着可以异步加载模块。

## 使用动态导入

以下示例显示了如何使用动态导入按需加载应用程序的子页面。

{% sample %}
{% samplefile "pages/index.js" %}

```js
import("./pages/about").then(function (page) {
  // Render page
  page.render();
});
```

{% endsamplefile %}
{% samplefile "pages/about.js" %}

```js
export function render() {
  // Render the page
}
```

{% endsamplefile %}
{% endsample %}

因为 `import()` 返回一个 Promise，你也可以使用 async/await 语法。

{% sample %}
{% samplefile "pages/index.js" %}

```js
async function load() {
  const page = await import("./pages/about");
  // Render page
  page.render();
}
load();
```

{% endsamplefile %}
{% samplefile "pages/about.js" %}

```js
export function render() {
  // Render the page
}
```

{% endsamplefile %}
{% endsample %}

### Tree shaking

当 Parcel 可以确定您使用动态导入模块的哪些导出时，它将对该模块中未使用的导出进行 tree shake。这适用于静态属性访问或解构，使用 `await` 或 Promise `.then` 语法。

{% note %}

**注意:** 对于这种 `await` 情况，不幸的是，未使用的导出只能在 `await` 未转译时被删除（即使用现代浏览器列表配置）。

{% endnote %}

{% sample null, "column" %}

{% samplefile %}

```js
let { x: y } = await import("./b.js");
```

{% endsamplefile %}

{% samplefile %}

```js
let ns = await import("./b.js");
console.log(ns.x);
```

{% endsamplefile %}

{% samplefile %}

```js
import("./b.js").then((ns) => console.log(ns.x));
```

{% endsamplefile %}

{% samplefile %}

```js
import("./b.js").then(({ x: y }) => console.log(y));
```

{% endsamplefile %}

{% endsample %}

## 共享包

当您的应用程序的多个部分依赖于相同的公共模块时，它们会自动去重到一个单独的包中。这允许常用的依赖项与您的应用程序代码并行加载，并由浏览器单独缓存。

例如，如果您的应用程序有多个页面的 `<script>` 标签依赖于相同的公共模块，这些模块将被拆分为“共享包”。这样，如果用户从一个页面导航到另一个页面，他们只需要下载该页面的新代码，而不是页面之间的公共依赖关系。

{% sample %}
{% samplefile "home.html" %}

```html
<!DOCTYPE html>
<div id="app"></div>
<script type="module" src="home.js"></script>
```

{% endsamplefile %}
{% samplefile "home.js" %}

```jsx
import ReactDOM from "react-dom";

ReactDOM.render(<h1>Home</h1>, app);
```

{% endsamplefile %}
{% samplefile "profile.html" %}

```html
<!DOCTYPE html>
<div id="app"></div>
<script type="module" src="profile.js"></script>
```

{% endsamplefile %}
{% samplefile "profile.js" %}

```jsx
import ReactDOM from "react-dom";

ReactDOM.render(<h1>Profile</h1>, app);
```

{% endsamplefile %}
{% endsample %}

Compiled HTML:

{% sample %}
{% samplefile "home.html" %}

```html
<!DOCTYPE html>
<div id="app"></div>
<script type="module" src="react-dom.23f6d9.js"></script>
<script type="module" src="home.fac9ed.js"></script>
```

{% endsamplefile %}
{% samplefile "profile.html" %}

```html
<!DOCTYPE html>
<div id="app"></div>
<script type="module" src="react-dom.23f6d9.js"></script>
<script type="module" src="profile.9fc67e.js"></script>
```

{% endsamplefile %}
{% endsample %}

在上面的示例中，`home.js` 和 `profile.js` 两者都依赖 `react-dom`, 因此它被拆分为一个单独的包，并通过向两个 HTML 页面添加一个额外的标签 `<script>` 来并行加载。

这也适用于已使用动态代码拆分的应用程序的不同部分 `import()`。两个动态导入之间共享的公共依赖项将被拆分并与动态导入的模块并行加载。

### 配置

默认情况下，Parcel 仅在共享模块达到大小阈值时创建共享包。这避免了拆分非常小的模块并创建额外的 HTTP 请求，即使使用 HTTP/2 也会产生开销。如果一个模块低于阈值，它将在页面之间复制。

Parcel 还具有最大并行请求限制，以避免一次过多的请求使浏览器过载，如果达到此限制，则会复制模块。在创建共享包时，较大的模块优先于较小的模块。

默认情况下，这些参数已针对 HTTP/2 进行了调整。但是，您可以调整这些选项以针对您的应用程序提高或降低它们。您可以通过项目根目录中的 package.json 中配置 `@parcel/bundler-default` 来完成此操作。

{% sample %}
{% samplefile "package.json" %}

```json5
{
  "@parcel/bundler-default": {
    minBundles: 1,
    minBundleSize: 3000,
    maxParallelRequests: 20,
  },
}
```

{% endsamplefile %}
{% endsample %}

可用的选项有：

- **minBundles** – 对于要拆分的文件，它必须由多个 `minBundles` 包使用.
- **minBundleSize** – 对于要创建的共享包，它必须比 `minBundleSize` 字节大（在缩小和 tree shaking 之前）。
- **maxParallelRequests** – 为防止过多并发请求使网络过载，这确保了最多 `maxParallelRequests` 可以同时加载同级包。
- **http** – 将上述值设置为针对 HTTP/1 或 HTTP/2 优化的默认值的简写。有关这些默认值，请参见下表。

| `http`      | `minBundles` | `minBundleSize` | `maxParallelRequests` |
| ----------- | ------------ | --------------- | --------------------- |
| 1           | 1            | 30000           | 6                     |
| 2 (default) | 1            | 20000           | 25                    |

您可以在 [web.dev](https://web.dev/granular-chunking-nextjs/) 上阅读有关此主题的更多信息。

## 内部化（internalized）的异步包

如果一个模块从同一个包中同步和异步导入，而不是将其拆分到一个单独的包中，异步依赖将被“内部化（internalized）”。这意味着它将与动态导入保存在同一个包中以避免重复，但包装在 `Promise` 中以保留语义。

出于这个原因，动态导入只是暗示不需要同步依赖，而不是保证会创建一个新的包。

## 重复数据删除

如果动态导入的模块具有在其所有可能的祖先中已经可用的依赖项，则将对其进行重复数据删除。例如，如果页面导入了一个动态导入模块也使用的库，则该库将仅包含在父级中，因为它在运行时已经在页面上。

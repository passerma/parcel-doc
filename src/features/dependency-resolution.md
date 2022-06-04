---
layout: layout.njk
title: 依赖解析
eleventyNavigation:
  key: features-dependency-resolution
  title: 📔 依赖解析
  order: 3
---

当 Parcel 构建您的源代码时，它会发现**依赖项**，这允许将代码分解为单独的文件并在多个地方重用。依赖项描述了在哪里可以找到包含您所依赖的代码的文件，以及有关如何构建它的元数据。

## 依赖说明符

依赖项说明符是一个字符串，它描述了依赖项相对于导入它的文件的位置。例如，在 JavaScript 中，`import`语句或`require`函数可用于创建依赖关系。在 CSS 中，`@import`和`url()`可以使用。通常，这些依赖项不指定完整的绝对路径，而是指定一个较短的说明符，由 Parcel 和其他工具解析为绝对路径。

Parcel 实现了 [Node module resolution algorithm](https://nodejs.org/api/modules.html#modules_all_together)的增强版本。它负责将依赖说明符转换为可以从文件系统加载的绝对路径。除了许多工具支持的标准依赖说明符外，Parcel 还支持一些附加说明符类型和功能。

### 相对说明符

相对说明符以`.`或`..`开头，并解析相对于导入文件的文件。

{% sample %}
{% samplefile "/path/to/project/src/client.js" %}

```javascript
import "./utils.js";
import "../constants.js";
```

{% endsamplefile %}
{% endsample %}

在上面的示例中，第一个导入将解析为 `/path/to/project/src/utils.js`，第二个将解析为 `/path/to/project/constants.js`。

#### 文件扩展名

建议在所有导入说明符中包含完整的文件扩展名。这既提高了依赖性解析性能，又减少了歧义。

也就是说，为了与 Node 中的 CommonJS 和 TypeScript 兼容，Parcel 允许为某些文件类型省略文件扩展名。可以省略的文件扩展名包括 `.ts`, `.tsx`, `.js`, `.jsx`, 和 `.json`。导入所有其他文件类型需要文件扩展名。

以下示例解析为与上述相同的文件。

{% sample %}
{% samplefile "/path/to/project/src/client.js" %}

```javascript
import "./utils";
import "../constants";
```

{% endsamplefile %}
{% endsample %}

请注意，只有在从 JavaScript 或 TypeScript 文件导入时，才能省略这些。在 HTML 和 CSS 等其他文件类型中定义的依赖项始终需要文件扩展名。

#### 目录索引文件

在 JavaScript、Typescript 和其他基于 JS 的语言中，依赖说明符可能会解析为目录而不是文件。如果目录包含一个`package.json`文件，主条目将按照[Package entries](#package-entries)部分中的说明进行解析。如果`package.json`不存在, 它将尝试解析到目录中的索引文件，例如`index.js` 或`index.ts`。索引文件支持上面列出的所有扩展名。

{% sample %}
{% samplefile "/path/to/project/src/app.js" %}

```javascript
import "./client";
```

{% endsamplefile %}
{% endsample %}

例如，如果`/path/to/project/src/client`是目录，则上述说明符可以解析为`/path/to/project/src/client/index.js`。

### 裸说明符

裸说明符以除`.`, `/`, or `~`之外的任何字符开头。 在 JavaScript、TypeScript 和其他基于 JS 的语言中，它们解析为`node_modules`。对于其他类型的文件，例如 HTML 和 CSS，裸说明符的处理方式与[relative specifiers](#relative-specifiers)相同。

{% sample %}
{% samplefile "/path/to/project/src/client/index.js" %}

```javascript
import "react";
```

{% endsamplefile %}
{% endsample %}

在上面的示例中，`react`可能会解析为`/path/to/project/node_modules/react/index.js`。确切的位置将取决于`node_modules`目录的位置以及包内的配置。

`node_modules`从导入文件向上搜索目录。搜索在项目根目录处停止。例如，如果导入文件位于`/path/to/project/src/client/index.js`以下位置，则将被搜索：

- `/path/to/project/src/client/node_modules/react`
- `/path/to/project/src/node_modules/react`
- `/path/to/project/node_modules/react`

一旦找到模块目录，就解析了包条目。有关此过程的更多详细信息，请参阅[Package entries](#package-entries)。

#### 子路径包

裸说明符也可以指定包内的子路径。例如，一个包可能会发布多个入口点，而不仅仅是一个入口点。

```javascript
import "lodash/clone";
```

上面的示例`lodash`在如上所述的`node_modules`目录中解析，然后`clone`在包中解析模块而不是其主入口点。例如，这可能是一个`node_modules/lodash/clone.js`文件。

#### 内置模块

Parcel 包括许多内置 Node.js 模块的垫片，例如`path`和`url`。当依赖说明符引用这些模块名称之一时，内置模块优先于任何安装的`node_modules` 具有相同名称的模块。在为节点环境构建时，内置模块会从包中排除，否则会包含一个 shim。有关内置模块的完整列表，请参阅[Node docs](https://nodejs.org/dist/latest-v16.x/docs/api/)文档。

在为 Electron 环境构建时，该`electron`模块也被视为内置模块并从包中排除。

### 绝对路径

绝对路径以`/`开头, 并解析相对于项目根目录的文件。项目根目录是项目的基本目录，通常包含包管理器锁定文件（例如`yarn.lock`或`package-lock.json`）或源代码控制目录（例如`.git`）。绝对说明符对于避免深度嵌套层次结构中非常长的相对路径可能很有用。

```javascript
import "/src/client.js";
```

上面的示例可以放在项目目录结构中的任何位置的任何文件中，并且始终解析`/path/to/project/src/client.js`。

### 波浪符说明符

波浪符说明符以`~`开头，并相对于导入文件中最近的项目根目录进行解析。项目根目录是一个包含文件`package.json`的目录，通常可以在改目录中找到`node_modules`，波浪符说明符可用于与绝对说明符类似的目的，但当您拥有多个包时更有用。

{% sample %}
{% samplefile "/path/to/project/packages/frontend/src/client/index.js" %}

```javascript
import "~/src/utils.js";
```

{% endsamplefile %}
{% endsample %}

上面的示例将解析为`/path/to/project/packages/frontend/src/utils.js`。

### 查询参数

依赖说明符还可以包括查询参数，这些参数指定解析文件的转换选项。例如，您可以指定宽度和高度以在加载图像时调整图像大小。

```css
.logo {
  background: url(logo.png?width=400&height=400);
}
```

有关图像的更多详细信息，请参阅[Image transformer](/recipes/image/)图片转换器文档。您还可以在自定义[Transformer](/plugin-system/transformer/)插件中使用查询参数。

{% note %}

**注意**: CommonJS 说明符（由`require`函数创建）不支持查询参数。

{% endnote %}

### URL 方案

依赖说明符可以使用 URL 方案来定位[Named pipelines](/features/plugins/#named-pipelines)。这些允许您指定不同的管道来编译文件，而不是默认管道。例如，该`bundle-text:`方案可用于将已编译的捆绑包内联为文本。有关更多详细信息，请参阅[Bundle inlining](/features/bundle-inlining/)。

有一些保留的 URL 方案可能不会用于命名管道，并且具有内置行为。

- `node:` – 指定内置节点模块的另一种方法。请参阅[Builtin modules](#builtin-modules)。
- `npm:` – URL 依赖项（例如在 HTML、CSS 或网络工作者中）从`node_modules`包中导入文件的一种方式。
- `http:`和`https:` – 完全限定的 URL 依赖项。这些在运行时解决，并且不被 Parcel 处理。
- `data:` – 包含内联依赖源代码的[数据 data URL](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs)目前尚未由 Parcel 实施，但保留供将来使用。
- `file:` – [文件 file URL](https://datatracker.ietf.org/doc/html/rfc8089)。保留供将来使用。
- `mailto:`和`tel:` - 常用的 URL 方案。这些都没有被 parcel 所触及。

### 全局说明符

Parcel 支持通过 glob 一次导入多个文件，但是，由于 glob 导入是非标准的，因此它们不包含在默认 Parcel 配置中。要启用它们，请添加`@parcel/resolver-glob`到您的`.parcelrc`。

{% sample %}
{% samplefile ".parcelrc" %}

```json
{
  "extends": "@parcel/config-default",
  "resolvers": ["@parcel/resolver-glob", "..."]
}
```

{% endsamplefile %}
{% endsample %}

启用后，您可以使用诸如`./files/*.js`。这将返回一个对象，其键对应于文件名。

```javascript
import * as files from "./files/*.js";
```

相当于：

```javascript
import * as foo from "./files/foo.js";
import * as bar from "./files/bar.js";

let files = {
  foo,
  bar,
};
```

具体来说，glob 模式的动态部分成为对象的键。如果有多个动态部分，将返回一个嵌套对象。例如，如果`pages/profile/index.js`文件存在，则以下内容将匹配它。

```javascript
import * as pages from "./pages/*/*.js";

console.log(pages.profile.index);
```

这也适用于 URL 方案`bundle-text:`，如 ，以及动态导入。使用动态导入时，生成的对象将包括文件名到函数的映射。可以调用每个函数来加载已解析的模块。这意味着每个文件都是按需加载的，而不是全部预先加载的。

```javascript
let files = import("./files/*.js");

async function doSomething() {
  let foo = await files.foo();
  let bar = await files.bar();
  return foo + bar;
}
```

Globs 也可用于从 npm 包中导入文件：

```js
import * as locales from "@company/pkg/i18n/*.js";

console.log(locales.en.message);
```

Glob 导入也适用于 CSS：

```css
@import "./components/*.css";
```

相当于：

```css
@import "./components/button.css";
@import "./components/dropdown.css";
```

## 包目录

解析包目录时，`package.json`会查阅该文件以确定包条目。Parcel 检查以下字段（按顺序）：

- `source` – 如果模块位于符号链接后面（例如在 monorepo 或 via 中`npm link`），则 Parcel 使用该`source`字段从源代码编译模块。如果包有多个入口点，该`source`字段也可以用作别名映射 - 有关详细信息，请参阅下面的[Aliases](#aliases)。
- `browser` – 特定于浏览器的包版本。如果为[浏览器环境（browser environment）](/features/targets/#environments)构建，浏览器字段会覆盖其他字段。如果包有多个入口点，该`browser`字段也可以用作别名映射 - 有关详细信息，请参阅下面的[Aliases](#aliases)。
- `module` – 包的 ES 模块版本。
- `main` – 包的 CommonJS 版本。

如果这些字段均未设置，或者它们指向的文件不存在，则解析回退到索引文件。有关详细信息，请参阅[目录索引文件（Directory index files）](#directory-index-files)。

## 别名 Aliases

别名可用于覆盖依赖项的正常解析。例如，您可能希望使用不同但与 API 兼容的替换来覆盖模块，或者将依赖项映射到由从 CDN 加载的库定义的全局变量。

别名是通过 package.json 中的`alias`定义的。它们可以在最接近`package.json`包含依赖项的源文件中本地定义，也可以`package.json`在项目根目录中全局定义。全局别名适用于项目中的所有文件和包，包括`node_modules`。

### 包别名

包别名将`node_modules`依赖项映射到不同的包或项目中的本地文件。例如，要在项目中将`react`和`react-dom` 替换为 Preact 您可以在项目根目录`package.json`中定义全局别名。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "alias": {
    "react": "preact/compat",
    "react-dom": "preact/compat"
  }
}
```

{% endsamplefile %}
{% endsample %}

您还可以使用定义别名的相对路径将模块映射到项目中的`package.json`文件。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "alias": {
    "react": "./my-react.js"
  }
}
```

{% endsamplefile %}
{% endsample %}

也支持仅对模块的某些[子路径 sub-paths](#package-sub-paths)进行别名。此示例将别名`lodash/clone`为`tiny-clone`。`lodash`包中的其他导入将不受影响。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "alias": {
    "lodash/clone": "tiny-clone"
  }
}
```

{% endsamplefile %}
{% endsample %}

这也适用于另一种方式：如果整个模块有别名，则该包的任何子路径导入都将在别名模块中解析。例如，如果您使用别名将`lodash`为`my-lodash`，并且导入`lodash/clone`，这将解析为`my-lodash/clone`。

### 文件别名

别名也可以定义为相对路径，以用不同的文件替换包中的特定文件。这可以使用`alias`字段来无条件地替换文件，或者使用`source`或`browser`字段来有条件地完成。有关这些字段的详细信息，请参阅上面[包目录](#package-entries)。

例如，要将某个文件替换为特定于浏览器的版本，您可以使用该`browser`字段。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "browser": {
    "./fs.js": "./fs-browser.js"
  }
}
```

{% endsamplefile %}
{% endsample %}

现在，如果`my-module/fs.js`在浏览器环境中导入，它们实际上会得到`my-module/fs-browser.js`. 这既适用于从外部（例如[子路径 package sub-paths](#package-sub-paths)）的导入，也适用于模块内部的导入。

### 全局别名

文件别名也可以使用 glob 定义，它允许使用单个模式替换许多文件。替换可以包括诸如`$1`访问捕获的全局匹配的模式。这可以使用`alias`字段来无条件地替换文件，或者使用`source`或`browser`字段来有条件地完成。有关这些字段的详细信息，请参阅上面的[包目录 Package entries](#package-entries)。

例如，您可以使用`source`字段来提供包中已编译代码与原始源代码之间的映射。当模块被符号链接时，或者在 monorepo 中，这将允许 Parcel 从源代码编译模块，而不是使用预编译版本。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "source": {
    "./dist/*": "./src/$1"
  }
}
```

{% endsamplefile %}
{% endsample %}

现在，每次导入目录中的`src`文件时，都会加载文件夹 dist 中的相应文件。

### 垫片别名

文件或包可以别名为`false`从构建中排除，并替换为空模块。例如，这对于从仅在 Node.js 中工作的浏览器构建中排除某些模块可能很有用。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "alias": {
    "fs": false
  }
}
```

{% endsamplefile %}
{% endsample %}

### 全局别名

文件或包也可以别名为将在运行时定义的全局变量。例如，可以从 CDN 加载特定库。而不是捆绑它，只要解决了对该库的依赖关系，它将被替换为对该全局变量的引用，而不是被捆绑。

这可以通过为具有`global`属性的对象创建别名来完成。以下示例将`jquery`依赖说明符别名为全局变量`$`。

{% sample %}
{% samplefile "package.json" %}

```json
{
  "alias": {
    "jquery": {
      "global": "$"
    }
  }
}
```

{% endsamplefile %}
{% endsample %}

## 配置其他工具

本节介绍如何配置其他工具以使用 Parcel 对节点解析算法的扩展。

### TypeScript

需要将 TypeScript 配置为支持 Parcel 功能，例如绝对和波浪号依赖说明符以及别名。这可以使用 中的`paths`选项来完成`tsconfig.json`。有关更多信息，请参阅[TypeScript 模块解析](https://www.typescriptlang.org/docs/handbook/module-resolution.html)文档。

例如，要将波浪号路径映射到根目录，可以使用以下配置：

{% sample %}
{% samplefile "tsconfig.json" %}

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "~*": ["./*"]
    }
  }
}
```

{% endsamplefile %}
{% endsample %}

启用对[URL schemes](#url-schemes)的支持，可以通过在项目中创建[ambient module](https://www.typescriptlang.org/docs/handbook/modules.html#ambient-modules) 例如，要将随方案加载的依赖项映射到字符串，您可以使用以下声明。这可以放置在文件`parcel.d.ts`中，例如项目中的任何位置。

{% sample %}
{% samplefile "parcel.d.ts" %}

```typescript
declare module "bundle-text:*" {
  const value: string;
  export default value;
}
```

{% endsamplefile %}
{% endsample %}

### Flow

Flow 需要配置为支持绝对和波浪符说明符以及别名。这可以使用您[module.name_mapper](https://flow.org/en/docs/config/options/#toc-module-name-mapper-regex-string)的`.flowconfig`.

例如，要映射绝对说明符以从项目根目录解析，可以使用此配置：

{% sample %}
{% samplefile ".flowconfig" %}

```
[options]
module.name_mapper='^\/\(.*\)$' -> '<PROJECT_ROOT>/\1'
```

{% endsamplefile %}
{% endsample %}

要启用[url 方案 URL schemes](#url-schemes)您需要创建到导出预期类型的`.flow`[描述文件 declaration file](https://flow.org/en/docs/declarations/)。例如，要将随`bundle-text:`方案加载的依赖项映射到字符串，您可以创建一个名为的文件`bundle-text.js.flow`，并将引用该方案的所有依赖项映射到它。

{% sample %}
{% samplefile "bundle-text.js.flow" %}

```javascript
// @flow
declare var value: string;
export default value;
```

{% endsamplefile %}
{% samplefile ".flowconfig" %}

```
[options]
module.name_mapper='^bundle-text:.*$' -> '<PROJECT_ROOT>/bundle-text.js'
```

{% endsamplefile %}
{% endsample %}

---
layout: layout.njk
title: Scope hoisting
description: 在生产版本中，Parcel 将模块连接到一个作用域中。这被称为"scope hoisting"。Parcel 还静态地分析每个模块的导入和导出，并删除所有未使用的内容。这叫做"tree shaking"。
eleventyNavigation:
  key: features-scope-hoisting
  title: 🌳 Scope hoisting
  order: 7
---

从历史上看，JavaScript 捆绑器通过将每个模块包装在一个函数中来工作，该函数在模块被导入时被调用。这确保了每个模块都有单独的隔离范围和在预期时间运行的副作用，并支持[hot module replacement](/features/development/#hot-reloading)等开发功能。但是，所有这些单独的功能在下载大小和[runtime performance](https://nolanlawson.com/2016/08/15/the-cost-of-small-modules/)方面都是有代价的。

在生产构建中，Parcel 会尽可能将模块连接到单个作用域中，而不是将每个模块包装在单独的函数中。这称为**“scope hoisting”**。这有助于使缩小更有效，并通过使模块之间的引用静态而不是动态对象查找来提高运行时性能。

Parcel 还静态分析每个模块的导入和导出，并删除所有未使用的内容。这称为 **"tree shaking"** 或 **"dead code elimination"** 。静态或[dynamic import](/features/code-splitting/#tree-shaking), [CommonJS](/languages/javascript/#commonjs) 和 [ES modules](/languages/javascript/#es-modules), 甚至[CSS modules](/languages/css/#tree-shaking)都支持 Tree shaking。

## How scope hoisting 工作原理

Parcel 对 scope hoisting 的实施通过独立和并行分析每个模块，最后将它们连接在一起。为了使连接到单个作用域安全，每个模块的顶级变量都被重命名以确保它们是唯一的。此外，导入的变量被重命名以匹配从已解析模块中导出的变量名称。最后，删除所有未使用的导出。

{% sample %}
{% samplefile "index.js" %}

```javascript
import { add } from "./math";

console.log(add(2, 3));
```

{% endsamplefile %}
{% samplefile "math.js" %}

```javascript
export function add(a, b) {
  return a + b;
}

export function square(a) {
  return a * a;
}
```

{% endsamplefile %}
{% endsample %}

编译为：

```javascript
function $fa6943ce8a6b29$add(a, b) {
  return a + b;
}

console.log($fa6943ce8a6b29$add(2, 3));
```

如您所见，`add`函数已重命名，并且引用已更新以匹配。`square`功能已被删除，因为它未被使用。

与将每个模块包装在一个函数中相比，这会产生更小、更快的输出。不仅没有额外的函数，而且也没有`exports`对象，而且对`add`函数的引用是静态的，而不是属性查找。

## 避免纾困 Avoiding bail outs

Parcel 可以静态分析许多模式，包括 ES 模块`import`和`export`语句、CommonJS`require()`和`exports`赋值、动态`import()`解构和属性访问等等。但是，当遇到无法提前静态分析的代码时，Parcel 可能不得不“退出”并将模块包装在一个函数中，以保留副作用或允许在运行时解析导出。

要确定为什么树抖动未按预期发生，请使用`--log-level verbose`CLI 选项运行 Parcel。这将为发生的每个救助打印诊断信息，包括显示导致它的原因的代码框架。

```javascript
parcel build src/app.html --log-level verbose
```

### 动态成员访问

Parcel 可以静态解析构建时已知的成员访问，但是当使用动态属性访问时，必须将模块的所有导出都包含在构建中，并且 Parcel 必须创建一个导出对象，以便可以在运行时解析该值.

```javascript
import * as math from "./math";

// ✅ Static property access
console.log(math.add(2, 3));

// 🚫 Dynamic property access
console.log(math[op](2, 3));
```

此外，Parcel 不会跟踪命名空间对象对另一个变量的重新分配。除了静态属性访问之外，任何导入命名空间的使用都将导致包含所有导出。

```javascript
import * as math from "./math";

// 🚫 Reassignment of import namespace
let utils = math;
console.log(utils.add(2, 3));

// 🚫 Unknown usage of import namespace
doSomething(math);
```

### 动态导入

Parcel 支持带有静态属性访问或解构的树抖动动态导入。`await`和 Promise`then`语法都支持这一点。`import()`但是，如果以任何其他方式访问返回的 Promise ，Parcel 必须保留已解析模块的所有导出。

{% note %}

**注意:** 对于这种`await`情况，不幸的是，未使用的导出只能在`await`未转译时被删除（即使用现代浏览器列表配置）。

{% endnote %}

```swift
// ✅ Destructuring await
let {add} = await import('./math');

// ✅ Static member access of await
let math = await import('./math');
console.log(math.add(2, 3));

// ✅ Destructuring Promise#then
import('./math').then(({add}) => console.log(add(2, 3)));

// ✅ Static member access of Promise#then
import('./math').then(math => console.log(math.add(2, 3)));

// 🚫 Dynamic property access of await
let math = await import('./math');
console.log(math[op](2, 3));

// 🚫 Dynamic property access of Promise#then
import('./math').then(math => console.log(math[op](2, 3)));

// 🚫 Unknown use of returned Promise
doSomething(import('./math'));

// 🚫 Unknown argument passed to Promise#then
import('./math').then(doSomething);
```

### CommonJS

除了 ES 模块，Parcel 还可以分析很多 CommonJS 模块。Parcel 支持在 CommonJS 模块中对`exports`, `module.exports`，和`this`和进行静态分配。这意味着属性名称必须在构建时静态知道（即不是变量）。

当看到非静态模式时，Parcel 会创建一个`exports`所有导入模块在运行时访问的对象。所有导出都必须包含在最终构建中，并且不能执行 tree shaking。

```javascript
// ✅ Static exports assignments
exports.foo = 2;
module.exports.foo = 2;
this.foo = 2;

// ✅ module.exports assignment
module.exports = 2;

// 🚫 Dynamic exports assignments
exports[someVar] = 2;
module.exports[someVar] = 2;
this[someVar] = 2;

// 🚫 Exports re-assignment
let e = exports;
e.foo = 2;

// 🚫 Module re-assignment
let m = module;
m.exports.foo = 2;

// 🚫 Unknown exports usage
doSomething(exports);
doSomething(this);

// 🚫 Unknown module usage
doSomething(module);
```

在导入方面，Parcel 支持静态属性访问和`require`调用的解构。当看到非静态访问时，必须包含已解析模块的所有导出，并且不能执行 tree shaking。

```javascript
// ✅ Static property access
const math = require("./math");
console.log(math.add(2, 3));

// ✅ Static destructuring
const { add } = require("./math");

// ✅ Static property assignment
const add = require("./math").add;

// 🚫 Non-static property access
const math = require("./math");
console.log(math[op](2, 3));

// 🚫 Inline require
doSomething(require("./math"));
console.log(require("./math").add(2, 3));
```

### 避免 `eval`

`eval`函数在当前范围内执行字符串中的任意 JavaScript 代码。这意味着 Parcel 无法重命名范围内的任何变量，以防它们被`eval`。在这种情况下，Parcel 必须将模块包装在一个函数中，并避免缩小变量名称。

```javascript
let x = 2;

// 🚫 Eval causes wrapping and disables minification
eval("x = 4");
```

如果您需要从字符串运行 JavaScript 代码，您可以改用[Function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/Function)构造函数。

### 避免顶层 `return`

CommonJS 允许`return`在模块的顶层（即在函数之外）使用语句。当看到这种情况时，Parcel 必须将模块包装在一个函数中，以便执行只停止该模块而不是整个包。此外，由于导出可能不是静态已知的（例如，如果返回是有条件的），因此禁用了树抖动。

```javascript
exports.foo = 2;

if (someCondition) {
  // 🚫 Top-level return causes wrapping and disables tree shaking
  return;
}

exports.bar = 3;
```

### 避免`module`和`exports`重新分配

当 CommonJS`module`或`exports`变量被重新赋值时，Parcel 无法静态分析模块的导出。在这种情况下，必须将模块包装在一个函数中并且禁用 tree shaking。

```javascript
exports.foo = 2;

// 🚫 Exports reassignment causes wrapping and disables tree shaking
exports = {};

exports.foo = 5;
```

### 避免有条件的`require()`

`import`与只允许在模块顶层使用的 ES 模块语句不同，`require`与只允许在模块顶层使用的 ES 模块语句不同，它`require`从条件语句或其他控制流语句中调用时，Parcel 必须将解析的模块包装在一个函数中，以便在正确的时间执行副作用。这也递归地应用于已解析模块的任何依赖项。

```javascript
// 🚫 Conditional requires cause recursive wrapping
if (someCondition) {
  require("./something");
}
```

## 副作用

许多模块只包含声明，如函数或类，但有些可能还包含副作用。例如，一个模块可能会在 DOM 中插入一些东西，将一些东西记录到控制台，分配给一个全局变量（即 polyfill），或者初始化一个单例。必须始终保留这些副作用以使程序正常工作，即使模块的导出未使用。

默认情况下，Parcel 包含所有模块，以确保始终运行副作用。但是，`package.json`的`sideEffects`字段可用于向 Parcel 和其他工具提示您的文件是否包含副作用。这对于将库包含在其 package.json 文件中最有意义。

`sideEffects`字段支持以下值：

- `false` – 此包中的所有文件都没有副作用。
- `string` – 包含副作用的 glob 匹配文件。
- `Array<string>` – 组包含副作用的 glob 匹配文件。

当文件被标记为无副作用时，如果 Parcel 没有任何已使用的导出，则在连接捆绑包时可以跳过整个文件。这可以显着减少包大小，特别是如果模块在其初始化期间调用辅助函数。

{% sample null, "column" %}
{% samplefile "app.js" %}

```js
import { add } from "math";

console.log(add(2, 3));
```

{% endsamplefile %}

{% samplefile "node_modules/math/package.json" %}

```json
{
  "name": "math"
  "sideEffects": false
}
```

{% endsamplefile %}

{% samplefile "node_modules/math/index.js" %}

```js
export { add } from "./add.js";
export { multiply } from "./multiply.js";

let loaded = Date.now();
export function elapsed() {
  return Date.now() - loaded;
}
```

{% endsamplefile %}
{% endsample %}

在这种情况下，仅使用`math`库中的`add`函数。`multiply`和`elapsed`是未使用的。通常，`loaded`仍需要加载因为它包含了在模块初始化期间运行的副作用。但是，因为`package.json`包含`sideEffects`字段，所以可以完全跳过`index.js`模块。

除了尺寸优势之外，使用`sideEffects`字段还具有构建性能优势。在上面的例子中，因为 Parcel 知道`multiply.js`有副作用，并且没有使用它的导出，所以它甚至根本没有被编译。但是，如果`export *`改为使用，则不是这样，因为 Parcel 不知道有哪些导出可用。

另一个好处`sideEffects`是它也适用于打包。如果一个模块导入一个 CSS 文件或包含一个动态的`import()`，如果模块未被使用，则不会创建包。

### 纯注释 PURE annotations

您还可以使用注释来注释单个函数调用`/*#__PURE__*/`，这会告诉压缩器在结果未使用时删除该函数调用是安全的。

```javascript
export const radius = 23;
export const circumference = /*#__PURE__*/ calculateCircumference(radius);
```

在此示例中，如果`circumference`未使用导出，则`calculateCircumference`函数也将不包含在内。如果没有 PURE 注释，`calculateCircumference`仍然会调用它以防它有副作用。

---
layout: layout.njk
title: Node模拟
eleventyNavigation:
  key: features-node-emulation
  title: 🐢 Node模拟
  order: 8
---

Parcel 包含几个模拟 Node.js API 的功能。这允许 npm 上最初为 Node 编写的许多模块也可以在浏览器中工作。此外，许多浏览器模块也采用了基于 Node.js 的 API 来处理环境变量等问题。

## 环境变量

Parcel 支持 JavaScript 中的内联环境变量。这可用于确定构建环境（例如开发、登台、生产）、注入 API 密钥等。

要访问环境变量，请从对象中读取相应的属性`process.env`。

```js
if (process.env.NODE_ENV === "development") {
  console.log("Happy developing!");
}
```

您还可以使用解构语法一次访问多个属性。

```js
let { NODE_ENV, API_TOKEN } = process.env;
```

`process.env`不支持以任何非静态方式（例如动态属性查找）进行访问。

### `NODE_ENV`

Parcel 会根据`NODE_ENV`模式自动设置环境变量。运行`parcel build`时, `NODE_ENV`默认设置为`production`，否则设置为`development`。。这可以通过设置`NODE_ENV`（例如在 shell 中）来覆盖。

### `.env`文件

Parcel 支持加载在项目根目录中的`.env`件中定义的环境变量。这支持由换行符分隔的`NAME=value`对。以`#`开头的行被视为注释。有关更多详细信息，请参见[dotenv](https://github.com/motdotla/dotenv)。

{% sample %}
{% samplefile ".env" %}

```ini
APP_NAME=test
API_KEY=12345
```

{% endsamplefile %}
{% endsample %}

除了`.env`，还可以创建例如`.env.production`和`.env.development`这些是基于`NODE_ENV`环境变量应用的（包括由 Parcel 自动设置时）。未在特定于环境的覆盖中设置的任何变量都回退到基本`.env`文件中定义的值。

`.env.local`文件还支持环境变量的本地覆盖，但是，当`NODE_ENV=test`测试为每个人产生相同的结果时，不使用它。这也支持特定于环境的覆盖，例如`.env.production.local`.

## Polyfilling & Excluding Builtin Node Modules

填充和排除内置节点模块

当针对浏览器和您的代码，或者更可能是依赖项时，导入内置 Node 模块，例如`crypto`、`fs`或`process`，Parcel 将自动使用以下 polyfill 之一。如果没有可用的 polyfill，则将使用空模块。您还可以使用[aliases](/features/dependency-resolution/#aliases)来覆盖这些。

| 本机模块  | npm 替换                   | 本机模块       | npm 替换             |
| --------- | -------------------------- | -------------- | -------------------- |
| assert    | `assert`                   | process        | `process/browser.js` |
| buffer    | `buffer`                   | punycode       | `punycode`           |
| console   | `console-browserify`       | querystring    | `querystring-es3`    |
| constants | `constants-browserify`     | stream         | `stream-browserify`  |
| crypto    | `crypto-browserify`        | string_decoder | `string_decoder`     |
| domain    | `domain-browser`           | sys            | `util/util.js`       |
| events    | `events`                   | timers         | `timers-browserify`  |
| http      | `stream-http`              | tty            | `tty-browserify`     |
| https     | `https-browserify`         | url            | `url`                |
| os        | `os-browserify/browser.js` | util           | `util/util.js`       |
| path      | `path-browserify`          | vm             | `vm-browserify`      |
| zlib      | `browserify-zlib`          |

## Shimming Builtin Node Globals

填充内置节点全局变量

以浏览器为目标时，Node 中可用的全局变量的用法被替换为不破坏为 Node 编写的代码：

- `process` 从`process`模块中自动导入，除非它是由布尔值或环境变量值替换的`process.browser`或`process.env.FOO`表达式的一部分。

- `Buffer`从`buffer`模块中自动导入的。

- `__filename`和`dirname`替换为相对于项目根目录的资产文件路径（或父文件夹）的字符串文字。

- `global`替换为对全局变量的引用（表现得像 newer`globalThis`）。

## 内联 fs.readFileSync

如果文件路径是静态可确定的并且在项目根目录内，则调用将`fs.readFileSync`替换为文件的内容。

- `fs.readFileSync(__dirname + "/file", "utf8")` – 文件内容为字符串。支持“utf8”、“utf-8”、“hex”和“base64”编码。
- `fs.readFileSync(__dirname + "/file")` – 一个[Buffer](https://nodejs.org/dist/latest-v16.x/docs/api/buffer.html)对象。请注意，Buffer polyfill 非常大，因此这可能是不受欢迎的。

`__dirname`和`__filename`变量可以在文件名参数中使用。可以使用通过运算符`+`和函数`path.join`的字符串连接。不支持其他函数、变量或动态计算。计算路径应该始终是绝对的，并且不依赖于当前工作目录。

{% sample %}
{% samplefile "index.js" %}

```js/3
import fs from "fs";
import path from "path";

const data = fs.readFileSync(path.join(__dirname, "data.json"), "utf8");
console.log(data);
```

{% endsamplefile %}
{% samplefile "data.json" %}

```json
{
  "foo": "bar"
}
```

{% endsamplefile %}
{% endsample %}

## 禁用这些功能

可以通过在`package.json`中创建`@parcel/transformer-js`键来禁用[environment variables](#environment-variables)和[`readFileSync` calls](#inlining-fs.readfilesync)。

{% sample %}
{% samplefile "package.json" %}

```json5
{
  "name": "my-project",
  "dependencies": {
    ...
  },
  "@parcel/transformer-js": {
    "inlineFS": false,
    "inlineEnvironment": false
  }
}
```

{% endsamplefile %}
{% endsample %}

`inlineEnvironment`也可以是一个 glob 字符串数组，它允许您过滤允许的环境变量。这是确保安全的好主意，因为 node_modules 中的第三方代码也可以读取环境变量。

```json5
{
  "name": "my-project",
  "dependencies": {
    ...
  },
  "@parcel/transformer-js": {
    "inlineEnvironment": ["SENTRY_*"]
  }
}
```

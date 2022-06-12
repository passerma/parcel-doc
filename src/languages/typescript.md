---
layout: layout.njk
title: TypeScript
eleventyNavigation:
  key: languages-typescript
  title: <img src="/assets/lang-icons/typescript.svg" alt=""/> TypeScript
  order: 4
---

[TypeScript](https://www.typescriptlang.org/) 是 JavaScript 的类型化超集，可编译为 JavaScript。 Parcel 支持 TypeScript 开箱即用，无需任何额外配置。

## 转译

每当您使用 `.ts` 或 `.tsx` 文件时，Parcel 都会自动转译 TypeScript。除了剥离类型以将 TypeScript 转换为 JavaScript 之外，Parcel 还根据需要编译类和异步等待等现代语言功能，[根据您的浏览器目标](/languages/javascript/#browser-compatibility)。它还自动转换 [JSX](/languages/javascript/#jsx)。有关更多详细信息，请参阅 JavaScript 文档的 [Transpilation](/languages/javascript/#transpilation) 部分。

`tsconfig.json` 文件可用于配置转译的某些方面。目前，支持 JSX 选项，以及 `experimentalDecorators` 和 `useDefineForClassFields` 选项。有关详细信息，请参阅 [TSConfig 参考](https://www.typescriptlang.org/tsconfig)。

{% sample %}
{% samplefile "tsconfig.json" %}

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "jsxImportSource": "preact"
  }
}
```

{% endsamplefile %}
{% endsample %}

### `isolatedModules`

因为 Parcel 单独处理每个文件，所以它隐式启用 [`isolatedModules`](https://www.typescriptlang.org/tsconfig#isolatedModules) 选项。这意味着某些需要跨文件类型信息才能编译的 TypeScript 功能（例如 const enum ）将不起作用。要在 IDE 中和类型检查期间收到有关这些功能使用情况的警告，您应该在 `tsconfig.json` 中启用此选项。

{% sample %}
{% samplefile "tsconfig.json" %}

```json
{
  "compilerOptions": {
    "isolatedModules": true
  }
}
```

{% endsamplefile %}
{% endsample %}

### TSC

[TSC](https://www.typescriptlang.org/docs/handbook/compiler-options.html) 是 Microsoft 的官方 TypeScript 编译器。虽然 Parcel 的 TypeScript 默认转译器比 TSC 快得多，但如果您在 `tsconfig.json` 中使用 Parcel 不支持的某些配置，则可能需要使用 TSC。在这些情况下，您可以通过将 `@parcel/transformer-typescript-tsc` 插件添加到您的 `.parcelrc` 中来使用它。

{% sample %}
{% samplefile ".parcelrc" %}

```json/3
{
  "extends": "@parcel/config-default",
  "transformers": {
    "*.{ts,tsx}": ["@parcel/transformer-typescript-tsc"]
  }
}
```

{% endsamplefile %}
{% endsample %}

即使使用 TSC，Parcel 仍然单独处理每个 TypeScript 文件，所以关于 `isolatedModules` 的注释仍然适用。此外，Parcel 目前不支持某些解析功能，例如`paths`。 TSC 转换器也不执行任何类型检查（[见下文](#type-checking)）。

### Babel

你也可以选择使用 Babel 来编译 TypeScript。如果找到包含 `@babel/preset-typescript` 的 Babel 配置，Parcel 将使用它来编译 `.ts` 和 `.tsx` 文件。请注意，这与上述隔离模块具有相同的 [警告](https://babeljs.io/docs/en/babel-plugin-transform-typescript#caveats)。有关详细信息，请参阅 JavaScript 文档中的 [Babel](/languages/javascript/#babel)。

## Resolution

Parcel 目前不支持 `tsconfig.json` 中的 `baseUrl` 或 `paths` 选项，它们是 TypeScript 特定的分辨率扩展。相反，您可以使用 Parcel 的 [tilde](/features/dependency-resolution/#tilde-specifiers) 或 [absolute](/features/dependency-resolution/#absolute-specifiers) 说明符来实现类似的目标。有关如何配置 TypeScript 以支持这些工具的信息，请参阅依赖解析文档中的 [配置其他工具](/features/dependency-resolution/#configuring-other-tools)。

## 生成类型 Generating typings

在构建库时，Parcel 可以从您的入口点提取类型并生成一个 `.d.ts` 文件。使用 `package.json` 中的 `types` 字段以及诸如 `main` 或 `module` 之类的目标来启用此功能。

{% sample %}
{% samplefile "package.json" %}

```json/3
{
  "source": "src/index.ts",
  "module": "dist/index.js",
  "types": "dist/index.d.ts"
}
```

{% endsamplefile %}
{% endsample %}

有关详细信息，请参阅 [使用 Parcel 构建库](/getting-started/library/)。

## 类型检查

默认情况下，Parcel 不执行任何类型检查。推荐的类型检查方法是使用支持 TypeScript 的编辑器（例如 VSCode），并使用 `tsc` 在 CI 中对代码进行类型检查。您可以使用 npm 脚本对其进行配置，使其与您的构建、测试和 linting 一起运行。

{% sample %}
{% samplefile "package.json" %}

```json/5
{
  "scripts": {
    "build": "parcel build src/index.ts",
    "test": "jest",
    "lint": "eslint",
    "check": "tsc --noEmit",
    "ci": "yarn build && yarn test && yarn lint && yarn check"
  }
}
```

{% endsamplefile %}
{% endsample %}

### 实验性验证插件

`@parcel/validator-typescript` 插件是一种在 Parcel 构建中进行类型检查的实验性方法。 它在生成捆绑包后在后台运行。 确保 `tsconfig.json` 中的 `include` 选项包含所有源文件。

{% sample %}
{% samplefile ".parcelrc" %}

```json/3
{
  "extends": "@parcel/config-default",
  "validators": {
    "*.{ts,tsx}": ["@parcel/validator-typescript"]
  }
}
```

{% endsamplefile %}
{% samplefile "tsconfig.json" %}

```json
{
  "include": ["src/**/*"],
  "compilerOptions": {
    "target": "es2021",
    "strict": true
  }
}
```

{% endsamplefile %}
{% endsample %}

{% warning %}

**警告**：包裹验证器插件是实验性的，可能不稳定。

{% endwarning %}

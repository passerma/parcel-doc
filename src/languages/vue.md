---
layout: layout.njk
title: Vue
eleventyNavigation:
  key: languages-vue
  title: <img src="/assets/lang-icons/vue.svg" alt=""/> Vue
  order: 9
---

[Vue.js](https://v3.vuejs.org) 是一个渐进式、可增量采用的 JavaScript 框架，用于在 Web 上构建 UI。 Parcel 使用 `@parcel/transformer-vue` 插件自动支持 Vue。当检测到 `.vue` 文件时，它会自动安装到您的项目中。

{% note %}

**注意**：Parcel 不支持在 Vue 2 中使用 SFC，您必须使用 [Vue 3](https://github.com/vuejs/core) 或更高版本。

{% endnote %}

## 示例用法

{% sample %}
{% samplefile "index.html" %}

```html
<!DOCTYPE html>
<div id="app"></div>
<script type="module" src="./index.js"></script>
```

{% endsamplefile %}
{% samplefile "index.js" %}

```jsx
import { createApp } from "vue";
import App from "./App.vue";

const app = createApp(App);
app.mount("#app");
```

{% endsamplefile %}
{% samplefile "App.vue" %}

```html
<template>
  <div>Hello {% raw %}{{ name }}{% endraw %}!</div>
</template>

<script>
  export default {
    data() {
      return {
        name: "Vue",
      };
    },
  };
</script>
```

{% endsamplefile %}
{% endsample %}

## HMR

Parcel 使用官方 Vue SFC 编译器，它支持开箱即用的 HMR，因此您将获得快速、反应式的开发体验。有关 Parcel 中 HMR 的更多详细信息，请参阅 [热重载 Hot reloading](/features/development/#hot-reloading)。

## Vue 3 功能

由于 Parcel 使用 Vue 3，您可以使用所有 Vue 3 功能，例如 [Composition API](https://vuejs.org/guide/extras/composition-api-faq.html)。

{% sample %}
{% samplefile "App.vue" %}

```html
<template>
  <button @click="increment">
    Count is: {% raw %}{{ state.count }}{% endraw %} Double is: {% raw %}{{
    state.double }}{% endraw %}
  </button>
</template>

<script>
  import { reactive, computed } from "vue";

  export default {
    setup() {
      const state = reactive({
        count: 0,
        double: computed(() => state.count * 2),
      });

      function increment() {
        state.count++;
      }

      return {
        state,
        increment,
      };
    },
  };
</script>
```

{% endsamplefile %}
{% endsample %}

## 语言支持

Parcel 支持 [JavaScript](/languages/javascript/)、[TypeScript](/languages/typescript/) 和 [CoffeeScript](/languages/coffeescript/) 作为 Vue 中的脚本语言。

几乎可以使用任何模板语言（[consolidate](https://www.npmjs.com/package/consolidate) 支持的所有模板语言）。

对于样式，支持 [Less](/languages/less)、[Sass](/languages/sass) 和 [Stylus](/languages/stylus)。此外，[CSS Modules](/languages/css/#css-modules) 和 [scoped style](https://vue-loader.vuejs.org/guide/scoped-css.html) 可以与 ` module` 和 `scoped` 修饰符。

{% sample %}
{% samplefile "App.vue" %}

```html
<style lang="scss" scoped>
  /* This style will only apply to this module */
  $red: red;
  h1 {
    background: $red;
  }
</style>

<style lang="less">
  @green: green;
  h1 {
    color: @green;
  }
</style>

<style src="./App.module.css">
  /* The content of blocks with a `src` attribute is ignored and replaced with
   the content of `src`. */
</style>

<template lang="pug"> div h1 This is the app </template>

<script lang="coffee">
  module.exports =
    data: ->
      msg: 'Hello from coffee!'
</script>
```

{% endsamplefile %}
{% endsample %}

## 自定义块 Custom Blocks

您可以在 Vue 组件中使用自定义块，但必须使用 `.vuerc`、`vue.config.js` 等配置 Vue 来定义如何预处理这些块。

{% sample %}
{% samplefile ".vuerc" %}

```json
{
  "customBlocks": {
    "docs": "./src/docs-preprocessor.js"
  }
}
```

{% endsamplefile %}
{% samplefile "src/docs-preprocessor.js" %}

```js
export default function (component, blockContent, blockAttrs) {
  if (blockAttrs.brief) {
    component.__briefDocs = blockContent;
  } else {
    component.__docs = blockContent;
  }
}
```

{% endsamplefile %}
{% samplefile "HomePage.vue" %}

```html
<template>
  <div>Home Page</div>
</template>

<docs> This component represents the home page of the application. </docs>

<docs brief> Home Page </docs>
```

{% endsamplefile %}
{% samplefile "App.vue" %}

```html
<template>
  <div>
    <child></child>
    docs: {% raw %}{{ docs.standard }}{% endraw %} in brief: {% raw %}{{
    docs.brief }}{% endraw %}
  </div>
</template>

<script>
  import Child from "./HomePage.vue";
  export default {
    components: {
      child: Child,
    },
    data() {
      let docs = { standard: Child.__docs, brief: Child.__docsBrief };
      return { docs };
    },
  };
</script>
```

{% endsamplefile %}
{% endsample %}

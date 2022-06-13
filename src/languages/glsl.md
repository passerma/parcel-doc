---
layout: layout.njk
title: GLSL
eleventyNavigation:
  key: languages-glsl
  title: <img src="/assets/lang-icons/openGL.svg" alt=""/> GLSL
  order: 14
---

Parcel 支持使用 `@parcel/transformer-glsl` 插件导入 [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API) 着色器。当检测到 `.glsl`、`.vert` 或 `.frag` 文件时，它会自动安装到您的项目中。

## 示例用法

GLSL 文件作为字符串导入 JavaScript，您可以将其加载到 WebGL 上下文中。

```js
import frag from './shader.frag'

// ...
gl.shaderSource(..., frag);
// ...
```

### 依赖项

Parcel 还使用编译指示支持 GLSL 文件中的依赖关系，包括来自 node_modules 中的库。这些被捆绑到一个着色器中，您可以将其加载到 WebGL 上下文中。

{% sample %}
{% samplefile "app.js" %}

```js
import frag from './shader.frag';

// ...
gl.shaderSource(..., frag);
// ...
```

{% endsamplefile %}
{% samplefile "shader.frag" %}

```glsl
// import a function from another file
#pragma glslify: calc_frag_color = require('./lib.glsl')

precision mediump float;
varying vec3 vpos;

void main() {
  gl_FragColor = calc_frag_color(vpos);
}
```

{% endsamplefile %}
{% samplefile "lib.glsl" %}

```glsl
// import a function from node_modules
#pragma glslify: noise = require('glsl-noise/simplex/3d')

vec4 calc_frag_color(vec3 pos) {
  return vec4(noise(pos * 25.0), 1);
}

// export a function
#pragma glslify: export(calc_frag_color)

```

{% endsamplefile %}
{% endsample %}

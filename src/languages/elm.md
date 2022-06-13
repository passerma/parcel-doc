---
layout: layout.njk
title: Elm
eleventyNavigation:
  key: languages-elm
  title: <img src="/assets/lang-icons/elm.svg" alt=""/> Elm
  order: 10
---

您可以像导入任何其他 JavaScript 文件一样导入 [Elm](https://elm-lang.org/) 文件。

npm 包 `elm` 需要事先手动安装。您还需要一个 `elm.json` 配置文件（运行 `yarn elm init` 开始并在必要时对其进行修改）。

{% sample null, "column" %}
{% samplefile "index.html" %}

```html
<!DOCTYPE html>
<div id="root"></div>
<script type="module" src="index.js"></script>
```

{% endsamplefile %}

{% samplefile "index.js" %}

```js
import { Elm } from "./Main.elm";

Elm.Main.init({ node: document.getElementById("root") });
```

{% endsamplefile %}

{% samplefile "Main.elm" %}

```elm
module Main exposing (..)

import Browser
import Html exposing (Html, button, div, text)
import Html.Events exposing (onClick)

main =
  Browser.sandbox { init = init, update = update, view = view }

type alias Model = Int

init : Model
init =
  0

type Msg = Increment | Decrement

update : Msg -> Model -> Model
update msg model =
  case msg of
    Increment ->
      model + 1

    Decrement ->
      model - 1


view : Model -> Html Msg
view model =
  div []
    [ button [ onClick Decrement ] [ text "-" ]
    , div [] [ text (String.fromInt model) ]
    , button [ onClick Increment ] [ text "+" ]
    ]
```

{% endsamplefile %}

{% endsample %}

## 时间旅行调试器 Time-travelling debugger

Elm 的调试模式在不为生产而构建时会自动启用（使用 `parcel build` 会自动禁用）。即使在开发模式下，您也可以设置环境变量 `PARCEL_ELM_NO_DEBUG=1` 来禁用它。

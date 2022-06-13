---
layout: layout.njk
title: GraphQL
eleventyNavigation:
  key: languages-graphql
  title: <img src="/assets/lang-icons/graphql.svg" alt=""/> GraphQL
  order: 13
---

Parcel 支持通过 `@parcel/transformer-graphql` 插件将在单独文件中定义的 [GraphQL](https://graphql.org) 查询导入 JavaScript。当检测到 `.graphql` 或 `.gql` 文件时，它会自动安装到您的项目中。

## 示例用法

GraphQL 文件作为字符串导入 JavaScript，您可以直接将其发送到服务器或与您喜欢的任何 GraphQL 库一起使用。

{% sample %}
{% samplefile "app.js" %}

```js
import query from "./query.graphql";
```

{% endsamplefile %}
{% samplefile "query.graphql" %}

```graphql
{
  user(id: 5) {
    firstName
    lastName
  }
}
```

{% endsamplefile %}
{% endsample %}

### 依赖项

Parcel 还支持使用特殊的注释语法将单独文件中定义的片段导入另一个 GraphQL 文件。这些将被捆绑到一个单一的 GraphQL 查询中，并作为字符串返回到您的代码中。

您可以从文件中导入所有片段：

```graphql
# import "fragments.graphql"
# import * from "fragments.graphql"
```

或列出您要导入的特定片段：

```graphql
# import UserFragment, AddressFragment from "fragments.graphql"
```

下面是一个完整示例，展示了如何将导入用作更大的 GraphQL 查询的一部分：

{% sample %}
{% samplefile "query.graphql" %}

```graphql
# import UserFragment from "user.graphql"
# import "address.graphql"

query UserQuery($id: ID) {
  user(id: $id) {
    ...UserFragment
    address {
      ...AddressFragment
    }
  }
}
```

{% endsamplefile %}
{% samplefile "user.graphql" %}

```graphql
fragment UserFragment on User {
  firstName
  lastName
}
```

{% endsamplefile %}
{% samplefile "address.graphql" %}

```graphql
fragment AddressFragment on Address {
  city
  state
  country
}
```

{% endsamplefile %}
{% endsample %}

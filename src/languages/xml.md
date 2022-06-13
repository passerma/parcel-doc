---
layout: layout.njk
title: XML
eleventyNavigation:
  key: languages-xml
  title: <img src="/assets/lang-icons/xml.svg" class="dark-invert" alt=""/> XML
  order: 17
---

Parcel 支持使用`@parcel/transformer-xml` 插件转换 XML 文件中定义的[RSS](https://en.wikipedia.org/wiki/RSS)和[Atom](<https://en.wikipedia.org/wiki/atom_(web_standard)>)提要。如果检测到 `.xml`，`.rss`或`.atom`文件，它将自动安装到您的项目中。

## 依赖

Parcel 转换 RSS 和 Atom 提要中的 URL 引用以匹配最终名称和 [public URL](/features/targets/#publicurl)，包括 [content hashes](/features/production/#content-hashing) 在适当的地方。

在 RSS 中，这包括：

- `<link>`
- `<url>`
- `<comments>`
- `<enclosure>`

在 Atom 中，这包括：

- `<link>`
- `<icon>`
- `<logo>`

## 嵌入式 HTML

RSS 和 Atom 提要中嵌入的 HTML 和 XHTML 内容也会按照 [HTML](/languages/html/) 中的说明进行转换。嵌入 HTML 中的所有 URL 引用也将被转换，并且引用的文件将使用相关的 Parcel 管道进行处理。

## HTML 引用

可以使用 `<link>` 元素从 HTML 文件中引用 RSS 和 Atom 提要。根据需要使用 `application/rss+xml` 或 `application/atom+xml` mime 类型。 Parcel 将确保以这种方式引用的 XML 文件不会收到内容哈希，并且随着时间的推移具有一致的 URL。

```html
<link
  href="feed.xml"
  rel="alternate"
  type="application/rss+xml"
  title="Blog RSS feed"
/>
```

## 例子

此示例显示了一个包含单个条目的 Atom 提要。两个 `<link>` 元素中的 URL 引用将被重写以包含公共 URL，并且帖子内容中引用的图像将被处理和内容散列。

```xml
<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
  <title>Example Feed</title>
  <subtitle>A subtitle.</subtitle>
  <link href="/" />
  <id>urn:uuid:60a76c80-d399-11d9-b91C-0003939e0af6</id>
  <updated>2021-12-13T18:30:02Z</updated>
  <entry>
    <title>Awesome post</title>
    <link href="post.html" />
    <id>urn:uuid:1225c695-cfb8-4ebb-aaaa-80da344efa6a</id>
    <updated>2021-12-13T18:30:02Z</updated>
    <summary>Some text.</summary>
    <content type="xhtml">
      <div xmlns="http://www.w3.org/1999/xhtml">
        <p>This is the entry content.</p>
        <img src="image.png" />
      </div>
    </content>
    <author>
      <name>John Doe</name>
      <email>johndoe@example.com</email>
    </author>
  </entry>
</feed>
```

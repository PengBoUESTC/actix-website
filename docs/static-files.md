---
title: 静态文件
---

import CodeBlock from "@site/src/components/code_block";

# 单个文件

可以使用自定义路径模式和 `NamedFile` 来提供静态文件。要匹配路径尾部，我们可以使用 `[.*]` 正则表达式。

<CodeBlock example="static-files" file="main.rs" section="individual-file" />

:::warning
使用 `[.*]` 正则表达式匹配路径尾部并返回 `NamedFile` 有严重的安全隐患。它可能允许攻击者在 URL 中插入 `../`，从而访问服务器用户有权限访问的所有主机文件。
:::

## 目录

要从特定目录和子目录提供文件，可以使用 [`Files`][files]。`Files` 必须通过 `App::service()` 方法注册，否则将无法提供子路径服务。

<CodeBlock example="static-files" file="directory.rs" section="directory" />

默认情况下，子目录的文件列表是禁用的。尝试加载目录列表将返回 _404 Not Found_ 响应。要启用文件列表，请使用 [`Files::show_files_listing()`][showfileslisting] 方法。

除了显示目录的文件列表，还可以重定向到特定的索引文件。使用 [`Files::index_file()`][indexfile] 方法配置此重定向。

## 配置

`NamedFiles` 可以指定各种选项来提供文件：

- `set_content_disposition` - 用于将文件的 mime 映射到相应 `Content-Disposition` 类型的函数
- `use_etag` - 指定是否应计算并包含 `ETag` 在头信息中。
- `use_last_modified` - 指定是否应使用文件修改时间戳并添加到 `Last-Modified` 头信息中。

以上所有方法都是可选的，并提供了最佳默认值，但可以自定义其中的任何一个。

<CodeBlock example="static-files" file="configuration.rs" section="config-one" />

配置也可以应用于目录服务：

<CodeBlock example="static-files" file="configuration_two.rs" section="config-two" />

[files]: https://docs.rs/actix-files/0.6/actix_files/struct.Files.html#
[showfileslisting]: https://docs.rs/actix-files/0.6/actix_files/struct.Files.html#method.show_files_listing
[indexfile]: https://docs.rs/actix-files/0.6/actix_files/struct.Files.html#method.index_file

---
title: 提取器
---

import CodeBlock from "@site/src/components/code_block";

# 类型安全的信息提取

Actix Web 提供了一种称为 _提取器_（即 `impl FromRequest`）的类型安全请求信息访问工具。有许多内置的提取器实现（参见 [实现者](https://docs.rs/actix-web/latest/actix_web/trait.FromRequest.html#implementors)）。

提取器可以作为处理函数的参数进行访问。Actix Web 支持每个处理函数最多 12 个提取器。参数位置无关紧要。

<CodeBlock example="extractors" file="main.rs" section="option-one" />

## 路径

[_Path_][pathstruct] 提供从请求路径中提取的信息。路径中可提取的部分称为“动态段”，并用花括号标记。你可以从路径中反序列化任何变量段。

例如，对于注册了 `/users/{user_id}/{friend}` 路径的资源，可以反序列化两个段，`user_id` 和 `friend`。这些段可以按声明的顺序作为元组提取（例如，`Path<(u32, String)>`）。

<CodeBlock example="extractors" file="path_one.rs" section="path-one" />

还可以通过将动态段名称与字段名称匹配，将路径信息提取到实现了 `serde` 的 `Deserialize` 特性的类型中。以下是使用 `serde` 的反序列化结构（确保启用了其 `derive` 特性）而不是元组类型的等效示例。

<CodeBlock example="extractors" file="path_two.rs" section="path-two" />

作为一种非类型安全的替代方法，还可以在处理函数中按名称查询请求的路径参数（参见 [`match_info` 文档][docsrs_match_info]）：

<CodeBlock example="extractors" file="path_three.rs" section="path-three" />

## 查询

[`Query<T>`][querystruct] 类型提供了请求查询参数的提取功能。其底层使用 `serde_urlencoded` crate。

<CodeBlock example="extractors" file="query.rs" section="query" />

## JSON

[`Json<T>`][jsonstruct] 允许将请求体反序列化为结构体。要从请求体中提取类型化信息，类型 `T` 必须实现 `serde::Deserialize`。

<CodeBlock example="extractors" file="json_one.rs" section="json-one" />

一些提取器提供了配置提取过程的方法。要配置提取器，将其配置对象传递给资源的 `.app_data()` 方法。在 _Json_ 提取器的情况下，它返回一个 [_JsonConfig_][jsonconfig]。你可以配置 JSON 负载的最大大小以及自定义错误处理函数。

以下示例将负载大小限制为 4kb 并使用自定义错误处理程序。

<CodeBlock example="extractors" file="json_two.rs" section="json-two" />

## URL 编码表单

URL 编码的表单体可以像 `Json<T>` 一样提取到结构体中。此类型必须实现 `serde::Deserialize`。

[_FormConfig_][formconfig] 允许配置提取过程。

<CodeBlock example="extractors" file="form.rs" section="form" />

## 其他

Actix Web 还提供了许多其他提取器，以下是一些重要的：

- [`Data`][datastruct] - 用于访问应用程序状态的部分。
- [`HttpRequest`][httprequest] - `HttpRequest` 本身就是一个提取器，以防你需要访问请求的其他部分。
- `String` - 你可以将请求的负载转换为 `String`。[_一个示例_][stringexample] 在 rustdoc 中可用。
- [`Bytes`][bytes] - 你可以将请求的负载转换为 _Bytes_。[_一个示例_][bytesexample] 在 rustdoc 中可用。
- [`Payload`][payload] - 主要用于构建其他提取器的低级负载提取器。[_一个示例_][payloadexample] 在 rustdoc 中可用。

## 应用程序状态提取器

应用程序状态可以通过 `web::Data` 提取器从处理函数中访问；然而，状态只能作为只读引用访问。如果你需要对状态进行可变访问，则必须实现它。

以下是一个存储处理请求数量的处理函数示例：

<CodeBlock example="request-handlers" file="main.rs" section="data" />

尽管此处理函数可以工作，但 `data.count` 只会计算 _每个工作线程_ 处理的请求数量。要计算所有线程处理的总请求数，应使用共享的 `Arc` 和 [原子操作][atomics]。

<CodeBlock example="request-handlers" file="handlers_arc.rs" section="arc" />

**注意**：如果你希望 _整个_ 状态在所有线程之间共享，请使用 `web::Data` 和 `app_data`，如 [共享可变状态][shared_mutable_state] 中所述。

在应用程序状态中使用阻塞同步原语（如 `Mutex` 或 `RwLock`）时要小心。Actix Web 异步处理请求。如果处理函数中的 [_临界区_][critical_section] 太大或包含 `.await` 点，这将是一个问题。如果这是一个问题，我们建议你也阅读 [Tokio 关于在异步代码中使用阻塞 `Mutex` 的建议][tokio_std_mutex]。

[pathstruct]: https://docs.rs/actix-web/4/actix_web/dev/struct.Path.html
[querystruct]: https://docs.rs/actix-web/4/actix_web/web/struct.Query.html
[jsonstruct]: https://docs.rs/actix-web/4/actix_web/web/struct.Json.html
[jsonconfig]: https://docs.rs/actix-web/4/actix_web/web/struct.JsonConfig.html
[formconfig]: https://docs.rs/actix-web/4/actix_web/web/struct.FormConfig.html
[datastruct]: https://docs.rs/actix-web/4/actix_web/web/struct.Data.html
[httprequest]: https://docs.rs/actix-web/4/actix_web/struct.HttpRequest.html
[stringexample]: https://docs.rs/actix-web/4/actix_web/trait.FromRequest.html#impl-FromRequest-for-String
[bytes]: https://docs.rs/actix-web/4/actix_web/web/struct.Bytes.html
[bytesexample]: https://docs.rs/actix-web/4/actix_web/trait.FromRequest.html#impl-FromRequest-5
[payload]: https://docs.rs/actix-web/4/actix_web/web/struct.Payload.html
[payloadexample]: https://docs.rs/actix-web/4/actix_web/web/struct.Payload.html
[docsrs_match_info]: https://docs.rs/actix-web/latest/actix_web/struct.HttpRequest.html#method.match_info
[actix]: /actix/docs/
[atomics]: https://doc.rust-lang.org/std/sync/atomic/
[shared_mutable_state]: /docs/application#shared-mutable-state
[critical_section]: https://en.wikipedia.org/wiki/Critical_section
[tokio_std_mutex]: https://tokio.rs/tokio/tutorial/shared-state#on-using-stdsyncmutex

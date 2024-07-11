---
title: 请求
---

import CodeBlock from "@site/src/components/code_block";

# JSON 请求

有几种选项可以用于 JSON 请求体的反序列化。

第一种选项是使用 _Json_ 提取器。首先，你定义一个接受 `Json<T>` 作为参数的处理函数，然后使用 `.to()` 方法注册这个处理函数。也可以通过使用 `serde_json::Value` 作为类型 `T` 来接受任意有效的 JSON 对象。

第一个 `JSON 请求` 的示例依赖于 `serde`：

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
```

第二个 `JSON 请求` 的示例依赖于 `serde`、`serde_json` 和 `futures`：

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1"
futures = "0.3"
```

如果你想为字段添加默认值，请参考 `serde` 的[文档](https://serde.rs/attr-default.html)。

<CodeBlock example="requests" file="main.rs" section="json-request" />

你也可以手动将负载加载到内存中，然后进行反序列化。

在以下示例中，我们将反序列化一个 _MyObj_ 结构体。我们需要先加载请求体，然后将 JSON 反序列化为一个对象。

<CodeBlock example="requests" file="manual.rs" section="json-manual" />

> 两种选项的完整示例可在[示例目录][examples]中找到。

## 内容编码

Actix Web 自动解压负载。支持以下编解码器：

- Brotli
- Gzip
- Deflate
- Zstd

如果请求头包含 `Content-Encoding` 头，则根据头的值解压请求负载。不支持多种编解码器，即：`Content-Encoding: br, gzip`。

## 分块传输编码

Actix 自动解码 _chunked_ 编码。[`web::Payload`][payloadextractor] 提取器已经包含了解码后的字节流。如果请求负载使用支持的压缩编解码器（br、gzip、deflate）进行压缩，则字节流会被解压。

## 多部分体

Actix Web 提供了一个外部 crate [`actix-multipart`][multipartcrate] 来支持多部分流。

> 完整示例可在[示例目录][multipartexample]中找到。

## Urlencoded 请求体

Actix Web 提供了对 _application/x-www-form-urlencoded_ 编码请求体的支持，使用 [`web::Form`][formencoded] 提取器，该提取器解析为反序列化实例。实例的类型必须实现 _serde_ 的 `Deserialize` 特性。

_UrlEncoded_ future 在以下几种情况下可能会解析为错误：

- 内容类型不是 `application/x-www-form-urlencoded`
- 传输编码是 `chunked`
- 内容长度大于 256k
- 负载终止时出错

<CodeBlock example="requests" file="urlencoded.rs" section="urlencoded" />

## 流式请求

_HttpRequest_ 是一个 `Bytes` 对象的流。它可以用来读取请求体负载。

在以下示例中，我们逐块读取并打印请求负载：

<CodeBlock example="requests" file="streaming.rs" section="streaming" />

[examples]: https://github.com/actix/examples/tree/master/json/json
[multipartstruct]: https://docs.rs/actix-multipart/0.2/actix_multipart/struct.Multipart.html
[fieldstruct]: https://docs.rs/actix-multipart/0.2/actix_multipart/struct.Field.html
[multipartexample]: https://github.com/actix/examples/tree/master/forms/multipart
[urlencoded]: https://docs.rs/actix-web/4/actix_web/dev/struct.UrlEncoded.html
[payloadextractor]: https://docs.rs/actix-web/4/actix_web/web/struct.Payload.html
[multipartcrate]: https://crates.io/crates/actix-multipart
[formencoded]: https://docs.rs/actix-web/4/actix_web/web/struct.Form.html

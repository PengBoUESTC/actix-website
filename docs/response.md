---
title: 响应
---

import CodeBlock from "@site/src/components/code_block";

# 响应

使用类似构建器的模式来构造 `HttpResponse` 实例。`HttpResponse` 提供了几种方法，这些方法返回一个 `HttpResponseBuilder` 实例，该实例实现了各种便捷的方法来构建响应。

> 查看 [文档][responsebuilder] 了解类型描述。

方法 `.body`、`.finish` 和 `.json` 完成响应创建并返回构造好的 _HttpResponse_ 实例。如果在同一个构建器实例上多次调用这些方法，构建器将会崩溃。

<CodeBlock example="responses" file="main.rs" section="builder" />

## JSON 响应

`Json` 类型允许使用格式良好的 JSON 数据进行响应：只需返回类型为 `Json<T>` 的值，其中 `T` 是要序列化为 _JSON_ 的结构类型。类型 `T` 必须实现 _serde_ 的 `Serialize` 特性。

要使以下示例工作，您需要在 `Cargo.toml` 中添加 `serde` 到您的依赖项：

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
```

<CodeBlock example="responses" file="json_resp.rs" section="json-resp" />

以这种方式使用 `Json` 类型而不是在 `HttpResponse` 上调用 `.json` 方法，可以立即清楚地表明该函数返回的是 JSON 而不是其他类型的响应。

## 内容编码

Actix Web 可以使用 [_Compress 中间件_][compressmidddleware] 自动压缩负载。支持以下编解码器：

- Brotli
- Gzip
- Deflate
- Identity

响应的 `Content-Encoding` 头默认设置为 `ContentEncoding::Auto`，它根据请求的 `Accept-Encoding` 头执行自动内容压缩协商。

<CodeBlock example="responses" file="auto.rs" section="auto" />

通过将 `Content-Encoding` 设置为 `Identity` 值，可以显式禁用处理程序上的内容压缩：

<CodeBlock example="responses" file="identity.rs" section="identity" />

在处理已经压缩的主体时（例如，提供预压缩的资产），手动设置响应上的 `Content-Encoding` 头以绕过中间件：

<CodeBlock example="responses" file="identity_two.rs" section="identity-two" />

[responsebuilder]: https://docs.rs/actix-web/4/actix_web/struct.HttpResponseBuilder.html
[compressmidddleware]: https://docs.rs/actix-web/4/actix_web/middleware/struct.Compress.html

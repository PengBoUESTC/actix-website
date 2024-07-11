---
title: 处理器
---

import CodeBlock from "@site/src/components/code_block";

# 请求处理器

请求处理器是一个异步函数，它接受零个或多个可以从请求中提取的参数（即 [_impl FromRequest_][implfromrequest]），并返回一个可以转换为 HttpResponse 的类型（即 [_impl Responder_][respondertrait]）。

请求处理分为两个阶段。首先调用处理器对象，返回任何实现 [_Responder_][respondertrait] 特性的对象。然后，对返回的对象调用 `respond_to()`，将其转换为 `HttpResponse` 或 `Error`。

默认情况下，Actix Web 为一些标准类型提供了 `Responder` 实现，例如 `&'static str`、`String` 等。

> 有关实现的完整列表，请查看 [_Responder 文档_][responderimpls]。

有效处理器的示例：

```rust
async fn index(_req: HttpRequest) -> &'static str {
    "Hello world!"
}
```

```rust
async fn index(_req: HttpRequest) -> String {
    "Hello world!".to_owned()
}
```

你也可以将签名更改为返回 `impl Responder`，这在涉及更复杂类型时效果很好。

```rust
async fn index(_req: HttpRequest) -> impl Responder {
    web::Bytes::from_static(b"Hello world!")
}
```

```rust
async fn index(req: HttpRequest) -> Box<Future<Item=HttpResponse, Error=Error>> {
    ...
}
```

## 使用自定义类型响应

要直接从处理器函数返回自定义类型，该类型需要实现 `Responder` 特性。

让我们为一个序列化为 `application/json` 响应的自定义类型创建一个响应：

<CodeBlock example="responder-trait" file="main.rs" section="responder-trait" />

## 流式响应体

响应体可以异步生成。在这种情况下，响应体必须实现流特性 `Stream<Item = Result<Bytes, Error>>`，即：

<CodeBlock example="async-handlers" file="stream.rs" section="stream" />

## 不同的返回类型（Either）

有时，你需要返回不同类型的响应。例如，你可以进行错误检查并返回错误，返回异步响应，或任何需要两种不同类型结果的情况。

对于这种情况，可以使用 [Either][either] 类型。`Either` 允许将两种不同的响应类型组合成一种类型。

<CodeBlock example="either" file="main.rs" section="either" />

[implfromrequest]: https://docs.rs/actix-web/4/actix_web/trait.FromRequest.html
[respondertrait]: https://docs.rs/actix-web/4/actix_web/trait.Responder.html
[responderimpls]: https://docs.rs/actix-web/4/actix_web/trait.Responder.html#foreign-impls
[either]: https://docs.rs/actix-web/4/actix_web/enum.Either.html

---
title: 错误处理
---

import CodeBlock from "@site/src/components/code_block";

# 错误处理

Actix Web 使用其自定义的 [`actix_web::error::Error`][actixerror] 类型和 [`actix_web::error::ResponseError`][responseerror] 特性来处理来自 Web 处理器的错误。

如果一个处理器返回一个 `Error`（指的是 [Rust 通用特性 `std::error::Error`][stderror]）并且该 `Result` 也实现了 `ResponseError` 特性，Actix Web 将会将该错误渲染为一个 HTTP 响应，并带有相应的 [`actix_web::http::StatusCode`][status_code]。默认情况下会生成一个内部服务器错误：

```rust
pub trait ResponseError {
    fn error_response(&self) -> HttpResponse<BoxBody>;
    fn status_code(&self) -> StatusCode;
}
```

一个 `Responder` 会将兼容的 `Result` 强制转换为 HTTP 响应：

```rust
impl<T: Responder, E: Into<Error>> Responder for Result<T, E>
```

上面的代码中的 `Error` 是 actix-web 的错误定义，任何实现了 `ResponseError` 的错误都可以自动转换为该错误。

Actix Web 为一些常见的非 actix 错误提供了 `ResponseError` 实现。例如，如果一个处理器返回一个 `io::Error`，该错误将被转换为 `HttpInternalServerError`：

```rust
use std::io;
use actix_files::NamedFile;

fn index(_req: HttpRequest) -> io::Result<NamedFile> {
    Ok(NamedFile::open("static/index.html")?)
}
```

请参阅 [Actix Web API 文档][responseerrorimpls] 以获取 `ResponseError` 的完整外部实现列表。

## 自定义错误响应示例

这是一个使用 [derive_more] crate 来声明错误枚举的 `ResponseError` 实现示例。

<CodeBlock example="errors" file="main.rs" section="response-error" />

`ResponseError` 对 `error_response()` 有一个默认实现，它将渲染一个 _500_（内部服务器错误），这就是上面 `index` 处理器执行时的结果。

重写 `error_response()` 以生成更有用的结果：

<CodeBlock example="errors" file="override_error.rs" section="override" />

## 错误助手

Actix Web 提供了一组错误助手函数，这些函数对于从其他错误生成特定的 HTTP 错误代码非常有用。在这里，我们使用 `map_err` 将不实现 `ResponseError` 特性的 `MyError` 转换为 _400_（错误请求）：

<CodeBlock example="errors" file="helpers.rs" section="helpers" />

请参阅 [actix-web 的 `error` 模块的 API 文档][actixerror] 以获取可用错误助手的完整列表。

## 错误日志记录

Actix 会在 `WARN` 日志级别记录所有错误。如果应用程序的日志级别设置为 `DEBUG` 并且启用了 `RUST_BACKTRACE`，则还会记录回溯。这些可以通过环境变量进行配置：

```shell-session
$ RUST_BACKTRACE=1 RUST_LOG=actix_web=debug cargo run
```

`Error` 类型使用原因的错误回溯（如果可用）。如果底层故障不提供回溯，则会构建一个新的回溯，指向发生转换的点（而不是错误的起源）。

## 错误处理的推荐实践

将应用程序产生的错误分为两大类可能是有用的：那些面向用户的错误和那些不是面向用户的错误。

前者的一个例子是，我可能会使用 failure 来指定一个 `UserError` 枚举，该枚举封装了一个 `ValidationError`，以便在用户发送错误输入时返回：

<CodeBlock example="errors" file="recommend_one.rs" section="recommend-one" />

这将完全按预期行为，因为使用 `display` 定义的错误消息是明确为了用户阅读而编写的。

然而，并不是所有错误都适合返回错误消息——在服务器环境中发生的许多故障，我们可能希望将具体细节隐藏起来。例如，如果数据库宕机并且客户端库开始产生连接超时错误，或者如果 HTML 模板格式不正确并在渲染时出错。在这些情况下，最好将错误映射到适合用户消费的通用错误。

这是一个将内部错误映射到带有自定义消息的面向用户的 `InternalError` 的示例：

<CodeBlock example="errors" file="recommend_two.rs" section="recommend-two" />

通过将错误分为面向用户的和非面向用户的，我们可以确保不会意外地向用户暴露应用程序内部抛出的错误，这些错误本不应该被用户看到。

## 错误日志记录

这是一个使用 `middleware::Logger` 的基本示例，它依赖于 `env_logger` 和 `log`：

```toml
[dependencies]
env_logger = "0.8"
log = "0.4"
```

<CodeBlock example="errors" file="logging.rs" section="logging" />

[actixerror]: https://docs.rs/actix-web/4/actix_web/error/struct.Error.html
[errorhelpers]: https://docs.rs/actix-web/4/actix_web/error/trait.ResponseError.html
[derive_more]: https://crates.io/crates/derive_more
[responseerror]: https://docs.rs/actix-web/4/actix_web/error/trait.ResponseError.html
[responseerrorimpls]: https://docs.rs/actix-web/4/actix_web/error/trait.ResponseError.html#foreign-impls
[stderror]: https://doc.rust-lang.org/std/error/trait.Error.html
[status_code]: https://docs.rs/actix-web/4/actix_web/http/struct.StatusCode.html

---
title: 入门指南
---

import RenderCodeBlock from '@theme/CodeBlock';
import CodeBlock from "@site/src/components/code_block";
import vars from "@site/vars";

## 安装 Rust

如果你还没有安装 Rust，我们推荐你使用 `rustup` 来管理你的 Rust 安装。 [官方 Rust 指南][rustguide] 有一个很棒的入门部分。

<p>
Actix Web 目前支持的最低 Rust 版本（MSRV）是 { vars.rustVersion }。运行 <code>rustup update</code> 将确保你拥有最新和最好的 Rust 版本。因此，本指南假设你运行的是 Rust { vars.rustVersion } 或更高版本。
</p>

## 你好，世界！

首先创建一个新的基于二进制的 Cargo 项目并切换到新目录：

```bash
cargo new hello-world
cd hello-world
```

通过在 `Cargo.toml` 文件中添加以下内容，将 `actix-web` 添加为项目的依赖项。

<!-- DEPENDENCY -->

<RenderCodeBlock className="language-toml">
{`[dependencies]
actix-web = "${vars.actixWebMajorVersion}"`}
</RenderCodeBlock>

请求处理程序使用接受零个或多个参数的异步函数。这些参数可以从请求中提取（参见 `FromRequest` 特性），并返回一个可以转换为 `HttpResponse` 的类型（参见 `Responder` 特性）：

将 `src/main.rs` 的内容替换为以下内容：

<CodeBlock example="getting-started" section="handlers" />

注意，一些处理程序直接使用内置宏附加了路由信息。这些宏允许你指定处理程序应该响应的方法和路径。你将在下面看到如何注册 `manual_hello`（即不使用路由宏的路由）。

接下来，创建一个 `App` 实例并注册请求处理程序。使用 `App::service` 为使用路由宏的处理程序注册，使用 `App::route` 为手动路由的处理程序注册，声明路径和方法。最后，应用程序在 `HttpServer` 内启动，它将使用你的 `App` 作为“应用程序工厂”来处理传入的请求。

进一步将以下 `main` 函数附加到 `src/main.rs`：

<CodeBlock example="getting-started" section="main" />

就是这样！使用 `cargo run` 编译并运行程序。 `#[actix_web::main]` 宏在 actix 运行时内执行异步主函数。现在你可以访问 `http://127.0.0.1:8080/` 或你定义的其他路由来查看结果。

<!-- LINKS -->

[rustguide]: https://doc.rust-lang.org/book/ch01-01-installation.html
[actix-web-codegen]: https://docs.rs/actix-web-codegen/

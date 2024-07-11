---
title: 什么是 Actix Web
---

import vars from "@site/vars";

# Actix Web 是一个 Crates 生态系统的一部分

很久以前，Actix Web 是建立在 `actix` actor 框架之上的。现在，Actix Web 与 actor 框架基本无关，而是使用不同的系统构建。尽管 `actix` 仍在维护，但随着 futures 和 async/await 生态系统的成熟，它作为通用工具的实用性正在减弱。目前，只有在 WebSocket 端点中才需要使用 `actix`。

我们称 Actix Web 为一个强大且务实的框架。实际上，它是一个带有一些独特之处的微框架。如果你已经是一个 Rust 程序员，你可能会很快适应它，但即使你来自其他编程语言，你也应该能很快上手 Actix Web。

<!-- TODO -->
<!-- actix-extras -->

使用 Actix Web 开发的应用程序将暴露一个包含在本地可执行文件中的 HTTP 服务器。你可以将其放在另一个 HTTP 服务器（如 nginx）后面，或者直接提供服务。即使在完全没有其他 HTTP 服务器的情况下，Actix Web 也足够强大，可以提供 HTTP/1 和 HTTP/2 支持以及 TLS（HTTPS）。这使得它在构建准备投入生产的小型服务时非常有用。

<p>
最重要的是：Actix Web 运行在 Rust { vars.rustVersion } 或更高版本上，并且与稳定版本兼容。
</p>

<!-- TODO -->
<!-- 它是建立在出色的 [Tokio][tokio] 异步 I/O 系统之上的 -->

<!-- 链接 -->

[tokio]: https://tokio.rs

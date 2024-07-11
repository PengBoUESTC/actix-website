---
title: HTTP/2
---

import RenderCodeBlock from '@theme/CodeBlock';
import CodeBlock from '@site/src/components/code_block';

`actix-web` 会在可能的情况下自动升级连接到 _HTTP/2_。

# 协商

<!-- TODO: 使用 rustls 示例 -->

当启用了 `rustls` 或 `openssl` 功能时，`HttpServer` 分别提供了 [`bind_rustls()`][bindrustls] 方法和 [`bind_openssl()`][bindopenssl] 方法。

<!-- 依赖 -->

<CodeBlock example="http2" file="manifest" section="deps" language="toml"></CodeBlock>

<CodeBlock example="http2" file="main.rs" section="main" />

[RFC 7540 §3.2][rfcsection32] 中描述的 HTTP/2 升级不被支持。对于明文和 TLS 连接，支持在已有知识的情况下启动 HTTP/2（[RFC 7540 §3.4][rfcsection34]）（使用较低级别的 `actix-http` 服务构建器时）。

> 查看 [TLS 示例][examples] 以获取具体示例。

[rfcsection32]: https://httpwg.org/specs/rfc7540.html#rfc.section.3.2
[rfcsection34]: https://httpwg.org/specs/rfc7540.html#rfc.section.3.4
[bindrustls]: https://docs.rs/actix-web/4/actix_web/struct.HttpServer.html#method.bind_rustls_0_22
[bindopenssl]: https://docs.rs/actix-web/4/actix_web/struct.HttpServer.html#method.bind_openssl
[tlsalpn]: https://tools.ietf.org/html/rfc7301
[examples]: https://github.com/actix/examples/tree/master/https-tls

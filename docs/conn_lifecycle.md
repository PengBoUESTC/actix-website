---
title: 连接生命周期
---

import MermaidDiagram from "@site/src/components/mermaid_diagram";
import connection_overview from '!!raw-loader!@site/static/img/diagrams/connection_overview.mmd';
import connection_accept from '!!raw-loader!@site/static/img/diagrams/connection_accept.mmd';
import connection_worker from '!!raw-loader!@site/static/img/diagrams/connection_worker.mmd';
import connection_request from '!!raw-loader!@site/static/img/diagrams/connection_request.mmd';

# 架构概述

在服务器开始监听所有套接字后，[`Accept`][accept] 和 [`Worker`][worker] 是负责处理传入客户端连接的两个主要循环。

一旦连接被接受，应用层协议处理将在从 [`Worker`][worker] 派生的特定协议 [`Dispatcher`][dispatcher] 循环中进行。

    请注意，以下图表仅概述了顺利的路径场景。

<MermaidDiagram value={connection_overview}  />

## 更详细的接受循环

<MermaidDiagram value={connection_accept}  />

大部分代码实现位于 [`actix-server`][server] crate 中的 [`Accept`][accept] 结构体。

## 更详细的工作循环

<MermaidDiagram value={connection_worker}  />

大部分代码实现位于 [`actix-server`][server] crate 中的 [`Worker`][worker] 结构体。

## 请求循环概述

<MermaidDiagram value={connection_request}  />

请求循环的大部分代码实现位于 [`actix-web`][web] 和 [`actix-http`][http] crates 中。

[server]: https://crates.io/crates/actix-server
[web]: https://crates.io/crates/actix-web
[http]: https://crates.io/crates/actix-http
[accept]: https://github.com/actix/actix-net/blob/master/actix-server/src/accept.rs
[worker]: https://github.com/actix/actix-net/blob/master/actix-server/src/worker.rs
[dispatcher]: https://github.com/actix/actix-web/blob/master/actix-http/src/h1/dispatcher.rs

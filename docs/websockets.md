---
title: Websockets
---

import CodeBlock from "@site/src/components/code_block";

# Websockets

Actix Web 支持使用 `actix-web-actors` crate 进行 WebSockets 通信。可以将请求的 `Payload` 转换为 [_ws::Message_][message] 的流，并使用 [_web::Payload_][payload] 处理实际消息，但使用 http actor 处理 websocket 通信更为简单。

以下是一个简单的 websocket 回声服务器示例：

<CodeBlock example="websockets" file="main.rs" section="websockets" />

> 一个简单的 websocket 回声服务器示例可以在 [examples 目录][examples] 中找到。

> 一个可以通过 websocket 或 TCP 连接进行聊天的聊天服务器示例可以在 [websocket-chat 目录][chat] 中找到。

[message]: https://docs.rs/actix-web-actors/2/actix_web_actors/ws/enum.Message.html
[payload]: https://docs.rs/actix-web/4/actix_web/web/struct.Payload.html
[examples]: https://github.com/actix/examples/tree/master/websockets
[chat]: https://github.com/actix/examples/tree/master/websockets/chat

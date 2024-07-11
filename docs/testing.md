---
title: 测试
---

import CodeBlock from "@site/src/components/code_block";

# 测试

每个应用程序都应该经过充分的测试。Actix Web 提供了工具来对你的应用程序进行集成测试，以及用于自定义提取器和中间件的单元测试工具。

Actix Web 提供了一种请求构建器类型。[_TestRequest_][testrequest] 实现了类似构建器的模式。你可以使用 `to_http_request()` 生成一个 `HttpRequest` 实例，并使用它调用你的处理程序或提取器。另请参见

## 应用程序的集成测试

有几种方法可以测试你的应用程序。Actix Web 可以用于在真实的 HTTP 服务器中运行具有特定处理程序的应用程序。

`TestRequest::get()`、`TestRequest::post()` 和其他方法可以用于向测试服务器发送请求。

要创建一个用于测试的 `Service`，请使用 `test::init_service` 方法，该方法接受一个常规的 `App` 构建器。

> 查看 [API 文档][actixdocs] 以获取更多信息。

<CodeBlock example="testing" file="integration_one.rs" section="integration-one" />

如果你需要更复杂的应用程序配置，测试应该与创建正常的应用程序非常相似。例如，你可能需要初始化应用程序状态。使用 `data` 方法创建一个 `App` 并附加状态，就像你在正常应用程序中一样。

<CodeBlock example="testing" file="integration_two.rs" section="integration-two" />

## 流响应测试

如果你需要测试流生成，只需调用 [`into_parts()`][resintoparts] 并将生成的主体转换为一个 future 并执行它，例如在测试 [_服务器发送事件_][serversentevents] 时。

<CodeBlock example="testing" file="stream_response.rs" section="stream-response" />

## 提取器的单元测试

单元测试对应用程序的价值相当有限，但在开发提取器、中间件和响应器时可能很有用。鉴于此，如果你想对自定义 `Responder` 进行断言，可以直接调用**独立定义的处理函数，而不使用路由宏**（如 `#[get("/")]`）。

<CodeBlock example="testing" file="main.rs" section="unit-tests" />

[serversentevents]: https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events
[resintoparts]: https://docs.rs/actix-web/4/actix_web/struct.HttpResponse.html#method.into_parts
[actixdocs]: https://docs.rs/actix-web/4/actix_web/test/index.html
[testrequest]: https://docs.rs/actix-web/4/actix_web/test/struct.TestRequest.html

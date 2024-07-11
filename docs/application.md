---
title: 应用程序
---

import CodeBlock from "@site/src/components/code_block";

# 编写应用程序

`actix-web` 提供了多种原语来使用 Rust 构建 Web 服务器和应用程序。它提供了路由、中间件、请求的预处理、响应的后处理等功能。

所有的 `actix-web` 服务器都是围绕 [`App`][app] 实例构建的。它用于注册资源和中间件的路由。它还存储在同一作用域内所有处理程序共享的应用程序状态。

应用程序的 [`scope`][scope] 充当所有路由的命名空间，即特定应用程序作用域的所有路由具有相同的 URL 路径前缀。应用程序前缀总是包含一个前导的 "/" 斜杠。如果提供的前缀不包含前导斜杠，则会自动插入。前缀应由值路径段组成。

> 对于作用域为 `/app` 的应用程序，任何带有路径 `/app`、`/app/` 或 `/app/test` 的请求都会匹配；然而，路径 `/application` 不会匹配。

<CodeBlock example="application" file="app.rs" section="setup" />

在这个例子中，创建了一个带有 `/app` 前缀和 `index.html` 资源的应用程序。该资源可以通过 `/app/index.html` URL 访问。

> 更多信息，请查看 [URL Dispatch][usingappprefix] 部分。

## 状态

应用程序状态在同一作用域内的所有路由和资源中共享。状态可以通过 [`web::Data<T>`][data] 提取器访问，其中 `T` 是状态的类型。状态也可以在中间件中访问。

让我们编写一个简单的应用程序并将应用程序名称存储在状态中：

<CodeBlock example="application" file="state.rs" section="setup" />

接下来，在初始化 App 时传入状态并启动应用程序：

<CodeBlock example="application" file="state.rs" section="start_app" />

可以在应用程序中注册任意数量的状态类型。

## 共享可变状态

`HttpServer` 接受一个应用程序工厂而不是一个应用程序实例。`HttpServer` 为每个线程构建一个应用程序实例。因此，应用程序数据必须多次构建。如果你想在不同线程之间共享数据，应该使用一个可共享的对象，例如 `Send` + `Sync`。

在内部，[`web::Data`][data] 使用 `Arc`。因此，为了避免创建两个 `Arc`，我们应该在注册之前创建我们的数据，使用 [`App::app_data()`][appdata]。

在以下示例中，我们将编写一个具有可变共享状态的应用程序。首先，我们定义我们的状态并创建我们的处理程序：

<CodeBlock example="application" file="mutable_state.rs" section="setup_mutable" />

并在 `App` 中注册数据：

<CodeBlock example="application" file="mutable_state.rs" section="make_app_mutable" />

关键要点：

- 在传递给 `HttpServer::new` 的闭包内初始化的状态是工作线程本地的，如果修改可能会变得不同步。
- 要实现全局共享状态，必须在传递给 `HttpServer::new` 的闭包外部创建，并移动/克隆进去。

## 使用应用程序作用域来组合应用程序

[`web::scope()`][webscope] 方法允许设置资源组前缀。此作用域表示一个资源前缀，该前缀将被预先添加到资源配置添加的所有资源模式中。这可以用于帮助在与原作者预期不同的位置挂载一组路由，同时仍保持相同的资源名称。

例如：

<CodeBlock example="application" file="scope.rs" section="scope" />

在上面的示例中，`show_users` 路由将具有 `/users/show` 的有效路由模式，而不是 `/show`，因为应用程序的作用域参数将被预先添加到模式中。然后，路由仅在 URL 路径为 `/users/show` 时匹配，并且当使用路由名称 `show_users` 调用 [`HttpRequest.url_for()`][urlfor] 函数时，它将生成具有相同路径的 URL。

## 应用程序守卫和虚拟主机

你可以将守卫视为一个简单的函数，它接受一个请求对象引用并返回 true 或 false。正式地，守卫是任何实现 [`Guard`][guardtrait] 特性的对象。Actix Web 提供了几种守卫。你可以查看 API 文档的 [functions section][guardfuncs]。

提供的守卫之一是 [`Host`][guardhost]。它可以用作基于请求头信息的过滤器。

<CodeBlock example="application" file="vh.rs" section="vh" />

## 配置

为了简化和重用，[`App`][appconfig] 和 [`web::Scope`][webscopeconfig] 都提供了 `configure` 方法。此函数对于将部分配置移动到不同的模块甚至库中非常有用。例如，某些资源的配置可以移动到不同的模块中。

<CodeBlock example="application" file="config.rs" section="config" />

上述示例的结果将是：

```
/         -> "/"
/app      -> "app"
/api/test -> "test"
```

每个 [`ServiceConfig`][serviceconfig] 可以有自己的 `data`、`routes` 和 `services`。

<!-- 链接 -->

[usingappprefix]: /docs/url-dispatch#using-an-application-prefix-to-compose-applications
[stateexample]: https://github.com/actix/examples/blob/master/basics/state/src/main.rs
[guardtrait]: https://docs.rs/actix-web/4/actix_web/guard/trait.Guard.html
[guardfuncs]: https://docs.rs/actix-web/4/actix_web/guard/index.html#functions
[guardhost]: https://docs.rs/actix-web/4/actix_web/guard/fn.Host.html
[data]: https://docs.rs/actix-web/4/actix_web/web/struct.Data.html
[app]: https://docs.rs/actix-web/4/actix_web/struct.App.html
[appconfig]: https://docs.rs/actix-web/4/actix_web/struct.App.html#method.configure
[appdata]: https://docs.rs/actix-web/4/actix_web/struct.App.html#method.app_data
[scope]: https://docs.rs/actix-web/4/actix_web/struct.Scope.html
[webscopeconfig]: https://docs.rs/actix-web/4/actix_web/struct.Scope.html#method.configure
[webscope]: https://docs.rs/actix-web/4/actix_web/web/fn.scope.html
[urlfor]: https://docs.rs/actix-web/4/actix_web/struct.HttpRequest.html#method.url_for
[serviceconfig]: https://docs.rs/actix-web/4/actix_web/web/struct.ServiceConfig.html

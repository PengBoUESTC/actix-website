---
title: 中间件
---

import CodeBlock from "@site/src/components/code_block";

# 中间件

Actix Web 的中间件系统允许我们在请求/响应处理过程中添加额外的行为。中间件可以挂钩到传入的请求过程中，使我们能够修改请求以及提前返回响应以停止请求处理。

中间件也可以挂钩到响应处理过程中。

通常，中间件涉及以下操作：

- 预处理请求
- 后处理响应
- 修改应用程序状态
- 访问外部服务（如 Redis、日志记录、会话）

中间件为每个 `App`、`scope` 或 `Resource` 注册，并按照注册的相反顺序执行。一般来说，中间件是一种实现了 [_Service trait_][servicetrait] 和 [_Transform trait_][transformtrait] 的类型。traits 中的每个方法都有默认实现。每个方法可以立即返回结果或返回一个 _future_ 对象。

以下演示了创建一个简单的中间件：

<CodeBlock example="middleware" file="main.rs" section="simple" />

或者，对于简单的用例，你可以使用 [_wrap_fn_][wrap_fn] 来创建小型的临时中间件：

<CodeBlock example="middleware" file="wrap_fn.rs" section="wrap-fn" />

> Actix Web 提供了几个有用的中间件，例如 _logging_、_user sessions_、_compress_ 等。

**警告：如果你多次使用 `wrap()` 或 `wrap_fn()`，最后一次出现的将首先执行。**

## 日志记录

日志记录作为中间件实现。通常将日志记录中间件注册为应用程序的第一个中间件。每个应用程序都必须注册日志记录中间件。

`Logger` 中间件使用标准的 log crate 来记录信息。你应该为 _actix_web_ 包启用日志记录以查看访问日志（[env_logger][envlogger] 或类似的）。

### 用法

使用指定的 `format` 创建 `Logger` 中间件。可以使用 `default` 方法创建默认的 `Logger`，它使用默认格式：

```ignore
  %a %t "%r" %s %b "%{Referer}i" "%{User-Agent}i" %T
```

<CodeBlock example="middleware" file="logger.rs" section="logger" />

以下是默认日志格式的示例：

```log
INFO:actix_web::middleware::logger: 127.0.0.1:59934 [02/Dec/2017:00:21:43 -0800] "GET / HTTP/1.1" 302 0 "-" "curl/7.54.0" 0.000397
INFO:actix_web::middleware::logger: 127.0.0.1:59947 [02/Dec/2017:00:22:40 -0800] "GET /index.html HTTP/1.1" 200 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:57.0) Gecko/20100101 Firefox/57.0" 0.000646
```

### 格式

- `%%` 百分号
- `%a` 远程 IP 地址（如果使用反向代理，则为代理的 IP 地址）
- `%t` 请求开始处理的时间
- `%P` 服务请求的子进程的进程 ID
- `%r` 请求的第一行
- `%s` 响应状态码
- `%b` 响应的大小（字节），包括 HTTP 头
- `%T` 服务请求所花费的时间，以秒为单位，浮点数格式为 .06f
- `%D` 服务请求所花费的时间，以毫秒为单位
- `%{FOO}i` request.headers['FOO']
- `%{FOO}o` response.headers['FOO']
- `%{FOO}e` os.environ['FOO']

## 默认头

要设置默认的响应头，可以使用 `DefaultHeaders` 中间件。如果响应头已经包含指定的头，则 _DefaultHeaders_ 中间件不会设置该头。

<CodeBlock example="middleware" file="default_headers.rs" section="default-headers" />

## 用户会话

Actix Web 提供了一个通用的会话管理解决方案。[**actix-session**][actixsession] 中间件可以使用多种后端类型来存储会话数据。

> 默认情况下，仅实现了 cookie 会话后端。可以添加其他后端实现。

[**CookieSession**][cookiesession] 使用 cookie 作为会话存储。`CookieSessionBackend` 创建的会话限制为存储少于 4000 字节的数据，因为负载必须适合一个 cookie。如果会话包含超过 4000 字节的数据，则会生成内部服务器错误。

cookie 可能具有 _signed_ 或 _private_ 的安全策略。每种策略都有相应的 `CookieSession` 构造函数。

_signed_ cookie 可以被查看但不能被客户端修改。_private_ cookie 既不能被查看也不能被客户端修改。

构造函数接受一个密钥作为参数。这是 cookie 会话的私钥——当此值更改时，所有会话数据都会丢失。

通常，你会创建一个 `SessionStorage` 中间件并使用特定的后端实现（如 `CookieSession`）进行初始化。要访问会话数据，必须使用 [`Session`][requestsession] 提取器。此方法返回一个 [_Session_][sessionobj] 对象，使我们能够获取或设置会话数据。

> `actix_session::storage::CookieSessionStore` 在 crate 特性 "cookie-session" 上可用。

<CodeBlock example="middleware" file="user_sessions.rs" section="user-session" />

## 错误处理程序

`ErrorHandlers` 中间件允许我们为响应提供自定义处理程序。

你可以使用 `ErrorHandlers::handler()` 方法为特定状态码注册自定义错误处理程序。你可以修改现有的响应或创建一个全新的响应。错误处理程序可以立即返回响应或返回一个解析为响应的 future。

<CodeBlock example="middleware" file="errorhandler.rs" section="error-handler" />

[sessionobj]: https://docs.rs/actix-session/0.7/actix_session/struct.Session.html
[requestsession]: https://docs.rs/actix-session/0.7/actix_session/struct.Session.html
[cookiesession]: https://docs.rs/actix-session/0.7/actix_session/storage/struct.CookieSessionStore.html
[actixsession]: https://docs.rs/actix-session/0.7/actix_session/
[envlogger]: https://docs.rs/env_logger/*/env_logger/
[servicetrait]: https://docs.rs/actix-web/4/actix_web/dev/trait.Service.html
[transformtrait]: https://docs.rs/actix-web/4/actix_web/dev/trait.Transform.html
[wrap_fn]: https://docs.rs/actix-web/4/actix_web/struct.App.html#method.wrap_fn

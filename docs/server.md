---
title: 服务器
---

import RenderCodeBlock from '@theme/CodeBlock';
import CodeBlock from '@site/src/components/code_block';
import vars from "@site/vars";

# HTTP 服务器

[**HttpServer**][httpserverstruct] 类型负责处理 HTTP 请求。

`HttpServer` 接受一个应用程序工厂作为参数，该工厂必须具有 `Send` + `Sync` 边界。更多内容请参见 _多线程_ 部分。

要启动 Web 服务器，首先必须绑定到网络套接字。使用 [`HttpServer::bind()`][bindmethod] 绑定到一个套接字地址元组或字符串，例如 `("127.0.0.1", 8080)` 或 `"0.0.0.0:8080"`。如果套接字被其他应用程序使用，这将失败。

在 `bind` 成功后，使用 [`HttpServer::run()`][httpserver_run] 返回一个 [`Server`][server] 实例。`Server` 必须 `await` 或 `spawn` 才能开始处理请求，并将运行直到收到关闭信号（例如，默认情况下是 `ctrl-c`；[在这里阅读更多](#graceful-shutdown)）。

<CodeBlock example="server" section="main" />

## 多线程

`HttpServer` 自动启动多个 HTTP _工作线程_，默认情况下，这个数量等于系统中的物理 CPU 数量。可以使用 [`HttpServer::workers()`][workers] 方法覆盖此数量。

<CodeBlock example="server" file="workers.rs" section="workers" />

一旦创建了工作线程，它们每个都会收到一个单独的 _应用程序_ 实例来处理请求。应用程序状态在线程之间不共享，处理程序可以自由地操作它们的状态副本而无需担心并发问题。

应用程序状态不需要是 `Send` 或 `Sync`，但应用程序工厂必须是 `Send` + `Sync`。

要在工作线程之间共享状态，请使用 `Arc`/`Data`。一旦引入共享和同步，需要特别注意。在许多情况下，由于锁定共享状态进行修改，性能成本会无意中增加。

在某些情况下，可以使用更高效的锁定策略来缓解这些成本，例如使用 [读/写锁](https://doc.rust-lang.org/std/sync/struct.RwLock.html) 而不是 [互斥锁](https://doc.rust-lang.org/std/sync/struct.Mutex.html) 来实现非独占锁定，但最具性能的实现通常是完全不发生锁定的实现。

由于每个工作线程按顺序处理其请求，阻塞当前线程的处理程序将导致当前工作线程停止处理新请求：

```rust
fn my_handler() -> impl Responder {
    std::thread::sleep(Duration::from_secs(5)); // <-- 不好的做法！会导致当前工作线程挂起！
    "response"
}
```

因此，任何长时间的、非 CPU 绑定的操作（例如 I/O、数据库操作等）都应表示为 futures 或异步函数。异步处理程序由工作线程并发执行，因此不会阻塞执行：

```rust
async fn my_handler() -> impl Responder {
    tokio::time::sleep(Duration::from_secs(5)).await; // <-- 可以。工作线程将在此处处理其他请求
    "response"
}
```

同样的限制也适用于提取器。当处理程序函数接收一个实现了 `FromRequest` 的参数，并且该实现阻塞当前线程时，工作线程在运行处理程序时将被阻塞。由于这个原因，在实现提取器时需要特别注意，并且在需要时也应异步实现。

## TLS / HTTPS

Actix Web 支持两种开箱即用的 TLS 实现：`rustls` 和 `openssl`。

`rustls` crate 特性用于 `rustls` 集成，`openssl` 用于 `openssl` 集成。

<!-- 依赖 -->

<RenderCodeBlock className="language-toml">
{`[dependencies]
actix-web = { version = "${vars.actixWebMajorVersion}", features = ["openssl"] }
openssl = { version = "0.10" }
`}
</RenderCodeBlock>

<CodeBlock example="server" file="ssl.rs" section="ssl" />

要创建 key.pem 和 cert.pem，请使用以下命令。**填写你自己的主题**

```shell-session
$ openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem \
    -days 365 -sha256 -subj "/C=CN/ST=Fujian/L=Xiamen/O=TVlinux/OU=Org/CN=muro.lxd"
```

要移除密码，然后将 nopass.pem 复制到 key.pem

```shell-session
$ openssl rsa -in key.pem -out nopass.pem
```

## 保持连接

Actix Web 保持连接打开以等待后续请求。

> _保持连接_ 行为由服务器设置定义。

- `Duration::from_secs(75)` 或 `KeepAlive::Timeout(75)`：启用 75 秒的保持连接计时器。
- `KeepAlive::Os`：使用操作系统保持连接。
- `None` 或 `KeepAlive::Disabled`：禁用保持连接。

<CodeBlock example="server" file="keep_alive.rs" section="keep-alive" />

如果选择了上述第一个选项，则在响应未明确禁止的情况下，保持连接将为 HTTP/1.1 请求启用，例如，通过设置 [连接类型][httpconnectiontype] 为 `Close` 或 `Upgrade`。可以使用 [`HttpResponseBuilder` 上的 `force_close()` 方法](https://docs.rs/actix-web/4/actix_web/struct.HttpResponseBuilder.html#method.force_close) 强制关闭连接。

> 对于 HTTP/1.0，保持连接是 **关闭** 的，对于 HTTP/1.1 和 HTTP/2.0，保持连接是 **开启** 的。

<CodeBlock example="server" file="keep_alive_tp.rs" section="example" />

## 优雅关闭

`HttpServer` 支持优雅关闭。在收到停止信号后，工作线程有特定的时间来完成请求处理。超时后仍然存活的工作线程将被强制删除。默认情况下，关闭超时设置为 30 秒。可以使用 [`HttpServer::shutdown_timeout()`][shutdowntimeout] 方法更改此参数。

`HttpServer` 处理多个操作系统信号。_CTRL-C_ 在所有操作系统上可用，其他信号在 Unix 系统上可用。

- _SIGINT_ - 强制关闭工作线程
- _SIGTERM_ - 优雅关闭工作线程
- _SIGQUIT_ - 强制关闭工作线程

> 可以使用 [`HttpServer::disable_signals()`][disablesignals] 方法禁用信号处理。

[server]: https://docs.rs/actix-web/4/actix_web/dev/struct.Server.html
[httpserverstruct]: https://docs.rs/actix-web/4/actix_web/struct.HttpServer.html
[bindmethod]: https://docs.rs/actix-web/4/actix_web/struct.HttpServer.html#method.bind
[httpserver_run]: https://docs.rs/actix-web/4/actix_web/struct.HttpServer.html#method.run
[bindopensslmethod]: https://docs.rs/actix-web/4/actix_web/struct.HttpServer.html#method.bind_openssl
[bindrusttls]: https://docs.rs/actix-web/4/actix_web/struct.HttpServer.html#method.bind_rustls
[workers]: https://docs.rs/actix-web/4/actix_web/struct.HttpServer.html#method.workers
[tlsalpn]: https://tools.ietf.org/html/rfc7301
[exampleopenssl]: https://github.com/actix/examples/tree/master/security/openssl
[shutdowntimeout]: https://docs.rs/actix-web/4/actix_web/struct.HttpServer.html#method.shutdown_timeout
[disablesignals]: https://docs.rs/actix-web/4/actix_web/struct.HttpServer.html#method.disable_signals
[httpconnectiontype]: https://docs.rs/actix-web/4/actix_web/http/enum.ConnectionType.html

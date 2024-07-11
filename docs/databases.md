---
title: 数据库
---

import CodeBlock from "@site/src/components/code_block";

# 异步选项

我们有几个展示使用异步数据库适配器的示例项目：

- [Postgres](https://github.com/actix/examples/tree/master/databases/postgres)
- [SQLite](https://github.com/actix/examples/tree/master/databases/sqlite)
- [MongoDB](https://github.com/actix/examples/tree/master/databases/mongodb)

# Diesel

当前版本的 Diesel (v1/v2) 不支持异步操作，因此使用 [`web::block`][web-block] 函数将数据库操作卸载到 Actix 运行时线程池中是很重要的。

你可以创建与应用程序在数据库上执行的所有操作相对应的动作函数。

<CodeBlock example="databases" file="main.rs" section="handler" />

现在你应该使用诸如 `r2d2` 之类的 crate 来设置数据库连接池，这样可以使许多数据库连接可供你的应用程序使用。这意味着多个处理程序可以同时操作数据库，并且仍然可以接受新连接。简单地说，就是在你的应用程序状态中使用连接池。（在这种情况下，不使用状态包装结构是有益的，因为连接池为你处理了共享访问。）

<CodeBlock example="databases" file="main.rs" section="main" />

现在，在请求处理程序中，使用 `Data<T>` 提取器从应用程序状态中获取连接池并从中获取连接。这提供了一个可以传递到 [`web::block`][web-block] 闭包中的拥有的数据库连接。然后只需使用必要的参数调用动作函数并 `.await` 结果。

这个示例还在使用 `?` 操作符之前将错误映射到 `HttpResponse`，但如果你的返回错误类型实现了 [`ResponseError`][response-error]，这不是必须的。

<CodeBlock example="databases" file="main.rs" section="index" />

就是这样！查看完整示例 [这里](https://github.com/actix/examples/tree/master/databases/diesel)。

[web-block]: https://docs.rs/actix-web/4/actix_web/web/fn.block.html
[response-error]: https://docs.rs/actix-web/4/actix_web/error/trait.ResponseError.html

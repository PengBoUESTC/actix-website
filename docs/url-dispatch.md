---
title: URL 分发
---

import CodeBlock from "@site/src/components/code_block";

# URL 分发

URL 分发提供了一种简单的方法，通过简单的模式匹配语言将 URL 映射到处理程序代码。如果其中一个模式与请求相关的路径信息匹配，则会调用特定的处理程序对象。

> 请求处理程序是一个接受零个或多个参数的函数，这些参数可以从请求中提取（即 [_impl FromRequest_][implfromrequest]），并返回一个可以转换为 HttpResponse 的类型（即 [_impl Responder_][implresponder]）。更多信息请参见 [处理程序部分][handlersection]。

## 资源配置

资源配置是向应用程序添加新资源的行为。资源有一个名称，作为 URL 生成的标识符。名称还允许开发人员向现有资源添加路由。资源还有一个模式，用于匹配 _URL_ 的 _PATH_ 部分（即方案和端口之后的部分，例如 _`http://localhost:8080/foo/bar?q=value`_ 中的 _/foo/bar_）。它不匹配 _QUERY_ 部分（即 _`http://localhost:8080/foo/bar?q=value`_ 中的 _q=value_）。

[_App::route()_][approute] 方法提供了一种简单的注册路由的方法。此方法向应用程序路由表添加单个路由。此方法接受一个 _路径模式_、_HTTP 方法_ 和一个处理函数。可以多次调用 `route()` 方法用于相同的路径，在这种情况下，多个路由会为相同的资源路径注册。

<CodeBlock example="url-dispatch" section="main" />

虽然 _App::route()_ 提供了简单的注册路由的方法，但要访问完整的资源配置，必须使用不同的方法。[_App::service()_][appservice] 方法向应用程序路由表添加单个 [资源][webresource]。此方法接受一个 _路径模式_、守卫和一个或多个路由。

<CodeBlock example="url-dispatch" file="resource.rs" section="resource" />

如果资源不包含任何路由或没有任何匹配的路由，它将返回 _NOT FOUND_ HTTP 响应。

### 配置路由

资源包含一组路由。每个路由依次包含一组 `guards` 和一个处理程序。可以使用 `Resource::route()` 方法创建新路由，该方法返回对新 _Route_ 实例的引用。默认情况下，_route_ 不包含任何守卫，因此匹配所有请求，默认处理程序是 `HttpNotFound`。

应用程序根据资源注册和路由注册期间定义的路由条件路由传入的请求。资源按通过 `Resource::route()` 注册的顺序匹配其包含的所有路由。

> 一个 _Route_ 可以包含任意数量的 _guards_，但只能有一个处理程序。

<CodeBlock example="url-dispatch" file="cfg.rs" section="cfg" />

在此示例中，如果请求包含 `Content-Type` 头，其值为 _text/plain_，并且路径等于 `/path`，则返回 `HttpResponse::Ok()`。

如果资源无法匹配任何路由，则返回 "NOT FOUND" 响应。

[_ResourceHandler::route()_][resourcehandler] 返回一个 [_Route_][route] 对象。路由可以使用类似构建器的模式进行配置。以下配置方法可用：

- [_Route::guard()_][routeguard] 注册一个新的守卫。每个路由可以注册任意数量的守卫。
- [_Route::method()_][routemethod] 注册一个方法守卫。每个路由可以注册任意数量的守卫。
- [_Route::to()_][routeto] 为此路由注册一个异步处理函数。只能注册一个处理程序。通常处理程序注册是最后的配置操作。

## 路由匹配

路由配置的主要目的是将请求的 `path` 与 URL 路径模式匹配（或不匹配）。`path` 表示请求的 URL 的路径部分。

_actix-web_ 的实现方式非常简单。当请求进入系统时，对于系统中存在的每个资源配置声明，actix 会检查请求的路径是否与声明的模式匹配。此检查按通过 `App::service()` 方法声明路由的顺序进行。如果找不到资源，则使用 _默认资源_ 作为匹配资源。

当声明路由配置时，它可能包含路由守卫参数。与路由声明关联的所有路由守卫必须为 `true`，才能在检查期间将路由配置用于给定请求。如果在提供给路由配置的一组路由守卫参数中有任何一个在检查期间返回 `false`，则跳过该路由，并通过有序的路由集继续路由匹配。

如果任何路由匹配，则路由匹配过程停止，并调用与该路由关联的处理程序。如果在所有路由模式都耗尽后没有路由匹配，则返回 _NOT FOUND_ 响应。

## 资源模式语法

actix 使用的模式匹配语言的语法非常简单。

路由配置中使用的模式可以以斜杠字符开头。如果模式不以斜杠字符开头，则在匹配时会在其前面加上一个隐式斜杠。例如，以下模式是等效的：

```
{foo}/bar/baz
```

和：

```
/{foo}/bar/baz
```

_变量部分_（替换标记）以 _\{identifier}_ 的形式指定，这意味着“接受直到下一个斜杠字符的任何字符，并将其用作 `HttpRequest.match_info()` 对象中的名称”。

模式中的替换标记匹配正则表达式 `[^{}/]+`。

`match_info` 是 `Params` 对象，表示根据路由模式从 _URL_ 中提取的动态部分。它作为 _request.match_info_ 可用。例如，以下模式定义了一个文字段（foo）和两个替换标记（baz 和 bar）：

```
foo/{baz}/{bar}
```

上述模式将匹配这些 URL，生成以下匹配信息：

```
foo/1/2        -> Params {'baz': '1', 'bar': '2'}
foo/abc/def    -> Params {'baz': 'abc', 'bar': 'def'}
```

但是，它不会匹配以下模式：

```
foo/1/2/        -> 不匹配（尾随斜杠）
bar/abc/def     -> 第一个段文字不匹配
```

段中的替换标记的匹配仅在模式中段的第一个非字母数字字符之前进行。例如，如果使用以下路由模式：

```
foo/{name}.html
```

文字路径 `/foo/biz.html` 将匹配上述路由模式，匹配结果将是 `Params {'name': 'biz'}`。但是，文字路径 `/foo/biz` 将不匹配，因为它不包含 `{name}.html` 表示的段末尾的文字 `.html`（它仅包含 biz，而不是 biz.html）。

要捕获两个段，可以使用两个替换标记：

```
foo/{name}.{ext}
```

文字路径 `/foo/biz.html` 将匹配上述路由模式，匹配结果将是 `Params {'name': 'biz', 'ext': 'html'}`。这是因为在两个替换标记 `{name}` 和 `{ext}` 之间有一个文字部分 `.`（句点）。

替换标记可以选择性地指定一个正则表达式，该正则表达式将用于决定路径段是否应匹配标记。要指定替换标记应仅匹配由正则表达式定义的特定字符集，必须使用稍微扩展的替换标记语法。在大括号内，替换标记名称后必须跟一个冒号，然后直接跟正则表达式。与替换标记关联的默认正则表达式 `[^/]+` 匹配一个或多个不是斜杠的字符。例如，替换标记 `{foo}` 可以更详细地拼写为 `{foo:[^/]+}`。您可以将其更改为任意正则表达式以匹配任意字符序列，例如 `{foo:\d+}` 仅匹配数字。

段必须包含至少一个字符才能匹配段替换标记。例如，对于 URL `/abc/`：

- `/abc/{foo}` 将不匹配。
- `/{foo}/` 将匹配。

> **注意**：在匹配模式之前，路径将被 URL 解码并解码为有效的 Unicode 字符串，表示匹配路径段的值也将被 URL 解码。

例如，以下模式：

```
foo/{bar}
```

匹配以下 URL 时：

```
http://example.com/foo/La%20Pe%C3%B1a
```

匹配字典将如下所示（值已 URL 解码）：

```
Params {'bar': 'La Pe\xf1a'}
```

路径段中的文字字符串应表示提供给 actix 的路径的解码值。您不希望在模式中使用 URL 编码的值。例如，而不是这样：

```
/Foo%20Bar/{baz}
```

您将希望使用类似这样的内容：

```
/Foo Bar/{baz}
```

可以获得“尾部匹配”。为此，必须使用自定义正则表达式。

```
foo/{bar}/{tail:.*}
```

上述模式将匹配这些 URL，生成以下匹配信息：

```
foo/1/2/           -> Params {'bar': '1', 'tail': '2/'}
foo/abc/def/a/b/c  -> Params {'bar': 'abc', 'tail': 'def/a/b/c'}
```

## 路由作用域

作用域帮助您组织共享公共根路径的路由。您可以在作用域内嵌套作用域。

假设您要组织用于查看“用户”的端点路径。这些路径可能包括：

- /users
- /users/show
- /users/show/\{id}

这些路径的作用域布局如下所示

<CodeBlock example="url-dispatch" file="scope.rs" section="scope" />

_作用域_ 路径可以包含作为资源的可变路径段。与非作用域路径一致。

您可以从 `HttpRequest::match_info()` 获取可变路径段。[`Path` 提取器][pathextractor] 也能够提取作用域级别的可变段。

## 匹配信息

所有表示匹配路径段的值都可以在 [`HttpRequest::match_info`][matchinfo] 中找到。可以使用 [`Path::get()`][pathget] 检索特定值。

<CodeBlock example="url-dispatch" file="minfo.rs" section="minfo" />

对于路径 '/a/1/2/' 的此示例，值 v1 和 v2 将解析为 "1" 和 "2"。

### 路径信息提取器

Actix 提供了类型安全的路径信息提取功能。[_Path_][pathstruct] 提取信息，目标类型可以用几种不同的形式定义。最简单的方法是使用 `tuple` 类型。元组中的每个元素必须对应路径模式中的一个元素。例如，你可以将路径模式 `/{id}/{username}/` 与 `Path<(u32, String)>` 类型匹配，但 `Path<(String, String, String)>` 类型将始终失败。

<CodeBlock example="url-dispatch" file="path.rs" section="path" />

也可以将路径模式信息提取到结构体中。在这种情况下，该结构体必须实现 _serde_ 的 `Deserialize` 特性。

<CodeBlock example="url-dispatch" file="path2.rs" section="path" />

[_Query_][query] 提供了类似的功能，用于请求查询参数。

## 生成资源 URL

使用 [_HttpRequest.url_for()_][urlfor] 方法根据资源模式生成 URL。例如，如果你配置了一个名为 "foo" 的资源，并且模式为 "\{a}/\{b}/\{c}"，你可以这样做：

<CodeBlock example="url-dispatch" file="urls.rs" section="url" />

这将返回类似于 `http://example.com/test/1/2/3` 的字符串（至少如果当前协议和主机名暗示 `http://example.com`）。`url_for()` 方法返回 [_Url 对象_][urlobj]，因此你可以修改此 URL（添加查询参数、锚点等）。`url_for()` 只能为 _命名_ 资源调用，否则会返回错误。

## 外部资源

有效的 URL 资源可以注册为外部资源。它们仅用于 URL 生成目的，在请求时从不考虑匹配。

<CodeBlock example="url-dispatch" file="url_ext.rs" section="ext" />

## 路径规范化和重定向到带斜杠的路由

规范化意味着：

- 在路径末尾添加斜杠。
- 将多个斜杠替换为一个。

处理程序一旦找到正确解析的路径就会返回。如果所有规范化条件都启用，顺序是 1) 合并，2) 合并和追加，3) 追加。如果路径在至少一个条件下解析，它将重定向到新路径。

<CodeBlock example="url-dispatch" file="norm.rs" section="norm" />

在这个例子中，`//resource///` 将被重定向到 `/resource/`。

在这个例子中，路径规范化处理程序为所有方法注册，但你不应依赖此机制来重定向 _POST_ 请求。斜杠追加的 _Not Found_ 重定向会将 _POST_ 请求变为 GET，从而丢失原始请求中的任何 _POST_ 数据。

可以仅为 _GET_ 请求注册路径规范化：

<CodeBlock example="url-dispatch" file="norm2.rs" section="norm" />

### 使用应用程序前缀来组合应用程序

`web::scope()` 方法允许设置特定的应用程序范围。此范围表示一个资源前缀，将被添加到资源配置添加的所有资源模式之前。这可以帮助在不同位置挂载一组路由，同时保持相同的资源名称。

例如：

<CodeBlock example="url-dispatch" file="scope.rs" section="scope" />

在上面的例子中，_show_users_ 路由将具有 _/users/show_ 的有效路由模式，而不是 _/show_，因为应用程序的范围将被添加到模式之前。然后，路由将仅在 URL 路径为 _/users/show_ 时匹配，当使用路由名称 show_users 调用 `HttpRequest.url_for()` 函数时，它将生成具有相同路径的 URL。

## 自定义路由守卫

你可以将守卫视为一个简单的函数，它接受一个 _request_ 对象引用并返回 _true_ 或 _false_。正式来说，守卫是任何实现 [`Guard`][guardtrait] 特性的对象。Actix 提供了几个谓词，你可以查看 API 文档的 [functions section][guardfuncs]。

这是一个简单的守卫，检查请求是否包含特定的 _header_：

<CodeBlock example="url-dispatch" file="guard.rs" section="guard" />

在这个例子中，_index_ 处理程序只有在请求包含 _CONTENT-TYPE_ 头时才会被调用。

守卫不能访问或修改请求对象，但可以在 [request extensions][requestextensions] 中存储额外信息。

### 修改守卫值

你可以通过将任何谓词值包装在 `Not` 谓词中来反转其含义。例如，如果你想为除 "GET" 之外的所有方法返回 "METHOD NOT ALLOWED" 响应：

<CodeBlock example="url-dispatch" file="guard2.rs" section="guard2" />

`Any` 守卫接受一个守卫列表，如果任何提供的守卫匹配，则匹配。例如：

```rust
guard::Any(guard::Get()).or(guard::Post())
```

`All` 守卫接受一个守卫列表，如果所有提供的守卫都匹配，则匹配。例如：

```rust
guard::All(guard::Get()).and(guard::Header("content-type", "plain/text"))
```

## 更改默认的 Not Found 响应

如果路径模式在路由表中找不到，或者资源找不到匹配的路由，则使用默认资源。默认响应是 _NOT FOUND_。可以使用 `App::default_service()` 覆盖 _NOT FOUND_ 响应。此方法接受一个 _configuration function_，与使用 `App::service()` 方法进行正常资源配置相同。

<CodeBlock example="url-dispatch" file="dhandler.rs" section="default" />

[handlersection]: /docs/handlers/
[approute]: https://docs.rs/actix-web/4/actix_web/struct.App.html#method.route
[appservice]: https://docs.rs/actix-web/4/actix_web/struct.App.html?search=#method.service
[webresource]: https://docs.rs/actix-web/4/actix_web/struct.Resource.html
[resourcehandler]: https://docs.rs/actix-web/4/actix_web/struct.Resource.html#method.route
[route]: https://docs.rs/actix-web/4/actix_web/struct.Route.html
[routeguard]: https://docs.rs/actix-web/4/actix_web/struct.Route.html#method.guard
[routemethod]: https://docs.rs/actix-web/4/actix_web/struct.Route.html#method.method
[routeto]: https://docs.rs/actix-web/4/actix_web/struct.Route.html#method.to
[matchinfo]: https://docs.rs/actix-web/4/actix_web/struct.HttpRequest.html#method.match_info
[pathget]: https://docs.rs/actix-web/4/actix_web/dev/struct.Path.html#method.get
[pathstruct]: https://docs.rs/actix-web/4/actix_web/dev/struct.Path.html
[query]: https://docs.rs/actix-web/4/actix_web/web/struct.Query.html
[urlfor]: https://docs.rs/actix-web/4/actix_web/struct.HttpRequest.html#method.url_for
[urlobj]: https://docs.rs/url/1.7.2/url/struct.Url.html
[guardtrait]: https://docs.rs/actix-web/4/actix_web/guard/trait.Guard.html
[guardfuncs]: https://docs.rs/actix-web/4/actix_web/guard/index.html#functions
[requestextensions]: https://docs.rs/actix-web/4/actix_web/struct.HttpRequest.html#method.extensions
[implfromrequest]: https://docs.rs/actix-web/4/actix_web/trait.FromRequest.html
[implresponder]: https://docs.rs/actix-web/4/actix_web/trait.Responder.html
[pathextractor]: /docs/extractors

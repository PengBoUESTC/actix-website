---
title: HTTP 服务器初始化
---

import CodeBlock from "@site/src/components/code_block";
import MermaidDiagram from "@site/src/components/mermaid_diagram";
import http_server from '!!raw-loader!@site/static/img/diagrams/http_server.mmd';

# 架构概述

下面是 HttpServer 初始化的图示，对应以下代码

```rust
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::to(|| HttpResponse::Ok()))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

<MermaidDiagram value={http_server}  />

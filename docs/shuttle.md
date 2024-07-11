---
title: 在 Shuttle 上托管
---

import CodeBlock from '@site/src/components/code_block';

# 在 Shuttle 上托管

<img width="300" src="https://raw.githubusercontent.com/shuttle-hq/shuttle/master/assets/logo-rectangle-transparent.png" alt="Shuttle Logo"/>

> [**Shuttle**](https://www.shuttle.rs) 是一个原生支持 Rust 的云开发平台，让你可以免费部署你的 Rust 应用。

Shuttle 开箱即用地支持 Actix Web。按照以下步骤在 Shuttle 上托管你的 Web 服务：

1. 在 `Cargo.toml` 中添加 Shuttle 依赖：

<CodeBlock example="shuttle" file="manifest" section="shuttle-deps" language="toml" />

2. 添加 `#[shuttle_runtime::main]` 注解并更新 `main` 函数如下：

<CodeBlock example="shuttle" section="shuttle-hello-world" />

3. 安装 `cargo-shuttle`：

```sh
cargo install cargo-shuttle
```

4. 在 Shuttle 平台上创建你的项目：

```sh
cargo shuttle project start
```

5. 部署！🚀

```sh
cargo shuttle deploy
```

你可以运行 `cargo shuttle run` 在本地测试你的应用。

查看一些完整的 Actix Web 示例 [这里](https://github.com/shuttle-hq/shuttle-examples/tree/main/actix-web)。

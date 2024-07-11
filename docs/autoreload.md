---
title: 自动重载
---

# 自动重载开发服务器

在开发过程中，让 cargo 在代码更改时自动重新编译是非常方便的。这可以通过使用 [`cargo-watch`] 非常容易地实现。

```sh
 cargo watch -x run
```

## 历史说明

本页面的旧版本曾推荐使用 systemfd 和 listenfd 的组合，但这种方法有很多陷阱，并且很难正确集成，特别是在更广泛的开发工作流程中。我们认为 [`cargo-watch`] 足以满足自动重载的需求。

[`cargo-watch`]: https://github.com/passcod/cargo-watch

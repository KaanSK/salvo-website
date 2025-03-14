---
title: "Serve Static"
weight: 8060
menu:
  book:
    parent: "middlewares"
---

将静态文件或者内嵌的文件作为服务提供的中间件.

## 配置 Cargo.toml

```toml
salvo = { version = "*", features = ["serve-static"] }
```

## 主要功能

* `StaticDir` 提供了对静态本地文件夹的支持. 可以将多个文件夹的列表作为参数. 比如:

    ```rust
    use salvo_core::routing::Router;
    use salvo_core::Server;
    use salvo_extra::serve::StaticDir;

    #[tokio::main]
    async fn main() {
        tracing_subscriber::fmt().init();
        
        let router = Router::new()
            .path("<**path>")
            .get(StaticDir::new(vec!["examples/static/body", "examples/static/girl"]));
        Server::new(TcpListener::bind("127.0.0.1:7878")).serve(router).await;
    }
    ```
    如果在第一个文件夹中找不到对应的文件, 则会到第二个文件夹中找.

* 提供了对 `rust-embed` 的支持, 比如:
    ```rust
    use rust_embed::RustEmbed;
    use salvo::prelude::*;
    use salvo::serve_static::static_embed;

    #[derive(RustEmbed)]
    #[folder = "static"]
    struct Assets;

    #[tokio::main]
    async fn main() {
        tracing_subscriber::fmt().init();

        let router = Router::with_path("<**path>").get(static_embed::<Assets>().with_fallback("index.html"));
        tracing::info!("Listening on http://127.0.0.1:7878");
        Server::new(TcpListener::bind("127.0.0.1:7878")).serve(router).await;
    }
    ```

    `with_fallback` 可以设置在文件找不到时, 用这里设置的文件代替, 这个对应某些单页网站应用来还是有用的.
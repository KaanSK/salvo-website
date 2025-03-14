---
title: "文件上传"
weight: 7060
menu:
  book:
    parent: "topics"
---

Salvo 处理文件上传也是相当简单的, 比如处理单个文件的上传:

```rust
#[handler]
async fn upload(req: &mut Request, res: &mut Response) {
    let file = req.file("file").await;
    if let Some(file) = file {
        let dest = format!("temp/{}", file.file_name().unwrap_or("file"));
        println!("{}", dest);
        let info = if let Err(e) = std::fs::copy(&file.path(), Path::new(&dest)) {
            res.set_status_code(StatusCode::INTERNAL_SERVER_ERROR);
            format!("file not found in request: {}", e)
        } else {
            format!("File uploaded to {}", dest)
        };
        res.render(Text::Plain(info));
    } else {
        res.set_status_code(StatusCode::BAD_REQUEST);
        res.render(Text::Plain("file not found in request"));
    };
}
```

处理多个文件上传:

```rust
#[handler]
async fn upload(req: &mut Request, res: &mut Response) {
    let files = req.files("files").await;
    if let Some(files) = files {
        let mut msgs = Vec::with_capacity(files.len());
        for file in files {
            let dest = format!("temp/{}", file.file_name().unwrap_or("file"));
            if let Err(e) = std::fs::copy(&file.path(), Path::new(&dest)) {
                res.set_status_code(StatusCode::INTERNAL_SERVER_ERROR);
                res.render(Text::Plain(format!("file not found in request: {}", e)));
            } else {
                msgs.push(dest);
            }
        }
        res.render(Text::Plain(format!("Files uploaded:\n\n{}", msgs.join("\n"))));
    } else {
        res.set_status_code(StatusCode::BAD_REQUEST);
        res.render(Text::Plain("file not found in request"));
    }
}
```
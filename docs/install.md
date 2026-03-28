# Agent/Tauri 项目接入指南

将当前模板的 Python 能力迁移到你已有的 Agent/Tauri 项目。

## 1) 复制文件

从本模板复制以下内容到目标项目：

- `tauri/.cargo`
- `tauri/.taurignore`
- `tauri/python/tauri_app`
- `tauri/pyproject.toml`
- `tauri/src/python`
- `tauri/tauri.bundle.json`

## 2) 合并配置

将以下配置块合并到你的项目（不要整文件覆盖）：

- `tauri/tauri.conf.json` 的 `build.features`
- `tauri/Cargo.toml` 中 `# Python Support Configuration` 对应配置
- 根目录 `.gitignore` 中 `# pytauri` 对应条目

## 3) 调整 Rust 入口

### `tauri/src/lib.rs`

```rust
use tauri::{ipc::Invoke, Wry};

pub mod python;

pub fn handler() -> impl Fn(Invoke<Wry>) -> bool + Send + Sync + 'static {
    tauri::generate_handler![]
}

pub fn builder() -> tauri::Builder<tauri::Wry> {
    tauri::Builder::default()
        // ...
        .invoke_handler(handler())
}
```

### `tauri/src/main.rs`

```rust
use std::convert::Infallible;
use std::error::Error;

use pytauri_lib::python::builder::builder;

fn main() -> Result<Infallible, Box<dyn Error>> {
    let exit_code = builder()?.build()?.run();
    std::process::exit(exit_code);
}
```

## 4) 验证接入

在项目根目录执行：

```powershell
cd tauri
uv venv --python-preference only-system
uv pip install -e .
cd ..
pnpm tauri dev
```

能正常启动且 Python 相关调用可用，即接入完成。


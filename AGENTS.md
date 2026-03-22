# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project Overview

TurboFetch is a high-performance concurrent file downloader written in Rust. It supports cross-platform operation, resumable downloads (HTTP Range), and a simple CLI interface.

## Build / Lint / Test Commands

```bash
# Build debug
cargo build

# Build release
cargo build --release

# Run the program
cargo run

# Run with arguments
cargo run -- <URL>

# Run all tests
cargo test

# Run a single test by name
cargo test test_name

# Run tests matching a pattern
cargo test download_

# Run tests in a specific module
cargo test module::test_name

# Lint (clippy)
cargo clippy

# Lint with warnings treated as errors
cargo clippy -- -D warnings

# Format code
cargo fmt

# Check formatting without applying
cargo fmt -- --check

# Check for outdated dependencies
cargo outdated

# Audit dependencies for vulnerabilities
cargo audit
```

Always run `cargo clippy` and `cargo fmt --check` before considering a task complete.

## Project Structure

```
src/
  main.rs        # Entry point, CLI parsing, and orchestration
```

As the project grows, prefer this layout:

```
src/
  main.rs        # CLI entry point
  lib.rs         # Public API and re-exports
  download.rs    # Core download logic
  concurrent.rs  # Concurrent chunk management
  resume.rs      # Resumable download / Range support
  error.rs       # Error types
  cli.rs         # CLI argument parsing
  utils.rs       # Shared utilities
```

## Code Style

### Imports

- Group imports: std → third-party crates → local modules
- Separate groups with a blank line
- Use `use` for frequently used items; prefer full paths for one-off items

```rust
use std::path::PathBuf;
use std::sync::Arc;

use reqwest::Client;
use tokio::sync::Semaphore;

use crate::error::DownloadError;
```

### Naming

- Types / structs / enums: `PascalCase` (e.g. `DownloadTask`, `ChunkRange`)
- Functions / variables / fields: `snake_case` (e.g. `fetch_chunk`, `file_size`)
- Constants: `SCREAMING_SNAKE_CASE` (e.g. `MAX_RETRIES`)
- Acronyms: treat as words (`HttpRange`, not `HTTPRange`)

### Error Handling

- Define custom error types with `thiserror` for library code
- Use `anyhow::Result` in `main.rs` and CLI-facing code
- Propagate errors with `?` — avoid `.unwrap()` in non-test code
- Provide context messages for user-facing errors

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum DownloadError {
    #[error("网络请求失败: {0}")]
    NetworkError(#[from] reqwest::Error),
    #[error("文件写入失败: {0}")]
    IoError(#[from] std::io::Error),
    #[error("不支持断点续传")]
    ResumeNotSupported,
}
```

### Async Code

- Use `tokio` as the async runtime
- Prefer `async fn` over returning `impl Future`
- Use `Arc` + `Semaphore` for concurrency control
- Always set timeouts on network requests

### Formatting

- 4-space indentation (Rust default)
- Max line length: 100 characters
- Trailing comma in multi-line lists and match arms
- Run `cargo fmt` to auto-format

### Comments

**所有注释必须使用中文。** 注释是代码的重要组成部分，必须认真对待。

- **每个函数、结构体、枚举都必须有 `///` 文档注释**，说明用途、参数、返回值和可能的错误
- **关键逻辑必须有行内注释**，解释"为什么这样做"而不仅仅是"做了什么"
- 模块顶部用 `//!` 注释说明模块职责
- 错误类型要注释每种错误的触发场景
- 不要写显而易见的注释（如 `i += 1; // i 加 1`）

```rust
//! 下载管理模块，负责文件的并发下载和断点续传。

/// 表示一个分块下载的范围。
#[derive(Debug, Clone)]
pub struct ChunkRange {
    /// 分块起始字节位置。
    pub start: u64,
    /// 分块结束字节位置（包含）。
    pub end: u64,
}

/// 下载指定 URL 的文件到本地路径。
///
/// # 参数
/// * `url` - 远程文件地址
/// * `dest` - 本地保存路径
/// * `concurrency` - 并发连接数
///
/// # 错误
/// 返回 `DownloadError`，可能情况：
/// - 网络请求失败
/// - 服务器不支持 Range 请求
/// - 文件写入失败
pub async fn download(url: &str, dest: &Path, concurrency: usize) -> Result<(), DownloadError> {
    // 先发送 HEAD 请求检测服务器是否支持 Range
    let supports_range = check_range_support(url).await?;

    if supports_range {
        // 支持断点续传，按并发数分块下载
        download_chunks(url, dest, concurrency).await?;
    } else {
        // 不支持则回退到单线程下载
        download_single(url, dest).await?;
    }

    Ok(())
}
```

### Testing

- Unit tests go in the same file as the code they test, inside `#[cfg(test)] mod tests`
- Integration tests go in `tests/` directory
- Use descriptive test names: `test_resume_partial_download`
- Use `#[tokio::test]` for async tests

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_chunk_range_split() {
        let ranges = ChunkRange::split(1000, 4);
        assert_eq!(ranges.len(), 4);
    }
}
```

## Commit Messages

Use Chinese with conventional commit format. See `prompts/commit.md` for the full prompt.

```
<type>(<scope>): <简述>

<body>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `ci`, `revert`

## Dependencies

Prefer well-maintained crates. Suggested for this project:

| Crate | Purpose |
|-------|---------|
| reqwest | HTTP client |
| tokio | Async runtime |
| clap | CLI argument parsing |
| indicatif | Progress bar |
| thiserror | Error types |
| anyhow | Error context |
| futures | Async utilities |

Add dependencies with `cargo add <crate>`.

## Platform Notes

- Test on Windows (primary dev platform)
- Use `std::path::Path` for file paths — never hardcode `/` or `\`
- Use `dirs` or `std::env::current_dir()` for cross-platform directory handling

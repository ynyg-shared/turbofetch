# TurboFetch

一个用 Rust 编写的高性能并发文件下载器，支持跨平台、断点续传和直观的命令行界面。

## 功能特性

- **并发下载** — 可配置并发数，同时下载多个文件
- **断点续传** — 支持 HTTP Range 请求，中断后可继续下载
- **跨平台** — 支持 Windows、macOS 和 Linux
- **高速下载** — 多线程分块下载，自动优化连接数
- **简单易用** — 简洁的命令行界面，开箱即用

## 安装

### 从源码编译

```bash
git clone https://github.com/ynyg-shared/turbofetch.git
cd turbofetch
cargo build --release
```

编译产物位于 `target/release/turbofetch`。

## 使用方法

```bash
# 下载单个文件
turbofetch <URL>

# 指定输出文件名
turbofetch <URL> -o filename.zip

# 指定并发数
turbofetch <URL> -c 8

# 断点续传
turbofetch <URL> -r

# 下载多个文件
turbofetch <URL1> <URL2> <URL3>
```

## 配置参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `-o, --output` | 输出文件名 | 远程文件名 |
| `-c, --concurrent` | 并发连接数 | 4 |
| `-r, --resume` | 断点续传 | false |
| `-d, --dir` | 下载目录 | 当前目录 |

## 作者

**屿你有关** — 978814197@qq.com

## 许可证

MIT

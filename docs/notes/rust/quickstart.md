---
title: 快速上手
createTime: 2025/09/10 19:16:36
permalink: /article/3t3w72a1/
---

## 1、安装rustup

::: code-tabs
@tab macOS&Linux

```shell
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

@tab window

```shell
# 前往 https://www.rust-lang.org/install.html 按照说明安装Rust
```

:::

## 2、安装链接器

**链接器(linker)** 是Rust用来将其编译的输出链接成一个文件的程序
在macOS上安装一个C编译器，它通常包括一个链接器。一些常见的Rust包依赖于C代码

::: code-tabs
@tab macOS

```shell
xcode-select --install
```

@tab Linux

```shell
# 根据发行版（distribution）文档安装 GCC 或 Clang
```

@tab Ubuntu

```shell
# 安装 build-essential 包
```

:::

## 3、rust命令使用

```shell
# 检查是否正确安装
rustc --version

# 更新
rustup update

# 删除
rustup self uninstall
```

## 4、初体验

::: tip first.rs
```rust
fn main() {
  println!("Hello, world!");
}

// rustc first.rs
// ./first
```

:::

## [cargo](https://doc.rust-lang.org/cargo)

cargo是rust的构建系统和包管理器

```shell
cargo --version

# 使用cargo创建项目
cargo new xxxx

# 注意：git 是一个常用的版本控制系统（version control system，VCS）。可以通过 --vcs 参数使 cargo new 切换到其它版本控制系统（VCS），或者不使用 VCS。运行 cargo new --help 查看可用的选项。

# 构建 Cargo 项目
cargo build

# 运行 Cargo 项目
cargo run

# 快速检查代码确保其可以编译
cargo check

# 优化编译项目
cargo build --release
```

```toml
[package]
name = "xxxx"
version = "0.1.0"
edition = "2024"

[dependencies]
```
---
title: 快速上手
createTime: 2025/09/10 19:16:36
permalink: /article/3t3w72a1/
---

## 1、安装rustup

`Rust`将编译和运行行为分为两个单独的步骤。`Rust`是一种**预编译静态类型（ahead-of-time compiled）语言**。可以编译程序，并且将可执行文件送人，可执行文件的运行甚至不需要安装`Rust`。
仅仅使用`rustc`编译简单程序是没问题的，但是随着项目的增长，还需要管理项目的方方面面，并且让项目代码易于分享，所以还需要一个叫`Cargo`的工具，能够编写真实世界的`Rust`程序

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

echo $PATH

# 更新
rustup update

# 删除
rustup self uninstall

# 运行
rustc xxx.rs # 编译
./xxx.rs

# 如果rustc --version已经成功显示rustup的版本，但是rustc --version在某些地方仍然不生效时
source ~/.cargo/env
source ~/.zshrc 
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

## [Cargo](https://doc.rust-lang.org/cargo)

`Cargo`是`Rust`的构建系统和包管理器

```shell
cargo --version

# 使用cargo创建项目
cargo new xxxx

# 注意：git 是一个常用的版本控制系统（version control system，VCS）。可以通过 --vcs 参数使 cargo new 切换到其它版本控制系统（VCS），或者不使用 VCS。运行 cargo new --help 查看可用的选项。

# 构建 Cargo 项目
# 会创建一个可执行文件 target/debug/hello_cargo （在 Windows 上是 target\debug\hello_cargo.exe），而不是放在目前目录下。由于默认的构建方法是调试构建（debug build），Cargo 会将可执行文件放在名为 debug 的目录中。可以通过这个命令运行可执行文件：./target/debug/xxxx
cargo build

# 运行 Cargo 项目
# 并没有重新构建，而是直接运行了二进制文件
cargo run

# 在不生成二进制文件的情况下快速检查代码确保其可以编译
cargo check

# 优化编译项目
# 在 target/release 而不是 target/debug 下生成可执行文件
cargo build --release

# 升级，会忽略Cargo.lock，并计算Cargo.toml声明的最新版本
cargo update

# 构建所有本地依赖提供的文档并在浏览器打开（我勒个都，这个是真不错啊，省的自己去找文档）
cargo doc --open
```

```toml
[package]  // 是一个片段 section 标题，表明下面的语句用来配置一个包。随着我们在这个文件增加更多的信息，还将增加其他 section。
name = "xxxx"
version = "0.1.0"
edition = "2024"

[dependencies]
```
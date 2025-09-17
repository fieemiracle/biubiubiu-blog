---
title: 包、Crate和模块管理
createTime: 2025/09/17 16:59:42
permalink: /rust/kwjkhmb9/
---

## 5.1 包和`Crate`

`Crate`有两种形式：二进制`Crate`和库`Crate`。

二进制`Crate`：可以被编译为可执行程序，比如命令行程序或者服务端

库`Crate`：没有`main`函数，不会编译为可执行程序，定义了可供多个项目复用的功能模块

关键字：命名项的 `路径（paths）`；用来将路径引入作用域的 `use` 关键字；以及使项变为公有的 `pub` 关键字、 `as` 关键字、`外部包（external packages）`和 `glob` 运算符（glob operator）。

::: important 模块小抄
1、**从 `crate` 根节点开始**: 当编译一个 `crate`, 编译器首先在 `crate` 根文件（通常，对于一个库 `crate` 而言是 `src/lib.rs`，对于一个二进制 `crate` 而言是 `src/main.rs`）中寻找需要被编译的代码。

2、**声明模块**: 在` crate` 根文件中，你可以声明一个新模块；比如，用 `mod garden;`, 声明了一个叫做 `garden` 的模块。编译器会在下列路径中寻找模块代码：

- *内联*，用大括号替换 `mod garden` 后跟的分号

- 在文件 `src/garden.rs`

- 在文件 `src/garden/mod.rs`

3、**声明子模块**: 在除了 `crate` 根节点以外的任何文件中，可以定义子模块。比如，可能在 `src/garden.rs` 中声明 `mod vegetables;`。编译器会在以父模块命名的目录中寻找子模块代码：

- 内联，直接在 `mod vegetables` 后方不是一个分号而是一个大括号

- 在文件 `src/garden/vegetables.rs`

- 在文件 `src/garden/vegetables/mod.rs`

4、**模块中的代码路径**: 一旦一个模块是你 `crate` 的一部分，你可以在隐私规则允许的前提下，从同一个 `crate` 内的任意地方，通过代码路径引用该模块的代码。举例而言，一个 `garden vegetables` 模块下的 `Asparagus` 类型可以通过 `crate::garden::vegetables::Asparagus` 访问。

5、**私有 vs 公用**: 一个模块里的代码默认对其父模块私有。为了使一个模块公用，应当在声明时使用 `pub mod` 替代 `mod`。为了使一个公用模块内部的成员公用，应当在声明前使用`pub`。

6、**use 关键字**: 在一个作用域内，use关键字创建了一个项的快捷方式，用来减少长路径的重复。在任何可以引用 `crate::garden::vegetables::Asparagus` 的作用域，可以通过 `use crate::garden::vegetables::Asparagus; `创建一个快捷方式，然后你就可以在作用域中只写 `Asparagus` 来使用该类型。
:::

## 5.2 引用模块树中项的路径

路径有两种形式：

（1）**绝对路径**：以 `crate` 根开头的完整路径，对于外部`crate`的代码，是以`crate`名开头的绝对路径，对于前`crate`的代码，则以字面值`crate`开头

（2）**相对路径**：从当前模块开始，以 `self`、`super`或当前模块中的某个标识符开头

绝对路径和相对路径都后跟一个或多个由双冒号（`::`）分割的标识符

```rust
mod front_of_house {
  mod hosting { // hosting模块是私有的
    fn add_to_waitlist() {}
  }
}

pub fn eat_at_restaurant() {
  // 绝对路径
  crate::front_of_house::hosting::add_to_waitlist();

  // 相对路径
  front_of_house::hosting::add_to_waitlist();
}
```
选择使用相对路径还是绝对路径要取决于你的项目，也取决于你是更倾向于将项的定义代码与使用该项的代码分开来移动，还是一起移动。
在 Rust 中，所有项（函数、方法、结构体、枚举、模块和常量）默认对父模块都是私有的。如果希望创建一个如函数或结构体的私有项，可以将其放入一个模块。

### 1、使用`pub`关键字暴露路径

```rust
mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}

// -- snip --

```

如上述修改还是有问题，因为；模块公有并不使其内容也是公有的。`add_to_waitlist` 函数是私有的。私有性规则不但应用于模块，还应用于结构体、枚举、函数和方法。


模块上的 `pub` 关键字只允许其父模块引用它，而不允许访问内部代码。因为模块是一个容器，只是将模块变为公有能做的其实并不太多；同时需要更深入地选择将一个或多个项变为公有。

需要改成：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

// -- snip --

```

## 5.3 使用`use`关键字将路径引入作用域

无论我们选择 `add_to_waitlist` 函数的绝对路径还是相对路径，每次我们想要调用 `add_to_waitlist` 时，都必须指定`front_of_house` 和 `hosting`。

使用`use`即可解决问题：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```
在作用域中增加 use 和路径类似于在文件系统中创建软连接（符号连接，symbolic link）。

### 1、使用 as 关键字提供新的名称

使用父模块可以区分这两个 `Result` 类型，如果使用`Result`的完整路径，则不知道用的是哪个`Resut`

```rust
use std::fmt;
use std::io;

fn function1() -> fmt::Result {
    // --snip--
}

fn function2() -> io::Result<()> {
    // --snip--
}
```

使用 `use` 将两个同名类型引入同一作用域这个问题还有另一个解决办法：在这个类型的路径后面，我们使用 `as` 指定一个新的本地名称或者别名。

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```

### 2、 使用 pub use 重导出名称

使用 `use` 关键字，将某个名称导入当前作用域后，该名称对此作用域之外还是私有的。若要让作用域之外的代码能够像在当前作用域中一样使用该名称，可以将` pub` 与 `use` 组合使用。这种技术被称为`重导出（re-exporting）`，因为在把某个项目导入当前作用域的同时，也将其暴露给其他作用域。

::: info 通过`pub use`使名称可从新作用域中被导入至任何代码
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```
:::

### 3、使用外部包

[crates.io](https://crates.io/) 上有很多 Rust 社区成员发布的包，将其引入你自己的项目都需要一道相同的步骤：在 Cargo.toml 列出它们并通过 use 将其中定义的项引入项目包的作用域中。

注意 std 标准库对于你的包来说也是外部 crate。因为标准库随 Rust 语言一同分发，无需修改 Cargo.toml 来引入 std，不过需要通过 use 将标准库中定义的项引入项目包的作用域中来引用它们。

```rust
use std::collections::HashMap;

```

### 4、使用嵌套路径来清理大量的 `use` 列表

当需要引入很多定义于相同包或相同模块的项时，为每一项单独列出一行会占用源码大量的垂直空间。

```rust
// --snip--
use std::cmp::Ordering;
use std::io;
// --snip--

```

可以使用嵌套路径将相同的项在一行中引入作用域。这么做需要指定路径的相同部分，接着是两个冒号，接着是大括号中的各自不同的路径部分

```rust
// --snip--
use std::{cmp::Ordering, io};
// --snip--

```

```rust
use std::io;
use std::io::Write;

// 可以写成
use std::{self, Write}
```

### 5、`glob` 运算符（慎用）

如果希望将一个路径下所有公有项引入作用域，可以指定路径后跟 `*` glob 运算符：
```rust
use std::collections::*;

```

## 5.4 将模块拆分成多个文件

::: tip

`src/lib.rs `

```rust 模块拆分前
// 声明 front_of_house 模块，其内容将位于 `src/front_of_house.rs`
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
  hosting::add_to_waitlist();
}
```

`src/front_of_house.rs`

```rust
// 在 `src/front_of_house.rs` 中定义 `front_of_house` 模块
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```
:::

::: info 模块拆分后
`src/front_of_house.rs`
```rust
pub mod hosting;
```
`src/front_of_house/hosting.rs`
```rust
pub fn add_to_waitlist() {}

```
`src/lib.rs` 中的 `pub use crate::front_of_house::hosting` 语句也并未发生改变，`use` 也不会对哪些文件会被编译为 `crate` 的一部分有任何影响。`mod` 关键字声明了模块，而 `Rust` 会在与模块同名的文件中查找模块的代码。
:::
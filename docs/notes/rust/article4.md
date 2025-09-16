---
title: 使用结构体组织关联的数据
createTime: 2025/09/16 17:25:04
permalink: /rust/ejun1hlf/
---

## 3.1 结构体的定义和实例化

结构体和元组医院，每一部分可以是不同类型。但不同于元组，结构体需要命名个部分数据一边能清楚的表明其值的意义。

结构体比元组更灵活：不需要依赖顺序来指定或访问示例中的值。

::: tip 结构体定义
```rust
struct User {
  name: String,
  age: i32,
  email: String,
  active: bool,
}
```
:::

::: info 创建结构体实例
```rust
fn main() {
  let user_one = User {
    name: String::from("Linda"),
    age: 18,
    email: String::from("jdjdjdj@qq.com"),
    active: true,
  }

  let user_second = User {
    age: 20,
    // ⚠️ 由于email非Copy trait，所以user_one在user_second创建之后就不可用了
    email: user_one.email,
    // 如果name和email都是用String创建，其余字段引用user_one，那在创建user_second的时候，user_one仍然可用
    ...user_one
  }
}
```
:::

::: caution 改变结构体字段的值
```rust
fn main() {
  // --snip--

  user_one.email = String::from("hhhsh@qq.com");
}
```

整个实例必须是可变的：不允许只将某个字段标记为可变

可以在函数体的最后一个表达式中构造一个结构体的新实例，来隐式地返回这个实例

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}

```
:::

::: important 使用没有命名字段的元组结构体来创建不同的类型

元组结构体：有着结构体名称提供的含义，但没有具体的字段名，只有字段类型

需要给元组取名字并且让元组称为与其他元组不同类型时，元组结构体比较有用
```rust
struct Color(i32, i32, i32);

struct Point(i32, i32, i32);

fn main() {
  let black = Color(0, 0, 0);

  let origin = Point(0, 0, 0);
}
```

:::

### 1、类单元结构体

一个没有任何字段的结构体

```rust
struct AlwaysEqual;

fn main() {
  let subject = AlwaysEqual;
}
```

```rust
#[derive(Debug)]
struct Rectangle {
  width: u32,
  height: u32,
}

fn main() {
  let rect1 = Rectangle {
    width: 20,
    height: 30,
  }

  println!("rect1 is {rect1:#?}"); // 接收的是引用。会打印到标准输出控制台流
  dbg!(&rect1); // 接收一个表达式的所有权。会打印到标准错误控制台流
}

```

## 3.2 结构体示例程序

## 3.3 方法语法

`方法（method）`与函数类似：它们使用 `fn` 关键字和名称声明，可以拥有参数和返回值，同时包含在某处调用该方法时会执行的代码。不过方法与函数是不同的，因为它们在结构体的上下文中被定义（或者是枚举或 `trait` 对象的上下文

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
  fn area(&self) -> u32 {
      self.width * self.height
  }
  fn square(size: u32) -> Self {
      Self {
          width: size,
          height: size,
      }
  }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

`impl` 块（`impl` 是 `implementation` 的缩写），这个 `impl` 块中的所有内容都将与 `Rectangle` 类型相关联

所有在 impl 块中定义的函数被称为 关联函数（associated functions），因为它们与 impl 后面命名的类型相关。

可以定义不以 self 为第一参数的关联函数（因此不是方法），因为它们并不作用于一个结构体的实例。

不是方法的关联函数经常被用作返回一个结构体新实例的构造函数。这些函数的名称通常为 new ，但 new 并不是一个关键字。
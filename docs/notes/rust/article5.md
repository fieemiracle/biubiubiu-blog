---
title: 枚举和模式匹配
createTime: 2025/09/16 20:54:48
permalink: /rust/jkdx9oxx/
---


## 4.1 枚举的定义

枚举给予一个途径去声明某个值是一个集合中的一员。

```rust
// V4和V6被称为枚举的变体
enum IpAddrKind {
  V4,
  V6
}

let ip_v4 = IpAddrKind::V4;
let ip_v6 = IpAddrKind::V6;

// 可以定义一个函数来接收任何IpAddrKind类型的参数
fn route(ip_kind: IpAddrKind) {}

// 和struct
struct IpAddr {
  kind: IpAddrKind,
  address: String::from("127.0.0.1"),
}

let home = IpAddr {
  kind: IpAddrKind::V4,
  address: String::from("127.0.0.1"),
};

let loopback = IpAddr {
  kind: IpAddrKind::V6,
  address: String::from("::1"),
};
```

上述枚举和结构体的结合还有一种较为简洁的方式来表达，仅仅使用枚举并将数据放进每一个枚举变体而不是将枚举作为结构体的一部分。

```rust
enum IpAddr {
  V4(String),
  V6(String)
}

let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));
```

直接将数据附加到枚举的每个变体上，这样就不需要一个额外的结构体了。
`IpAddr::V4()` 是一个获取 `String` 参数并返回 `IpAddr` 类型实例的函数调用。
作为定义枚举的结果，这些构造函数会自动被定义。（好家伙，这句话什么意思？）

用枚举替代结构体还有一个优势：每个变体可以处理不同类型和数量的数据。例如：将V4地址存储为四个u8值而V6地址仍然表现为一个String，就不能使用结构体了，枚举可以：

```rust
enum IpAddr {
  V4(u8, u8, u8, u8),
  V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

```

事实证明，存储和编码IP地址实在是太常见了，以致标准库提供了一个开箱即用的定义：

```rust
struct Ipv4Addr {
  // ...
}
struct Ipv6Addr {
  // ...
}

enum IpAddr {
  V4(Ipv4Addr),
  V4(Ipv6Addr),
}
```

```rust
enum Message {
  Quit, // 没有关联任何数据
  Move { x: i32, y: i32 }, // 类似结构体包含命名字段
  Write(String), // 包含单独一个String
  ChangeColor(i32, i32, i32), // 包含三个i32
}

// 使用impl来为枚举定义方法
impl Message {
  fn call(&self) {
    // ...
  }
}
let m = Message::Write(String::from("hello"));
m.call();

// 有关联值的枚举的方式和定义多个不同类型的结构体的方式很像
struct QuitMessage; // 类单元结构体
struct MoveMessage {
  x: i32,
  y: i32,
}
struct WriteMessage(String); // 元组结构体
struct ChangeColorMessage(i32, i32, i32); // 元组结构体
```

### 1、`Option`枚举及其相对于空值的优势

`Option` 是标准库定义的另一个枚举。

空值尝试表达的概念仍然是有意义的：空值是一个因为某种原因目前无效或缺失的值。

Rust并没有空值，但是拥有一个可以编码存在或不存在概念的枚举，`Option<T>`

::: info

```rust
enum Option<T> { // T帆型类型参数
  None,
  Some(T),
}

let some_number = Some(5);
let some_char = Some('e');

let absent_number: Option<i32> = None;

```
`Option<T>`枚举被包含在`prelude`之中，无需将其显式引入作用域，变体也不需要`Option::`前缀
:::

## 4.2 `match`控制流结构

`match`是控制流运算符，允许将一个值与一系列的模式相比较，并根据相匹配的模式执行相应代码
模式可由字面值、变量、通配符和其他内容构成

::: caution 一个枚举和一个以枚举变体作为模式的`match`表达式

```rust
enum Coin {
  Penny,
  Nickel,
  Dime,
  Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
  match coin {
    Coin::Penny => {
      println!("Lucky penny!");
      1
    },
    Coin::Nickel => 5,
    Coin::Dime => 10,
    Coin::Quarter => 25,
  }
}

```

:::

### 1、绑定值的模式

::: tip Quarter变体也存放了一个UsState值的Coin枚举

```rust
#[derive(Debug)] // 这样可以立刻看到州的名称
enum UsState {
  Alabama,
  Alaska,
  // --snip--
}

enum Coin {
  Penny,
  Nickel,
  Dime,
  Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
  match coin {
    Coin::Penny => 1,
    Coin::Nickel => 5,
    Coin::Dime => 10,
    Coin::Quarter(state) => {
      println!("State quarter from {state:?}!");
      25
    }
  }
}


```
:::

### 2、匹配 `Option<T>`

::: important 一个在 `Option<i32>` 上使用 `match` 表达式的函数

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
  match x {
    None => None,
    Some(i) => Some(i + 2)
  }
}

let five = Some(9);
let six = plus_one(five);
let none = plus_one(None);
```

:::

### 3、匹配是穷尽的

`match`的分支必须覆盖所有的可能性

```rust
fn plus(x: Option<T>) -> Option<T> {
  match x {
    Some(i) => Some(i * 10) // 分支没有None的情况，会造成一个bug
  }
}
```

Rust中的匹配是穷尽的：必须穷举到最后的可能性来使代码有效

### 4、通配模式和_占位符

```rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    other => move_player(other), // 将匹配所有未被特殊列出的值，必须将通配分支放在最后，因为模式是按顺序匹配的
    // _ => rerool(), // 不想使用通配模式获取的值时，也可以使用_这个特殊模式，可以匹配任意值而不绑定到该值
    // _ => (), // ()表示单元值，无事发生
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn move_player(num_spaces: u8) {}
fn rerool() {}
```

## 4.3 `if let` 和 `let else` 简洁控制流

::: warning
`match` 只关心当值为 `Some` 时执行代码
```rust
let config_max = Some(3u8);
match config_max {
    Some(max) => println!("The maximum is configured to be {max}"),
    _ => (),
}
```

`if let` 语法获取通过等号分隔的一个模式和一个表达式

```rust
let config_max = Some(3u8);
if let Some(max) = config_max {
  println!("123");
}
```
:::

### 1、使用`let else`保持在愉快路径

```rust
impl UsState {
  fn existed_in(&self, year: u16) -> bool {
    match self {
      UsState::Alabama => year >= 1819,
      UsState::Alaska => year >= 1959,
      // -- snip --
    }
  }
}

// 等价于
// 方式1：使用嵌套在 `if let` 中的条件来检查一个州在 1900 年是否存在
fn describe_state_quarter(coin: Coin) -> Option<String> {
  if let Coin::Quarter(state) = coin {
    if state.existed_in(1900) {
      Some(format!("{state:?} is pretty old, for America!"))
    } else {
      Some(format!("{state:?} is relatively new."))
    }
  } else {
    None
  }
}

// 或者
// 方式2：使用 `if let` 来产生一个值或提前返回
fn describe_state_quarter(coin: Coin) -> Option<String> {
  let state = if let Coin::Quarter(state) = coin {
    state
  } else {
    return None;
  };

  if state.existed_in(1900) {
    Some(format!("{state:?} is pretty old, for America!"))
  } else {
    Some(format!("{state:?} is relatively new."))
  }
}

// 方式1和方式2都有些繁琐，`if let`一个分支产生一个值，而另一个分支则直接从函数中返回
// `let...else`语法左侧是一个模式，右侧是一个表达式，没有if分支，只有else分支

fn describe_state_quarter(coin: Coin) -> Option<String> {
  let Coin::Quarter(state) = coin else {
    return None;
  };

  if state.existed_in(1900) {
    Some(format!("{state:?} is pretty old, for America!"))
  } else {
    Some(format!("{state:?} is relatively new."))
  }
}

```
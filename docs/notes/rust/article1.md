---
title: 猜数字游戏
createTime: 2025/09/12 16:38:54
permalink: /rust/vfvg81gn/
---

## 猜数字游戏

::: important

```rust

// 获取用户输入并打印结果作为输出，将 io 输入/输出库引入当前作用域
// Rust 设定了若干个会自动导入到每个程序作用域中的标准库内容，这组内容被称为 预导入（prelude） 内容
// 如果需要的类型不在预导入内容中，就必须使用 use 语句显式地将其引入作用域
use std::cmp::Ordering;
use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    // println!("The secret number is: {secret_number}");

    loop {
        println!("Please input your guess.");

        // let 创建一个变量，默认是不可变的
        // mut 创建一个可变的变量
        // String 是一个标准库提供的字符串类型，它是 UTF-8 编码的可增长文本块。
        // ::new 表明new 是 String 类型的一个 关联函数（associated function）
        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        // 虽然变量重名了，但是Rust允许用一个新值来遮蔽（这个功能常用于将一个类型的值转换为另一个类型的值）
        // （服了，这都行，怎么有一种被颠覆三观的感觉😱）
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        // Rust有一个静态强类型系统，也有类型推断
        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            },
        }
    }
}

```

:::

## Rust特点

### **内存安全与零成本抽象**

- Rust通过所有权系统在编译时防止内存泄漏、悬空指针等内存安全问题

- 无需垃圾回收器，性能接近C/C++

### **所有权系统（Ownership）**

- 每个值都有一个所有者

- 值在所有者离开作用域时被自动清理

- 通过借用（borrowing）实现引用，避免数据竞争

### **变量默认不可变**

```rust
let guess = String::new();  // 不可变
let mut guess = String::new();  // 可变，需要显式声明
```

### **强类型系统与类型推断**

```rust
let guess: u32 = match guess.trim().parse() {  // 显式类型注解
    Ok(num) => num,
    Err(_) => continue,
};
```

### **模式匹配（Pattern Matching）**

```rust
match guess.cmp(&secret_number) {
    Ordering::Less => println!("Too small!"),
    Ordering::Greater => println!("Too big!"),
    Ordering::Equal => {
        println!("You win!");
        break;
    },
}
```

### **错误处理机制**

- 使用`Result<T, E>`类型处理可能失败的操作

- `expect()`方法处理错误，`match`语句优雅处理`Ok`和`Err`情况

### **宏系统**
```rust
println!("Hello, world!");  // 宏调用，注意!
```

### **包管理与依赖**

```TOML
[dependencies]
rand = "0.8.5"
```

### **变量遮蔽（Shadowing）**

```rust
let guess = String::new();  // 字符串类型
let guess: u32 = match guess.trim().parse() {  // 遮蔽为数字类型
    Ok(num) => num,
    Err(_) => continue,
};
```

### **模块系统**

- 使用`use`语句导入标准库和第三方库

- 预导入`（prelude）`自动包含常用类型

### **函数式编程特性**

- 支持闭包、迭代器

- 链式调用和函数组合

### **并发安全**

- 编译时保证线程安全

- 通过类型系统防止数据竞争

#### **性能优化**

- 零成本抽象

- 编译时优化

- 无运行时开销
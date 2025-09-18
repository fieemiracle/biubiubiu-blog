---
title: 常见集合
createTime: 2025/09/17 20:00:38
permalink: /rust/8qdy30u5/
---

## 6.1 使用 `Vector` 储存列表

`Vec<T>`，也被称为 `vector`。`vector` 允许在一个单独的数据结构中储存多于一个的值，在内存中彼此相邻地排列所有的值。`vector` 只能储存 ==相同类型== 的值。

### 1、新建`Vector`
::: tip 新建一个的空 `vector` 来储存 `i32` 类型的值

```rust
let v: Vec<i32> = Vec::new();
```

:::

::: info 新建一个包含初值的 `vector`

```rust
let v = vec![1, 2, 3];
```

:::

### 2、更新`Vector`
::: warning 使用`push`方法向`vector`增加值

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);

```

:::

使用 `&` 和 `[]` 会得到一个索引位置元素的引用。当使用索引作为参数调用 `get` 方法时，会得到一个可以用于 `match` 的 `Option<&T>`

### 3、读取`Vector`的元素

::: caution 使用索引语法或 `get` 方法来访问 `vector` 中的项

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("The third element is {third}");

// let test: &i32 = &v[100]; // 引用不存在的元素会造成panic

let third: Option<&i32> = v.get(2);
match third {
  Some(third) => println!("The third element is {third}"),
  None => println!("There is no third element."),
}

// let test: Option<&i32> = v.get(100); // 传递数组外的索引，不会造成panic而是返回None

```

:::

::: important 尝试在拥有 vector 中项的引用的同时向其增加一个元素
```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);

println!("The first element is: {first}");

```
一旦程序获取了一个有效的引用，借用检查器将会执行所有权和借用规则来确保 `vector` 内容的这个引用和任何其他引用保持有效。

`不能在相同作用域中同时存在可变和不可变引用的规则`

报错原因：在 `vector` 的结尾增加新元素时，在没有足够空间将所有元素依次相邻存放的情况下，可能会要求分配新内存并将老的元素拷贝到新的空间中。这时，第一个元素的引用就指向了被释放的内存。借用规则阻止程序陷入这种状况。
:::

### 4、遍历`Vector`中的元素

::: tip `for`循环

遍历所有元素。使用`for`循环获取`i32`值的`vector`的每一个元素的不可变引用并将其打印。

```rust
let vector_one = vec![100, 34, 488];
for item in &verctor_one {
  println!("{item}");
}
```
遍历可变`Vector`的每一个元素的可变引用以便改变每个元素
```rust
let mut vector_two = vec![100, 34, 488];
for item in &mut vector_two {
  println!("{item}");

  * item += 50;
}
```
为了修改可变引用所指向的值，在使用 += 运算符之前必须使用`解引用运算符（*）获取 i 中的值`
:::

### 5、使用枚举储存更多种类型

`vector` 只能储存相同类型的值,枚举的成员都被定义为相同的枚举类型
```rust
enum SpreadsheetCell {
  Int(i32),
  Float(f64),
  Text(String),
}

let row = vec![
  SpreadsheetCell::Int(3),
  SpreadsheetCell::Text(String::from("blue")),
  SpreadsheetCell::Float(10.12),
];

```

## 6.2 使用`字符串`储存`UTF-8`编码的文本

### 1、新建字符串

```rust
// 新建一个空的 String
let mut string_one = String::new();

// 使用 to_string 方法从字符串字面值创建 String
let string_two = "initial contents";
let s_one = string_two.to_string(); 

// 使用 to_string 方法从字符串字面值创建 String
let s_one = "initial contents".to_string(); 

// 使用 String::from 函数从字符串字面值创建 String
let s = String::from("initial contents");

```

### 2、更新字符串

```rust
let mut ss = String::from("foo");
ss.push_str("bar");
ss.push("hhh");
```

### 3、使用 + 运算符或 format! 宏拼接字符串

::: info
```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // 注意 s1 被移动了，不能继续使用

fn add(self, s: &str) -> String {}

```
`&s2`的类型是`&String`。当add函数被调用时，Rust使用了一个被称为`Deref强制转换`的技术，会把`&s2`转换为 `&s2[..]`,add函数没有获取s2的所有权，但是获取self的所有权

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3; // 显得笨重
let s = format!("{s1}-{s2}-{s3}"); // 使用的是引用，没有获取所有权，返回带有接过内容的String

```
:::

### 4、 索引字符串
```rust
let s1 = String::from("hi");
let h = s1[0]; // 会报错

```

`String`是一个`Vec<u8>`的封装，`不支持索引访问字符串`（跟如何在内存存储字符串有关系）
每个 `Unicode` 标量值需要`两个字节`存储。因此一个`字符串字节值的索引`并不总是对应一个有效的 `Unicode` 标量值

Rust 不允许使用索引获取 String 字符的原因是，索引操作预期总是需要常数时间（O(1)）。但是对于 String 不可能保证这样的性能，因为 Rust 必须从开头到索引位置遍历来确定有多少有效的字符。
## 6.3 使用`Hash Map`储存键值对
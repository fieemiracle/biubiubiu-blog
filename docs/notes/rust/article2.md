---
title: 常见编程概念
createTime: 2025/09/12 16:59:56
permalink: /rust/sefuc7iz/
---

## 1.1 变量与可变性

### 写在前面

- [x] 变量默认是不可变的

::: tip
```rust
let name = "lisa"

name = "blob" // ❌，不允许
```
:::

- [x] 通过 `mut`，允许把不可变变量使其可变
::: tip
```rust
let mut name = "Linda";

name = "John"; // ✅，允许
name = name.len(); // ❌，不允许，不能改变变量的类型
let name: u32 = match name.len().expect("error"); // ✅，声明类型，允许

let name = name.len(); // ✅，遮蔽，允许

```
:::

- [x] `常量 (constants)` 是绑定到一个名称的不允许改变的值，不允许使用`mut`。
      
      常量使用`const`关键字，声明时必须注明值的类型。

      常量可以在任何作用域中声明，包括全局作用域。

      常量只能被设置为常量表达式，而不可以是其他任何只能运行时计算出的值。

      `Rust`对常量的命名约定是在单词之间使用全大写加下划线。

::: tip
```rust
const THREHOLD_COUNTS: u32 = 3
```
:::

## 1.2 数据类型

- [x] 在`Rust`中，每一个值都有一个特定数据类型
- [x] 两类数据类型子集：(1)`标量(scalar)`、(2)`复合(compound)`
- [x] `Rust`是`静态类型（statically typed）`语言，在编译时就必须知道所有变量的类型

### 标量类型

  四种基本标量类型：`整型`、`浮点型`、`布尔类型`、`字符类型`

#### 整型

整型是一个没有小数部分的数字。

有符号数以二级制补码形式存储，每一个有符号的变体可以存储包含$-(2^{n-1})$ ~ $(2^{n-1}) - 1$在内的数字；

无符号的变体可以存储从$0$ ~ $(2^{n} - 1)$

`isize`和`usize`类型依赖运行程序的计算机架构：64位架构上是`64位`；32架构上是`32位`

::: tip Rust中的整型

| 长度        | 有符号           | 无符号  |
| ------------- |:-------------:| -----:|
| 8-bit      | i8 | u8    |
| 16-bit     | i16        |     u16 |
| 32-bit     | i32        |     u32 |
| 64-bit     | i64        |     u64 |
| 128-bit    | i128       |    u128 |
| 架构相关    | isize      |   usize |

:::

::: important Rust中的整型字面量

| 数字字面值        | 例子           |
| ------------- |:-------------:|
| Decimal(十进制)     | 98_222 |
| Hex(十六进制)     | 0xff       |
| Octal(八进制)     | 0o77        |
| Binary(二进制)     | 0b1111_0000   |
| Byte(单字节字符)(仅限于`u8`)    | b'A'      |

:::

::: warning 整型溢出


```rust
let count: u8 = 256
```

指定`count`为`u8`类型，但是存储了非0 ~ $(2^{n} - 1)$的值就会发生`整型溢出(integer overflow)`。这会导致：

（1）在`debug`模式编译时，Rust检查这类问题并使程序`panic`（`panic`： `Rust`用来表明程序因错误而退出）。

（2）使用`--release` `flag`在`release`模式中构建时，`Rust`不会检测会导致`panic`的整型溢出。相反发生整型溢出时，`Rust`会进行一种被称为二级制补码wrapping的操作，即比此类型能容纳的最大值还大的值会回绕到最小值，值256变0，值257变1，依此类推。

为显式地处理溢出的可能性，可以使用几类标准库提供的原始数字类型方法：

（1）所有模式下都可以使用`wrapping_*`方法进行wrapping，如`wrapping_add`

（2）如果`checked_*`方法出现溢出，返回None值

（3）用`overflow_*`方法返回值和一个布尔值，表示是否出现溢出

（4）用`staturating_*`方法在值的最小值和最大值进行饱和处理

:::


#### 浮点型

`Rust`的浮点数类型是`f32(单精度浮点数)`和`f64(双精度浮点数)`，分别占32位和64位。默认类型是`f64`，在现代`CPU`中，与`f32`速度几乎一样，不过精度更高，所有的浮点型都是有符号的。浮点数采用`IEEE-754`标准表示

#### 数值运算

`Rust`中的所有数字类型都支持基本数学运算：`加减乘除&取余`。整数除法会向零舍入到最接近的整数。

```rust
fn main() {
  let sum = 5 + 10;

  let difference = 95.5 - 4.3;

  let product = 4 * 10;

  let quotientc = 56.7 / 32.2;
  let truncated = -5 / 3;

  let remainder = 43 % 5;
}
```

#### 布尔类型

`Rust`中的布尔类型有两个可能的值：`true`&`false`

```rust
let t = true;

let f: bool = false;
```

#### 字符类型

`Rust`的`char`类型是语言中最原始的字母类型

::: warning

```rust
fn main() {
  let c = 'Z';
  let z: char = 'ℤ';
  let heart_eyed_cat = '🐱';
}
```

⚠️ 注意

（1）用单引号声明`char`字面值，使用双引号声明字符串字面值。

（2）`char`类型的大小为四个字节，并代表了一个`Unicode`标量值

（3）在 `Rust` 中，带变音符号的字母（Accented letters），中文、日文、韩文等字符，emoji（绘文字）以及零长度的空白字符都是有效的 char 值

:::

#### 复合类型

`Rust`有两个原生的复合类型：元组和数组

##### 元组类型

元组是一个将多个不同类型的值组合进一个复合类型的主要方式。元组固定长度：一旦声明，其长度不会增大或缩小。

元组的每个位置都有一个类型，而且这些不同值的类型也不必相同。

为了从元组中获取单个值，可以使用`模式匹配（pattern matching）`来`解构（destructure）`元组值

不带任何值的元组有个特殊的名称，叫做`单元（unit）元组`。()表示空值或空的返回类型，如果表达式不返回任何其他值，则会隐式返回单元值

```rust
fn main() {
  let tup: (i32, f64, u8) = (500, 6.4, 1)

  let (x, y, z) = tup;

  let first_val = tup.0;
  let second_val = tup.1;
  let third_val = tup.2;
}
```

##### 数组类型

与元组不同，数组中的每个元素的类型必须相同。数组的长度也是固定的。

当想要在`栈（stack）`而不是在`堆（heap）`上为数据分配空间，或者想要确保总是有固定数量的元素时，数组非常有用。

但是数组不如`vector`（`vector`类型是标准库提供的一个允许增长和缩小长度的类似数组的集合类型）类型灵活。

数组是可以在`栈（stack）`上分配的已知固定大小的单个内存块。可以使用索引来访问数组的元素

::: caution

```rust
fn main() {
  let arr = ["apple", "watermelon", "peach"];

  let number: [i32:5] = [1, 2, 3, 4, 5];

  let same_num = [3; 5]; // 等价于 [3，3，3，3，3]

  let first_num = number[0];
}
```

:::

## 1.3 函数

`Rust`代码中的函数和变量名使用`snake case`规范风格。`main`函数是很多程序的入口点，即使不调用也会执行。

`Rust` 不关心函数定义所在的位置，只要函数被调用时出现在调用之处可见的作用域内就行。

```rust
fn main() {
  println!("Hello rust");

  another_func(3, 'h');
}

fn another_func(num: i32, unit_label: char) {
  println!("Another function: {num}{unit_label}");
}
```

### 参数

参数是特殊变量，是函数签名的一部分。当函数拥有参数（形参）时，可以为这些参数提供具体的值（实参）。

在函数签名中，必须声明每个参数的类型

### 语句和表达式

`Rust`是一门基于表达式`（expression-based）`的语言。

- `语句（Statements）`是执行一些操作但不返回值的指令
- `表达式（Expressions）`计算并产生一个值

::: tip

```rust
fn main() {
  let name = (let nickname = "Linda Polci");
}
```

- 函数定义也是语句

- 语句不返回值

- 上述main函数调用会报错，是因为赋值语句不返回所赋的值。（比如`javascript&c`可以这么写： x = y = 10，`Rust`不能这么写）

- 表达式会计算出一个值，表达式可以是语句的一部分，函数调用是一个表达式，宏调用是一个表达式，用大括号创建的一个新的块作用域也是一个表达式

```rust
fn main() {
  let y = {
    let x = 10;
    x + 1 // 表达式的结尾没有分号，如果在表达式的结尾加上分毫，就变成语句，而语句不回返回值
  };
}
```
:::

#### 具有返回值的函数

函数可以向调用他的代码返回值。可以不对返回值命名，但要在箭头(->)后声明类型

在`Rust`中，函数的返回值等同于函数题最后一个表达式的值，使用`return`关键字和指定值，可以从函数中提前返回；

但大部分函数隐式的返回最后的表达式

```rust
fn five() -> i32 {
  5
}
```

## 1.4 控制流

根据条件是否为真决定是否执行某些代码，以及根据条件是否为真重复运行一段代码的能力是大部分编程语言的基本组成部分。

Rust中最常见的用来控制执行流的结构是if表达式和循环

### if表达式

if表达式允许根据条件执行不同的代码分支

::: tip

```rust
fn main() {
  let number = 3;

  if number > 5 {
    println!("condition is true");
  } else if number > 3 {
    println!("condition is true 1");
  } else {
    println!("condition is false");
  }
}
```

- 条件必须是`bool`，`Rust`不会自动将非布尔值转换为布尔值，必须总是显式地使用布尔值作为if的条件

- 在let语句中使用if

```rust
fn main() {
  let condition = true;
  let number1 = if condition { 10 } else { 9 };
}
```

⚠️注意

- number1被赋值整个if表达式，一位置if的每个分支的可能的返回值都必须是相同类型，因为变量只有一个类型
:::

### 使用循环重复执行

Rust有三种循环：loop、while、for


```rust
fn main() {
  let mut loop_count = 0;
  let result = loop {
    loop_count += 1;

    println!("again");

    if loop_count == 3 {
      break loop_count * 2;
    }
  }
  println!("result = {result}");
}
```

### 循环标签：在多个循环之间消除歧义

如果存在嵌套循环，`break` 和 `continue` 应用于此时最内层的循环。可以选择在一个循环上指定一个 `循环标签（loop label）`，然后将标签与 `break` 或 `continue` 一起使用，使这些关键字应用于已标记的循环而不是最内层的循环。

```rust
fn main() {
  let mut count = 0;
  'counting_up': loop {
    let mut remaining = 10;

    loop {
      if remaining == 9 {
        break;
      }

      if count == 2 {
        break 'counting_up';
      }
      remaining -= 1;
    }
    count += 1;
  }
}
```

### while 条件循环

```rust
fn main() {
  let mut number = 3;

  while number != 0 {
    number -= 1;
  }
}
```

### 使用 for 遍历集合

```rust
fn main() {
  let people = ["Linda", "Bob", "Menda"];

  for element in people {
    pringln!("name is {element}");
  }
}
```

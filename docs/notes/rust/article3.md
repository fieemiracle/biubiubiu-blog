---
title: 所有权
createTime: 2025/09/15 17:03:40
permalink: /rust/2t3mqsx4/
---

## 2.1 什么是所有权

`所有权（ownership）`是`Rust`用于管理内存的一组规则，所有程序都必须管理其运行时使用计算机内存的方式。

一些语言中具有`垃圾回收机制`，在程序运行时有规律地寻找不再使用的内存；

另一些语言中，`程序员必须亲自分配和释放内存`；

`Rust` 则选择了第三种方式：通过所有权系统管理内存，编译器在编译时会根据一系列的规则进行检查。如果违反了任何这些规则，程序都不能编译。在运行时，所有权系统的任何功能都不会减慢程序的运行。

::: important `栈（Stack）`与`堆（Heap）`

- 在`Rust`系统编程语言中，值位于栈上还是堆上在更大程度影响了语言的行为以及为何必须做出这样的抉择

- **栈**：栈以放入值的顺序并以相反顺序取出值，`先进后出（后进先出）`。栈中的所有数据都必须占用已知且固定的大小，在编译时大小未知或大小可能变化的数据更改为存储在堆上

- **堆**：堆是缺乏组织的，当向堆放入数据时，需要请求一定大小的空间。内存分配器在堆的某处找到一块足够大的空位，把它标记为已使用，并返回一个表示该位置地址的指针，这个过程就是`在堆上分配内存`

- 入栈比在堆上分配内存要快，因为（入栈时）分配器无需为存储新数据取搜索内存空间；其位置总是在栈顶；相比之下，在堆上分配内存则需要更多的工作，这是因为分配器必须首先找到一块足够存放数据的内存空间，并接着做一些记录为下一次分配作准备
  
- 访问堆上的数据比访问栈上的数据慢，因为必须通过指针来访问，现代处理器在内存中跳转越少就越快。

- 所有权的主要目的就是管理堆数据
:::

### 1、所有权规则

::: tip 所有权的规则
- `Rust` 中的每一个值都有一个 `所有者（owner）`。

- 值在任一时刻有且只有一个所有者。

- 当所有者离开作用域，这个值将被丢弃。
:::

### 2、变量作用域

作用域是一个`项（item）`在程序中有效的范围。

### 3、`String`类型

`String`类型管理被分配到堆上的数据，能够存储在编译时位置大小的文本，可以使用`from`函数基于字符串字面值来创建`String`

两个`::`是运算符，允许将特定的`from`函数置于`String`类型的`命名空间（namespace）`下，而不需要类似`string_from`这样的名字

```rust
fn main() {
  let mut str = String::from("Hello"); // String可变

  str.push_str(", Rust");

  let str1 = "Hello world!" // 字符串字面值，编译时就知道其内容
}
```

### 4、内存与分配

::: tip 字符串字面值&`String`类型在内存的处理上

（1）**字符串字面值**  编译时就知道其内容，文本被直接变吗进最终的可执行文件，快速且高效，不可变

（2）不能为每个在编译时大小未知的文本而将一块内存放入二进制文件中，并且它的大小还可能随着程序运行而改变

（3）**String类型**  为支持可变、可增长的文本片段，需要在堆上分配一块在编译时未知大小的内存来存放内容，意味着：

  - 必须在运行时内存分配器请求内存。例如：当调用 `String::from` 时，它的`实现 (implementation)` 请求其所需的内存

  - 需要一个当处理完`String`时将内存返回给分配器的方法，像Rust这样没有`垃圾回收(garbage collector, GC)`的语言中，
      我们需要识别出不再使用的内存并调用代码显式释放

  - 若忘记回收，会浪费内存；过早回收，会出现无效变量；重复回收，会出现二次释放bug

  - `Rust`采用：内存在拥有它的变量离开作用域后就被自动释放

```rust
fn main() {
  let s = String::from("Hello"); // 从此处起，s是有效的

  println!("{s}"); // 使用s
} // 此作用域已结束，自动调用drop函数释放s的内存
// s不再有效
```

:::


### 5、使用移动的变量与数据交互

::: caution

```rust
// 示例1
let x = 10;
let y = x; // 在rust中，整数是有已知固定大小的简单值，所以有两个变量，x和y，值都为5

// 示例2
let s1 = String::from("hello");
let s2 = s1; // 在rust中，此处不叫拷贝，而是移动，此时访问s1会报错，s1不再有效

```

- **String类型**  如果示例2中被认为是拷贝，则会出现一下问题
- （1）`String`类型包括三个部分，栈、长度和容量，如果是拷贝，意味着从栈上拷贝了它的指针、长度和容量，
      但是实际上并没有拷贝指针指向堆上的数据

- （2）若`Rust`也拷贝了堆上的数据，内存就是两份，那在堆上的数据比较大的时候会对运行时性能造成非常大的影响。
      这样当变量离开作用域后，`Rust` 会自动调用`drop`函数并清理变量的堆内存，两个数据指针指向了同一位置，
      当`s2`和`s1`离开作用域，都会尝试释放相同的内存，
      这个叫`二次释放`的错误（`两次释放相同内存会导致内存污染，可能会导致潜在的安全漏洞`）

- （3）实际上，当`let s2 = s1;`操作之后，Rust认为`s1`不再有效，所以不需要在s1离开作用域清理任何东西

- （4）不同于其他语言的`浅拷贝`和`深拷贝`，虽然拷贝指针、长度和容量而不拷贝数据可能听起来像浅拷贝。
      但是因为 `Rust` 同时使第一个变量无效，这个操作被称为 `移动（move）`，而不是叫做浅拷贝

```rust
fn main() {
   let mut s = String::from("hello");
  s = String::from("ahoy");
  println!("{s}, world!");
}

```
- **作用域与值**  `作用域`、`所有权`和通过`drop`函数释放内存之间的关系反过来也同样成立，
    当给一个已存在的变量赋一个全新的值时，`Rust` 会立即调用`drop`并释放原始值的内存
:::

### 6、使用克隆的变量与数据交互

当需要深度复制`String`中堆上的数据，而不仅仅是栈上的数据，需要使用`clone`方法

```rust
fn main() {
  let s1 = String::from("Hello WORLD!");

  let s2 = s1.clone();

  println!("{s1}, {s2}");
}
```

Rust有个叫`Copy trait`的特殊注解，可以用在类似整型这样的存储在栈上的类型上，不允许自身或其任何部分实现了`Drop trait`的类型使用`Copy trait`

如果对其离开作用域时需要特殊处理的类型使用`Copy`注解，会出现一个编译时错误

任何不需要分配内存或某种形式资源的类型都可以实现`Copy`，如下是一些 Copy 的类型：

（1）`所有整数类型`

（2）`布尔类型`

（3）`所有浮点数类型`

（4）`字符类型`

（5）`元组`，当且仅当其包含的类型也都实现`Copy`的时候，比如`let tup = (i32, String);`没有实现`Copy`


### 7、所有权与函数

将值传递给函数与给变量赋值的原理相似。向函数传递值可能会移动或者复制，就像赋值语句一样。

::: tip

```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效
    println!("{}", x);   // 尝试使用s，会抛出一个编译时错误

    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，
    println!("{}", x);              // 所以在后面可继续使用 x

} // 这里，x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 没有特殊之处

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{some_string}");
} // 这里，some_string 移出作用域并调用 `drop` 方法。
  // 占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{some_integer}");
} // 这里，some_integer 移出作用域。没有特殊之处
```

:::

### 8、返回值与作用域

::: important 返回值也可以转移所有权

```rust
fn main() {
    let s1 = gives_ownership();        // gives_ownership 将它的返回值传递给 s1

    let s2 = String::from("hello");    // s2 进入作用域

    let s3 = takes_and_gives_back(s2); // s2 被传入 takes_and_gives_back, 
                                       // 它的返回值又传递给 s3
} // 此处，s3 移出作用域并被丢弃。s2 被 move，所以无事发生
  // s1 移出作用域并被丢弃

fn gives_ownership() -> String {       // gives_ownership 将会把返回值传入
                                       // 调用它的函数

    let some_string = String::from("yours"); // some_string 进入作用域

    some_string                        // 返回 some_string 并将其移至调用函数
}

// 该函数将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String {
    // a_string 进入作用域

    a_string  // 返回 a_string 并移出给调用的函数
}
```

:::

::: info 转移返回值的所有权

变量的所有权总是遵循相同的模式：将值赋给另一个变量时它会移动。当持有堆中数据值的变量离开作用域时，其值将通过 drop 被清理掉，除非数据被移动为另一个变量所有。

在每一个函数中都获取所有权并接着返回所有权有些啰嗦。如果我们想要函数使用一个值但不获取所有权该怎么办呢？如果我们还要接着使用它的话，每次都传进去再返回来就有点烦人了，除此之外，我们也可能想返回函数体中产生的一些数据。

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{s2}' is {len}.");
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 返回字符串的长度

    (s, length) // 需要返回值来交还所有权
}
```

:::

::: tip 返回参数的所有权

但是这未免有些形式主义，而且这种场景应该很常见。幸运的是，`Rust` 对此提供了一个不用获取所有权就可以使用值的功能，叫做 `引用（references）`。

:::

## 2.2 引用和借用

引用像一个指针，只是个地址，由此访问储存于该地址的属于其他变量的数据。与指针不同，引用在其声明周期内保证指向某个特定类型的有效值。

```rust
fn main() {
  let s1 = String::from("hello");

  let len = calculate_length(&s1); // 以一个对象的引用作为参数而不是获取值的所有权

}

fn calculate_length(s: &String) -> usize { // s 是 String 的引用
  s.len() // 不需要返回值交还所有权
} // 这里，s 离开了作用域。但因为它并不拥有引用值的所有权
// 所以什么也不会发生
```

创建一个引用的行为称为借用，并不拥有它的所有权

::: tip 变量默认是不可变的，引用也一样，默认不允许修改引用的值

```rust
fn main() {
  let s = String::from("hello");

  change(&s);
}

fn change(some_string: &String) {
  some_string.push_str(", world"); // 会报错
}
```
要允许修改一个借用的值，可以调整为可变引用（mutable reference）

```rust
fn main() {
  let mut s = String::from("hello");

  change(&mut s);
}

fn change(some_string: &mut String) {
  some_string.push_str(", world"); // 会报错
}
```

可变引用有一个很大的限制：如果已经有一个对该变量的可变引用，就不能再创建对该变量的引用

```rust
fn main() {
  let mut s = String::from("apple");

  let r1 = &mut s;
  let r2 = &mut s; // 无效，不允许在同一时间多次将s作为可变变量借用

  println!("{} {}", r1, r2);
}
```
这个限制可以在编译时避免数据竞争。数据竞争类似于竞态条件，可由三个行为造成：

（1）两个或更多指针同时访问同一数据

（2）至少一个指针被用来写入数据

（3）没有同步数据访问的机制

数据竞争会导致未定义行为，难以在运行时追踪，并且难以诊断和修复；Rust通过拒绝编译存在数据竞争的代码来避免此问题

可以使用大括号创建一个新的作用域，以拥有多个可变引用，只是不能同时拥有：
```rust
fn main() {
  let mut s = String::from("hello");

  {
    let r2 = &mut s;
  } // r2在这里离开了作用域

  let r1 = &mut s;

}
```

同时使用可变和不可变引用时也强制采用类似的规则：
```rust
fn main() {
  let mut s = String::from("hello");

  let r1 = &s; // 没问题
  let r2 = &s; // 没问题
  let r3 = &mut s; // 大问题

  println!("{}, {}, and {}", r1, r2, r3);
}
```

注意一个引用的作用域从声明的地方开始一直持续到最后一次使用为止。
```rust
fn main() {
  let mut s = String::from("hello");

  let r1 = &s; // 没问题
  let r2 = &s; // 没问题

  println!("{r1} and {r2}");
  // 此位置之后 r1 和 r2 不再使用

  let r3 = &mut s; // 没问题

   println!("{r3}");
}
```
:::

### 1、悬垂引用（Dangling References）

在具有指针的语言中，很容易通过释放内存时保留指向它的指针而错误地生成一个`悬垂指针（dangling pointer）`—— 指向可能已被分配给其他用途的内存位置的指针。相比之下，在 `Rust` 中编译器确保引用永远也不会变成悬垂引用：当拥有一些数据的引用，编译器确保数据不会在其引用之前离开作用域。

如下创建一个悬垂引用，看看 `Rust` 如何通过通过一个编译时错误来防止它：

```rust
fn main() {
  let reference_to_nothing = dangle();
}

fn dangle() -> &String { // dangle 返回一个字符串的引用   会报错❌
  let s = String::from("reference");

  &s // 返回字符串 s 的引用
} // 这里 s 离开作用域并被丢弃。其内存被释放。意味着这个引用会指向一个无效的String
// 危险！
```
正确的写法如下：
```rust
fn main() {
  let reference_to_nothing = dangle();
}

fn dangle() -> String {
  let s = String::from("reference");

  s
}
```

### 2、引用的规则

（1）在任意给定时间，要么只能有一个可变引用，要么只能有多个不可变引用

（2）引用必须总是有效

## 2.3 `Slice`类型

`切片（slice）`允许引用集合中一段连续的元素序列，而不用引用整个集合。slice是一种音乐，它不拥有所有权。

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes(); // 将 String 转化为字节数组

    // iter 方法在字节数组上创建一个迭代器；enumerate 包装了 iter 的结果
    // enumerate 返回的元组中，第一个元素是索引，第二个元素是集合中元素的引用
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}

fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word 的值为 5

    s.clear(); // 这清空了字符串，使其等于 ""

    println!("word = {word}"); // 虽然word仍然是5 但是不得不时刻担心word的索引与s中的数据不再同步

    // word 在此处的值仍然是 5，
    // 但是没有更多的字符串让我们可以有效地应用数值 5。word 的值现在完全无效！
}

```

`字符串slice`是`String`中的部分值引用：
```rust
let s = String::from("Hello world");

let hello = &s[0..5]; // 或写成  &s[..5]
let world = &s[6..11]; // 或写成  &s[6..]
let total = &s[..];
```

::: important 字符串字面值就是slice

```rust
let s = "hello world!";
```
`s`的类型就是`&str`：是一个指向二进制程序特定位置的`slice`，`&str`是一个不可变引用
:::


::: info `字符串slice`作为参数

```rust
fn first_word(s: &str) -> &str {}
```
:::

::: tip 其他类型的`slice`

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);

```
`slice`的类型就是`&[i32]`：是一个指向二进制程序特定位置的`slice`，`&str`是一个不可变引用
:::


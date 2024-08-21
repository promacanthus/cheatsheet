---
title: Rust Cheat Sheet
description: Rust Cheat Sheet
layout: default
---

* TOC
{:toc}

Rust 速查手册（Rust Cheatsheet）是一个非常有用的工具，可以帮助开发者快速回顾 Rust 的核心概念和语法。以下是一个简单的 Rust 速查手册大纲，涵盖了一些常见的 Rust 特性：

## 介绍

### 注释

```rust
// 单行注释

/* 
 * 多行注释
 */

/// 文档注释
```

### 格式化输出

`std::fmt` 中定义的宏，用于格式化和打印 Strings 的实用工具。

#### `format!`

```rust
format!("Hello");                 // => "Hello"
// 使用 Display 格式，`{}` 以优雅且有好的方式格式化文本。
format!("Hello, {}!", "world");   // => "Hello, world!"
format!("The number is {}", 1);   // => "The number is 1"
// 使用 Debug 格式，`:?` 指示符会以调试格式输出值。
format!("{:?}", (3, 4));          // => "(3, 4)"
// 使用命名参数格式化，在格式字符串中使用变量名，然后在参数列表中提供对应的值。
format!("{value}", value=4);      // => "4"

let people = "Rustaceans";
format!("Hello {people}!");       // => "Hello Rustaceans!"
format!("{} {}", 1, 2);           // => "1 2"
// 使用宽度和填充选项。`:04` 表示输出应该至少有 4 个字符宽，不足的部分用 0 填充。
format!("{:04}", 42);             // => 带前导零的 "0042"
// 使用美化的 Debug 格式。`:#?` 指示符会以更易读的多行格式输出复杂类型。
format!("{:#?}", (100, 200));     // => "(
                                  // 100,
                                  //       200, )"
                                  //
```

#### `write!`

`write!` 和 `writeln!` 是两个宏，用于将格式字符串发送到指定的流。这用于防止格式字符串的中间分配，而是直接写入输出。 在底层，这个函数实际上是调用在 `std::io::Write` 和 `std::fmt::Write` trait 上定义的 `write_fmt` 函数。

```rust
#![allow(unused)]
#![allow(unused_must_use)]
fn main() {
    use std::io::Write;
    let mut w = Vec::new();
    write!(&mut w, "Hello {}!", "world");
}
```

#### `print!`

`print!` 和 `println!` 是两个宏，用于将格式字符串发送到 `stdout`。与 `write!` 宏类似，这些宏的目标是避免在打印输出时进行中间分配。

```rust
print!("Hello {}!", "world");
println!("I have a newline {}", "character at the end");
```

`eprint!` 和 `eprintln!` 宏分别与 `print!` 和 `println!` 相同，只不过它们将其输出发送到 `stderr`。

```rust
eprint!("{}", x); 
eprintln!("{}", x);
```

#### `format_args!`

`format_args!` 是一个奇怪的宏，用于安全地传递描述格式字符串的不透明对象。该对象不需要创建任何堆分配，并且仅引用栈上的信息。 在幕后，所有相关的宏都在此方面实现。

```rust
#![allow(unused)]
#![allow(unused_must_use)]
fn main() {
    use std::fmt;
    use std::io::{self, Write};

    let mut some_writer = io::stdout();
    write!(
        &mut some_writer,
        "{}",
        format_args!("print with a {}", "macro")
    );

    fn my_fmt_fn(args: fmt::Arguments<'_>) {
        write!(&mut io::stdout(), "{args}");
    }
    my_fmt_fn(format_args!(", or a {} too", "function"));
}

// output: print with a macro, or a function too
```

`format_args!` 宏的结果是 `fmt::Arguments` 类型的值。 然后可以将此结构体传递到此模块内部的 `write` 和 `format` 函数，以处理格式字符串。该宏的目的是在处理格式化字符串时甚至进一步防止中间分配。

#### Debug

使用 `#[derive(Debug)]` 属性可以使类型自动实现 `Debug` trait。

```rust
#[derive(Debug)]
struct DebugPrintable(i32);

fn main() {
    let v = DebugPrintable(3);
    println!("{:?}", v);
}

// output: DebugPrintable(3)
```

所有 `std` 库类型都天生可以使用 `{:?}` 来打印。

#### Display

`fmt::Debug` 通常看起来不太简洁，因此自定义输出的外观经常是更可取的。这需要通过手动实现 `fmt::Display` 来做到。

```rust
use std::fmt;

#[derive(Debug)]
struct List(Vec<i32>);

impl fmt::Display for List {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        let vet = &self.0;
        write!(f, "[")?;

        for (count, v) in vet.iter().enumerate() {
            if count != 0 {
                write!(f, ", ")?;
            }
            write!(f, "{}", v)?;
        }

        write!(f, "]")
    }
}

fn main() {
    let v = List(vec![1, 2, 3]);
    println!("{}", v);
}


// output: [1, 2, 3]
```

> 在 Rust 中，通过 `&self.0` 来获取元组结构体或新类型结构体中第一个字段的引用是很常见的模式。这种方式允许我们访问封装在**自定义类型**中的内部数据，同时保持了 Rust 的**所有权**和**借用规则**。这种模式在实现自定义类型的特征(如 `Display` 特征)时特别有用，因为它让我们能够以一种安全和高效的方式操作内部数据。

## 数据类型

### 原生类型

变量都能够显式地给出**类型说明**（type annotation）。数字还可以通过**后缀**（suffix）或**默认方式**来声明类型。

* 整型默认为 `i32` 类型，
* 浮点型默认为 `f64` 类型。

| 类型分类 | 子类型     | 具体类型                                                                     |
| -------- | ---------- | ---------------------------------------------------------------------------- |
| 标量     | 有符号整数 | `i8`、`i16`、`i32`、`i64`、`i128` 和 `isize`（**指针宽度**）                 |
| 标量     | 无符号整数 | `u8`、`u16`、`u32`、`u64`、`u128` 和 `usize`（**指针宽度**）                 |
| 标量     | 浮点数     | `f32`、`f64`                                                                 |
| 标量     | 字符       | `char`（单个 `Unicode` 字符，如 `'a'`，`'α'` 和 `'∞'`，每个都是 **4** 字节） |
| 标量     | 布尔型     | `bool`（只能是 `true` 或 `false`）                                           |
| 标量     | 单元类型   | `()`（其唯一可能的值就是 `()` 这个空元组）                                   |
| 复合     | 元组       | 如 `(1, true)`                                                               |
| 复合     | 数组       | 如 `[1, 2, 3]`                                                               |

原生类型之间可以通过 `as` 关键字进行显式类型转换。

```rust
fn main() {
    let decimal = 65.4321_f32;
    let integer = decimal as u64;
    let character = integer as u8;
    println!("Casting: {} -> {} -> {}", decimal, integer, character);
}
```

### 字面量

整数 `1`、浮点数 `1.2`、字符 `'a'`、字符串 `"abc"`、布尔值 `true` 和单元类型 `()` 可以用数字、文字或符号之类的 “**字面量**”（literal）来表示。

另外，通过加前缀 `0x`、`0o`、`0b`，数字可以用十六进制、八进制或二进制记法表示。

为了改善可读性，可以在数值字面量中插入下划线，比如：`1_000` 等同于 `1000`，`0.000_001` 等同于 `0.000001`。

### 元组

元组是一个可以包含各种类型值的组合。元组使用括号 `()` 来构造（construct），而每个元组自身又是一个类型标记为 `(T1, T2, ...)` 的值，其中 `T1`、`T2` 是每个元素的类型。

```rust
// 元组可以充当函数的参数和返回值
fn reverse(pair: (i32, bool)) -> (bool, i32) {
    // 通过元组的下标来访问具体的值
    println!("first value: {}",pair.0);
    println!("second value: {}",pair.1);

    // 创建单元素元组需要一个额外的逗号，这是为了和被括号包含的字面量作区分。
    println!("one element tuple: {:?}", (5u32,));
    println!("just an integer: {:?}", (5u32));

    // 元组可以被解构（deconstruct），从而将值绑定给变量
    let (integer, boolean) = pair;
    (boolean, integer)
}
```

### 数组和切片

数组（array）是一组拥有相同类型 `T` 的对象的集合，在内存中是连续存储的。数组使用中括号 `[]` 来创建，且它们的**大小在编译时会被确定**。数组的类型标记为 `[T; length]`（`T` 为元素类型，`length` 表示数组大小）。

```rust
fn main() {
    // 定长数组（类型标记是多余的）
    let xs: [i32; 5] = [1, 2, 3, 4, 5];
    // 所有元素可以初始化成相同的值
    let ys: [i32; 500] = [0; 500];
    // 数组是在栈中分配的
    println!("array occupies {} bytes", mem::size_of_val(&xs));
}
```

切片（slice）类型和数组类似，但其**大小在编译时是不确定**。

切片是一个双字对象（two-word object），

* 第一个字是一个指向数据的指针，
* 第二个字是切片的长度。(“字” 的宽度和 `usize` 相同。)

slice 可以用来借用数组的一部分,类型标记为 `&[T]`。

```rust
use std::mem;

fn main() {
    let xs: [i32; 5] = [1, 2, 3, 4, 5];
    // 数组可以自动被借用成为 slice
    analyze_slice(&xs);
    // slice 可以指向数组的一部分
    analyze_slice(&xs[1..4]);
}

// 此函数借用一个 slice
fn analyze_slice(slice: &[i32]) {
    println!("first element of the slice: {}", slice[0]);
    println!("the slice has {} elements", slice.len());
}
```

### 类型转换

#### From/Into

Rust 使用 `trait` 解决类型之间的转换问题。最一般的转换会用到 `From` 和 `Into` 两个 `trait`。

> 不过，即便常见的情况也可能会用到特别的 `trait`，尤其是从 `String` 转换到别的类型，以及把别的类型转换到 `String` 时。

`From` trait 允许一种类型定义 “怎么根据另一种类型生成自己”，因此它提供了一种类型转换的简单机制。

`Into` trait 就是把 `From` trait 倒过来，为自定义类型实现 `From` 的同时，就自动获得了 `Intro`。**使用 `Into` trait 通常要求指明要转换到的类型，因为编译器大多数时候不能推断它**。

```rust
use std::convert::From;

#[derive(Debug)]
struct Number {
    value: i32,
}

impl From<i32> for Number {
    fn from(value: i32) -> Self {
        Self { value }
    }
}

fn main() {
    let my_str = "hello";
    let _my_string = String::from(my_str);

    // 自定义类型实现 From trait。
    let num = Number::from(30);
    println!("My number is {:?}", num);

    let int = 5;
    // Number 这个类型声明是必须的。
    let num: Number = int.into();
    println!("My number is {:?}", num);
}
```

#### TryFrom/TryInto

`TryFrom` 和 `TryInto` 是类型转换的**通用** trait。不同于 `From/Into` ，`TryFrom/TryInto` 用于易出错的转换，因此其返回值是 `Result` 型。

```rust
use std::convert::TryFrom;
use std::convert::TryInto;

#[derive(Debug, PartialEq)]
struct EvenNumber(i32);

impl TryFrom<i32> for EvenNumber {
    type Error = String;

    fn try_from(value: i32) -> std::result::Result<Self, Self::Error> {
        if value % 2 == 0 {
            Ok(EvenNumber(value))
        } else {
            Err(String::from("The number is not even!"))
        }
    }
}

fn main() {
    // TryFrom
    assert_eq!(EvenNumber::try_from(8), Ok(EvenNumber(8)));
    assert_eq!(
        EvenNumber::try_from(5),
        Err("The number is not even!".to_string())
    );

    // TryInto
    let result: Result<EvenNumber, String> = 8i32.try_into();
    assert_eq!(result, Ok(EvenNumber(8)));
    let result: Result<EvenNumber, String> = 5i32.try_into();
    assert_eq!(result, Err("The number is not even!".to_string()));
}
```

#### ToString

要把任何类型转换成 `String`，只需要实现那个类型的 `ToString` trait，通常不直接这么做，应该实现 `fmt::Display` trait，它会自动提供 `ToString`。

```rust
use std::{fmt, string::ToString};

struct Circle {
    radius: f64,
}

impl fmt::Display for Circle {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Circle of radius {}", self.radius)
    }
}

// 这是直接实现 std::string::ToString 这个 trait。
// impl ToString for Circle {
//     fn to_string(&self) -> String {
//         format!("Circle of radius {}", self.radius)
//     }
// }

fn main() {
    let circle = Circle { radius: 6.0 };
    println!("{}", circle.to_string());
}
```

#### FromStr

只要对目标类型实现了 `std::str::FromStr` trait，就可以用 `parse()` 把字符串转换成目标类型。

```rust
fn main() {
    // 使用类型注解，声明待转换的类型
    let parsed: i32 = "5".parse().unwrap();
    // 使用 <>，声明待转换的类型
    let turbo_parsed = "5".parse::<i32>().unwrap();
    println!("Sum: {}", parsed + turbo_parsed);
}
```

## 结构体和枚举

### struct

```rust
// 单元结构体
struct Unit;

// 元组结构体
struct Pair(i32, i32);

// 带有字段的常规结构体
struct Person {
    name: String,
    age: u8,
}

fn main() {
    // 初始化单元结构体
    let unit = Unit;

    // 初始化元组结构体
    let pair = Pair(1, 2);
    // 解构元组结构体
    let Pair(integer, decimal) = pair;
    println!("pair contains {:?} and {:?}", integer, decimal);
    // 访问元组结构体
    println!("pair contains {:?} and {:?}", pair.0, pair.1);

    // 初始化结构体
    let alice = Person {
        name: String::from("Alice"),
        age: 20,
    };
    // 访问结构体的字段
    println!("{} is {} years old.", alice.name, alice.age);
    // 使用结构体更新语法创建新结构体
    let bob = Person {
        name: String::from("Bob"),
        ..alice
    };
    println!("{} is {} years old.", bob.name, bob.age);
}
```

单元结构体的使用示例：

```rust
// 作为标记类型
struct Kilometers;
fn convert_to_miles(value: f64, _unit: Kilometers) -> f64 {
    value * 1000 as f64
}

// 实现 traits
trait Printable {
    fn print(&self);
}

struct Logger;

impl Printable for Logger {
    fn print(&self) {
        println!("Logging...");
    }
}

fn main() {
    let logger = Logger;
    logger.print();
}

// 零大小类型
use std::mem;

struct Empty;

fn main() {
    println!("Size of Empty: {} bytes", mem::size_of::<Empty>());
}
```

### enum

enum 关键字允许创建一个从数个不同取值中选其一的枚举类型（enumeration）。任何一个在 struct 中合法的取值在 enum 中也合法。

```rust
#![allow(dead_code)] // 屏蔽未使用代码的警告
enum WebEvent {
    PageLoad,                 // 单元结构体
    PageUnload,               // 单元结构体
    Paste(String),            // 元组结构体
    Click { x: i64, y: i64 }, // 常规结构体
}

fn inspect(event: WebEvent) {
    match event {
        WebEvent::PageLoad => println!("page loaded"),
        WebEvent::PageUnload => println!("page unloaded"),
        // 从 enum 中解构出值
        WebEvent::Paste(s) => println!("pasted {}", s),
        WebEvent::Click { x, y } => println!("clicked at {}, {}", x, y),
    }
}

// 类型别名
#[allow(non_camel_case_types)] // 屏蔽非驼峰命名的警告
type Event = WebEvent;

fn main() {
    let _load = Event::PageLoad;
}
```

可以用 `type` 语句给已有的类型取个新的名字。类型的名字必须遵循驼峰命名法（像是 CamelCase 这样），否则编译器将给出警告。

**别名的主要用途是避免写出冗长的模版化代码**。

### 常量

Rust 有两种常量，可以在任意作用域声明，包括全局作用域。它们都需要显式的类型声明：

* `const`：不可改变的值。
* `static`：具有 `'static` 生命周期的，是可变的变量（须使用 `static mut` 关键字）。

> 有个特例就是 "`string`" 字面量。它可以不经改动就被赋给一个 `static` 变量，因为它的类型标记：`&'static str` 就包含了所要求的生命周期 `'static`。其他的引用类型都必须特地声明，使之拥有 `'static` 生命周期。这两种引用类型的差异似乎也无关紧要，因为无论如何，`static` 变量都得显式地声明。

```rust
// 全局变量是在所有其他作用域之外声明的。
static LANGUAGE: &'static str = "Rust";
const THRESHOLD: i32 = 10;

fn main() {
    println!("LANGUAGE: {}", LANGUAGE);
    println!("THRESHOLD: {}", THRESHOLD);
}
```

## 变量

Rust 通过静态类型确保类型安全。变量绑定可以在声明时说明类型，使用 `let` 绑定操作可以将值（比如**字面量**）绑定（bind）到**变量**。

```rust
fn main() {
    let an_integer = 1u32;
    let a_boolean = true;
    let unit = ();

    // 将 `an_integer` 复制到 `copied_integer`。
    let copied_integer = an_integer;
    // 编译器会对未使用的变量绑定产生警告；可以给变量名加上下划线前缀来消除警告。
    let _unused_variable = 3u32;

    // 变量绑定默认是不可变的， `mut` 关键字使其可变。
    let mut mutable_binding = 1;

    // 先声明变量绑定
    let a_binding;
    // 再初始化，编译器禁止使用未经初始化的变量。
    a_binding = 2; 
}
```

变量绑定有一个作用域（scope），它被限定只在一个**代码块**（block）中生存（live）。 代码块是一个被 `{}` 包围的语句集合。另外也允许变量遮蔽（variable shadowing）。

### 冻结

当数据被相同的名称不变地绑定时，它还会冻结（freeze）。在不可变绑定超出作用域之前，无法修改已冻结的数据：

```rust
fn main() {
    let mut _mutable_integer = 7i32;

    {
        // 被不可变的 `_mutable_integer` 遮蔽
        let _mutable_integer = _mutable_integer;
    }

    // `_mutable_integer` 在这个作用域没有冻结
    _mutable_integer = 3;
}
```

## 控制流

### if/else

```rust
fn main() {
    let n = 5;
    if n < 0 {
        print!("{} is negative", n);
    } else if n > 0 {
        print!("{} is positive", n);
    } else {
        print!("{} is zero", n);
    }
}
```

### 循环

#### loop

`loop` 关键字来表示一个无限循环。

* 使用 `break` 语句在任何时候退出一个循环，
* 使用 `continue` 跳过循环体的剩余部分并开始下一轮循环。

```rust
#![allow(unreachable_code)]
fn main() {
    'outer: loop {
        println!("Entered the outer loop");
        'inner: loop {
            println!("Entered the inner loop");
            break; // 这只是中断内部的循环
            break 'outer; // 这会中断外层循环
        }
        println!("This point will never be reached");
    }
    println!("Exited the outer loop");
}
```

`loop` 有个用途是尝试一个操作直到成功为止。

若操作返回一个值，则可能需要将其传递给代码的其余部分：将该值放在 `break` 之后，它就会被 `loop` 表达式返回。

```rust
fn main() {
    let mut counter = 0;
    let result = loop {
        counter += 1;
        if counter == 10 {
            break counter * 2;
        }
    };
    assert_eq!(result, 20);
}
```

#### while

`while` 关键字可以用作当型循环（**当条件满足时循环**）。

```rust
fn main() {
    let mut n = 3;
    while n < 5 {
        n += 1;
    }
    assert_eq!(n, 5);
}
```

#### for

`for in` 结构可以遍历一个 `Iterator`（迭代器）。

将集合转变为迭代器，有如下三个函数，分别以不同方式返回结合中的数据。

* `into_iter()`：会消耗集合。在每次迭代中，集合中的数据本身会被提供。**一旦集合被消耗了，之后就无法再使用了**，因为它已经在循环中被 “移除”（move）了。
* `iter()`：在每次迭代中**借用**集合中的一个元素。这样集合本身不会被改变，**循环之后仍可以使用**。
* `iter_mut()`：**可变地**（mutably）借用集合中的每个元素，从而允许集合被就地修改。

如果没有特别指定，`for` 循环会对给出的集合应用 `into_iter()` 函数，把它转换成一个迭代器。

```rust
fn main() {
    // 使用区间标记 a..b（前闭后开），创建一个迭代器
    for i in 1..11 {
        println!("{}", i)
    }

    // 使用区间标记 a..=b（前闭后闭），创建一个迭代器
    for i in 1..=10 {
        println!("{}", i)
    }

    let names = vec!["Bob", "Frank", "Ferris"];
    // 相当于默认使用 names.into_iter()
    for name in names {
        match name {
            "Ferris" => println!("There is a rustacean among us!")
            _=> println!("Hello {}", name)
        }
    }

    for name in names.iter() {
        match name {
            "Ferris" => println!("There is a rustacean among us!")
            _=> println!("Hello {}", name)
        }
    }

    for name in names.iter_mut() {
        *name = match name {
            &mut "Ferris" => println!("There is a rustacean among us!"),
            - => println!("Hello {}", name)
        }
        println!("names: {:?}", names);
    }
}
```

### match

#### 模式匹配

Rust 通过 `match` 关键字来提供**模式匹配**，和 `switch` 用法类似。第一个匹配的分支会被比对，并且**所有可能的值都必须被覆盖**。

```Rust
fn main() {
    let number = 13;
    match number {
        1 => println!("One"),                    // 匹配单个值
        2 | 3 | 5 | 7 | 11 => println!("Prime"), // 匹配多个值
        13..=19 => println!("A teen"),           // 匹配闭区间
        _ => println!("Ain't special"),          // 匹配任意值
    }

    let boolean = true;
    let binary = match boolean {
        false => 0,
        true => 1,
    };
    println!("{} -> {}", boolean, binary);
}
```

#### 解构匹配

`match` 代码块能以多种方式解构匹配，如元组、枚举、指针、引用等。

> 对指针来说，**解构**（destructure）和**解引用**（dereference）要区分开，因为这两者的概念是不同的。
>
> * 解引用使用 `*`
> * 解构使用 `&`、`ref`、和 `ref mut`

```rust
#[allow(dead_code)]
enum Color {
    Red,
    Blue,
    Green,
    RGB(u32, u32, u32),
    HSV(u32, u32, u32),
    HSL(u32, u32, u32),
    CMY(u32, u32, u32),
}

fn main() {
    // 解构元组
    let triple = (-1, 0, 1);
    match triple {
        // 解构出第二个和第三个元素
        (-1, y, z) => println!("First is `0`, `y` is {:?}, and `z` is {:?}", y, z),
        // .. 用于忽略元组其他部分
        (1, ..) => println!("First is `1`, and the rest doesn't matter"),
        _ => println!("It doesn't matter what values are in the tuple"),
    }

    // 解构枚举
    let color = Color::RGB(122, 17, 40);
    match color {
        Color::Red => println!("The color is Red!"),
        Color::RGB(r, g, b) => println!("Red {} Green {} Blue {}", r, g, b),
        _ => println!(),
    }

    let reference = &4;
    // 解构引用（注意：与解引用区分开）
    match reference {
        // 带上引用符号 & 去匹配
        &val => println!("Got a value via destructuring: {:?}", val),
    }
    // 解引用（注意：与解构引用区分开）
    match *reference {
        // 直接匹配值
        val => println!("Got a value via dereferencing: {:?}", val),
    }

    // 定义一个非引用类型的变量 `_not_reference` 的类型是 `i32`
    let _not_reference = 3;
    // 使用 `ref` 关键字，创建引用类型的变量 `_is_reference` 的类型是 `&i32`
    let ref _is_reference = 3;

    // 解构非引用类型
    let value = 5;
    match value {
        // 使用 `ref` 关键字，创建引用类型的变量 `r` 的类型是 `&i32`
        ref r => println!("Got a reference to a value: {:?}", r),
    }

    // 解构可变类型
    let mut value = 5;
    match value {
        // 使用 `ref mut` 关键字，创建可变引用类型的变量 `r` 的类型是 `&mut i32`
        ref mut m => {
            *m += 10;
            println!("We added 10. `value` is now {:?}", m)
        }
    }

    // 解构结构体
    struct Foo {
        x: (u32, u32),
        y: u32,
    }
    let foo = Foo { x: (1, 2), y: 3 };
    match foo {
        Foo { x: (1, b), y } => println!("First of x is 1, b = {},  y = {} ", b, y),
        _ => println!("Other"),
    }
    // 解构结构体并重命名变量，顺序不重要
    let Foo { y: i, x: j } = foo;
    println!("i = {}, j = {:?}", i, j);
    // 忽略结构体中的某些变量
    let Foo { y, .. } = foo;
    println!("y = {}", y);
}
```

#### 守卫

加上 `match` **守卫语句**（`guard`） 来过滤分支。

```rust
fn main() {
    let pair = (2, -2);
    match pair {
        (x, y) if x == y => println!("equal"),
        (x, _) if x % 2 == 1 => println!("The first one is odd"),
        _ => println!("No match"),
    }
}
```

#### 绑定

绑定是 Rust 中 `match` 表达式的一个强大特性。它允许我们在匹配模式的同时**捕获**和**命名**匹配的值。这在处理复杂数据结构或需要在匹配臂中使用匹配值时特别有用。

绑定使用 `@` 符号来实现。基本语法是 `name @ pattern`：

* `name` 是想给匹配值赋予的名称，
* `pattern` 是要匹配的模式。

```rust
fn age() -> u32 {
    15
}

fn some_number() -> Option<u32> {
    Some(10)
}

fn main() {
    match age() {
        0 => println!("not born yet"),
        // 在匹配区间的时候，使用 @ 来间接获取值
        n @ 1..=12 => println!("age {:?}", n),
        n => println!("age {:?}", n),
    }

    match some_number() {
        // 匹配 Some 中的值，并绑定到 n
        Some(n @ 42) => println!("number: {}", n),
        // 匹配 Some 中的任意值
        Some(n) => println!("number: {}", n),
        _ => println!("no match"),
    }
}
```

### if let

`if let` 是 Rust 中一种**简洁的模式匹配方式**，主要用于只关心**一种匹配情况**的场景。它通常用于替代 `match` 表达式，特别是当我们只需要匹配一个模式并忽略其他情况时。

`if let 模式 = 表达式 { 代码块 } else { 可选的 else 代码块 }`

```rust
enum Color {
    Bar,
    RGB(u8, u8, u8),
    HSV(u8, u8, u8),
}

fn main() {
    let some_value = Some(5);

    // 使用 match
    match some_value {
        Some(x) => println!("Got a value: {}", x),
        None => (),
    }

    // 使用 if let，更简洁，只关心一种匹配情况
    if let Some(x) = some_value {
        println!("Got a value: {}", x);
    } else {
        println!("Didn't get a value");
    }

    // 匹配任意枚举值
    let color = Color::RGB(0, 160, 255);
    if let Color::RGB(r, g, b) = color {
        println!("RGB color: {}, {}, {}", r, g, b);
    }

    let a = Color::Bar;
    if let Color::Bar = a {
        println!("Bar");
    }
}
```

> 另一个好处是：`if let` 允许匹配枚举非参数化的变量，即枚举未注明 `#[derive(PartialEq)]`，也没有为其实现 `PartialEq`。在这种情况下，通常 `if Foo::Bar == a` 会**出错**，因为此类枚举的实例不具有可比性。但是，`if let Foo::Bar = a` 是**可行**的。

### while let

`while let` 是 Rust 中的一种循环结构，它结合了 `while` 循环和**模式匹配**的特性。这个结构允许**在某个模式持续匹配时重复执行代码块**。

`while let 模式 = 表达式 { 代码块 }`

```rust
fn main() {
    let mut optional = Some(0);

    // 使用传统的 while 循环
    while optional.is_some() {
        let value = optional.unwrap();
        println!("got: {}", value);
        optional = if value < 5 { Some(value + 1) } else { None };
    }

    // 使用 while let 更简洁，避免显示 unwrap()，减少运行时错误的风险
    let mut optional = Some(0);
    while let Some(value) = optional {
        println!("got: {}", value);
        optional = if value < 5 { Some(value + 1) } else { None };
    }

    // 结合 mut 变量
    let mut values = vec![1, 2, 3, 4, 5];
    while let Some(mut value) = values.pop() {
        value *= 2;
        println!("Current value: {}", value);
    }
}
```

> 注意：
>
> * `while let` 没有可选的 `else/else if` 分支。
> * `while let` 不会自动实现 `break` 或 `continue`。如果需要这些控制流，需要显式地使用它们。
> * 如果模式匹配失败，循环会立即结束。确保你的逻辑考虑到了这一点。

## 函数

函数（function）使用 `fn` 关键字来声明。**函数的参数需要标注类型**，就和变量一样，如果函数返回一个值，返回类型必须在箭头 `->` 之后指定。

**函数最后的表达式将作为返回值**。也可以在函数内使用 `return` 语句来提前返一个值，甚至可以在循环或 `if` 内部使用。

* 一个 “不” 返回值的函数。实际上会返回一个单元类型 `()`。
* 当函数返回 `()` 时，函数签名可以省略返回类型

### 方法

方法（method）是依附于对象的函数。这些方法通过关键字 `self` 来访问对象中的数据和其他。方法在 `impl` 代码块中定义。

| 特征 | 静态方法 | 实例方法 |
|------|----------|----------|
| 定义方式 | 在 `impl` 块中，不以 `self` 作为第一个参数 | 在 `impl` 块中，第一个参数是 `self`、`&self` 或 `&mut self` |
| 调用语法 | 使用双冒号 `::` 通过类型名调用 | 使用点号 `.` 通过实例调用 |
| 示例调用 | `String::new()`, `Vec::<i32>::new()` | `some_string.len()`, `some_vec.push(1)` |
| 是否需要实例 | 不需要实例即可调用 | 需要实例才能调用 |
| 常见用途 | 构造函数、工厂方法、辅助函数 | 操作实例数据、实现对象行为 |
| `self` 关键字 | 不使用 `self` | 使用 `self`、`&self` 或 `&mut self` |
| 返回 `Self` | 可以返回 `Self` 表示当前类型 | 通常不返回 `Self`，但可以 |
| 访问实例数据 | 不能直接访问实例数据 | 可以访问实例数据 |

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // 静态方法（关联函数）
    fn new(width: u32, height: u32) -> Self {
        Self { width, height }
    }
    // 实例方法，`&self` 是 `self: &Self` 的语法糖
    fn area(&self) -> u32 {
        self.width * self.height
    }
    // 实例方法，`self` 是 `self: Self` 的语法糖
    fn compare(self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
    // 要求调用者是可变的，`&mut self` 为 `self: &mut Self` 的语法糖
    fn translate(&mut self, x: f64, y: f64) {
        self.p1.x += x;
        self.p2.x += x;

        self.p1.y += y;
        self.p2.y += y;
    }
}

fn main() {
    // 调用静态方法
    let rect = Rectangle::new(10, 20);
    // 调用实例方法
    println!("Area: {}", rect.area());

    let mut square = Rectangle {
        width: 1.0,
        height: 2.0,
    };
    square.translate(10.0, 20.0);
}
```

### 闭包

Rust 中的闭包（closure），也叫做 lambda 表达式是一类**能够捕获周围作用域中变量的函数**。

> 闭包表达式产生的类型就是 “**闭包类型**”，不属于引用类型。

| 特性 | 函数 | 闭包 |
|------|------|------|
| 语法 | 使用 `fn` 关键字定义 | 使用 `\|参数\| { 函数体 }` 形式 |
| 类型推断 | 需要显式声明参数和返回值类型 | 可以根据上下文推断类型 |
| 捕获环境 | 不能捕获定义环境中的变量 | 可以捕获定义环境中的变量 |
| 存储和传递 | 可以直接存储在变量中或作为参数传递 | 需要使用特定的 trait（`Fn`、`FnMut`、`FnOnce`） |
| 生命周期 | 没有特定的生命周期限制 | 受捕获变量的生命周期影响 |
| 定义位置 | 可以在全局作用域中定义 | 通常在局部作用域中定义 |
| 重载 | 可以重载（通过不同的函数名） | 不能重载 |
| 递归 | 可以直接递归调用自身 | 需要特殊处理才能实现递归，如使用 `Rc<RefCell<>>` 或 `Box<dyn Fn()>` |

```rust
// 函数
fn add_one(x: i32) -> i32 {
    x + 1
}

// 闭包
let add_one = |x: i32| -> i32 { x + 1 };
```

#### 捕获变量

闭包可以通过以下方式捕获变量，即实现了不通的 `trait`：

* 如果闭包只是读取变量，通过不可变引用：`&T` 捕获；即实现了 `Fn` trait：此闭包可以多次调用而**不改变**捕获的环境。
* 如果闭包需要修改变量，通过可变引用：`&mut T` 捕获；即实现了 `FnMut` trait：此闭包可以多次调用并**改变**捕获的环境。
* 如果闭包需要获取所有权或者变量类型不支持复制，通过值：`T` 捕获； 即实现了 `FnOnce` trait：此闭包理论上可以被多次调用，但是由于它会消耗捕获的值，所以通常只能被调用一次。

**闭包优先通过引用来捕获变量，并且仅在需要时使用其他方式。**

> 闭包的 `trait` 是层级关系：`Fn` 是 `FnMut` 的子 `trait`，`FnMut` 是 `FnOnc`e 的子 `trait`。这意味着：
>
> 实现了 `Fn` 的闭包也实现了 `FnMut` 和 `FnOnce`
> 实现了 `FnMut` 的闭包也实现了 `FnOnce`

```rust
fn main() {
    // 根据对捕获变量的不同操作，闭包实现的 trait 不同。

    // Fn()
    let color = String::from("green");
    let print = || println!("color: {}", color);
    print();

    // FnMut()
    let mut count = 0;
    let mut inc = || {
        count += 1;
        println!("count: {}", count)
    };
    inc();

    // FnOnce()
    let movable = Box::new(3);
    let consume = || {
        println!("`movable`: {:?}", movable);
        std::mem::drop(movable); // 将 movable 回收掉
    };
    consume();
}
```

在竖线 `|` 之前使用 `move` 会强制闭包取得**被捕获变量的所有权**：

```rust
let x = vec![1, 2, 3];
let equal_to_x = move |z: Vec<i32>| z == x;
```

#### 作为输入和输出参数

目前 Rust 只支持返回具体（非泛型）的类型。按照定义，匿名的闭包的类型是未知的，所以只有使用 `impl Trait` 才能返回一个闭包。还必须使用 `move` 关键字，表明所有的捕获都是通过值进行的。**这是必须的，因为在函数退出时，任何通过引用的捕获都被丢弃，在闭包中留下无效的引用。**

```rust
// 输入参数
fn apply<F>(f: F) -> i32
where
    F: FnOnce() -> i32,
{
    f()
}

// 输出参数
fn create_fn() -> impl Fn() {
    // str 不需要额外的内存分配，是一个引用，
    // 不涉及所有权转移，是不可变的，有轻微的性能优势
    let str: &'static str = "Fn";
    let text: String = "Fn".to_owned();
    move || println!("str: {}, text: {}", str, text)
}
```

#### 闭包中模式匹配

以下这三种写法都是正确的，但第二种（使用 `&x`）通常被认为是最简洁和惯用的。

重要的是要理解，这里的 `&` 在模式匹配中的作用与在其他上下文中的作用不同：

* 在表达式中，`&` 用于创建引用。
* 在模式中，`&` 用于匹配引用并解引用。
  
这种双重用途可能初看起来有点混淆，但它是 Rust 模式匹配系统的一个强大特性，允许我们以一种非常灵活和表达性强的方式处理数据结构。

```rust
fn main() {
    let number = vec![1, 2, 3];
    // 使用 &x（不解引用）
    numbers.iter().any(|x| *x % 2 == 0)
    // 使用 &x（解引用）
    numbers.iter().any(|&x| x % 2 == 0)
    // 不使用模式匹配（显式解引用）
    numbers.iter().any(|x| (*x) % 2 == 0)
}
```

Rust 中闭包参数的模式匹配和解引用，为什么要这样做？

* 便利性：这允许我们在闭包体内直接使用 `x` 作为一个 `i32`，而不需要每次都写 `*x`。
* 简洁性：它使代码更加简洁和易读。
* 灵活性：这种模式匹配允许我们根据需要选择是使用引用还是值。

### 高阶函数

Rust 中的高阶函数体现了**函数式编程**的特性。高阶函数是指可以接受一个或多个函数作为参数，和/或返回一个函数的函数。

优点：

* 代码复用：高阶函数允许抽象通用的操作模式。
* 灵活性：可以在运行时动态选择和组合行为。
* 表达力：能够简洁地表达复杂的操作。

注意事项：

* 性能：虽然 Rust 的零成本抽象通常能够优化高阶函数，但在某些情况下可能会有轻微的性能开- 销。
* 可读性：过度使用高阶函数可能会降低代码的可读性。
* 生命周期：当使用引用时，需要注意生命周期的问题。

```rust
// 自定义高阶函数
fn apply_twice<F>(f: F, x: i32) -> i32
where
    F: Fn(i32) -> i32,
{
    f(f(x))
}

fn create_adder(y: i32) -> impl Fn(i32) -> i32 {
    move |x| x + y
}

// 函数指针
fn call_with_one(f: fn(i32) -> i32) -> i32 {
    f(1)
}

fn add_one(x: i32) -> i32 {
    x + 1
}

fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    // 常见高阶函数 map、filter、fold
    let _squares: Vec<i32> = numbers.iter().map(|&x| x * x).collect();
    let _even_numbers: Vec<&i32> = numbers.iter().filter(|&&x| x % 2 == 0).collect();
    let _sum: i32 = numbers.iter().fold(0, |acc, &x| acc + x);

    let double = |x| x * 2;
    let result = apply_twice(double, 5);
    println!("{}", result); // 输出：20

    let adder = create_adder(2);
    let result = adder(3);
    println!("{}", result); // 输出：5

    let res = call_with_one(add_one);
}
```

| 特性 | 函数指针 | 闭包 |
|------|----------|------|
| 环境捕获 | 不能捕获环境 | 可以捕获定义环境中的变量 |
| 类型 | 具体类型，如 `fn(i32) -> i32` | 实现 `Fn`、`FnMut` 或 `FnOnce` trait |
| 语法 | 使用 `fn` 关键字定义 | 使用 `\|参数\| 表达式` 语法 |
| 大小 | 固定大小（通常是一个机器字） | 大小可变，取决于捕获的环境 |
| 使用场景 | 简单回调，不需要捕获环境 | 更灵活，适用于需要访问环境的场景 |
| Trait 实现 | 自动实现所有 `Fn`、`FnMut` 和 `FnOnce` trait | 根据对捕获变量的使用方式实现相应 trait |
| 分发方式 | 总是静态分发（编译时） | 可以静态分发（泛型）或动态分发（trait 对象） |
| 内存布局 | 指向代码的指针 | 包含捕获变量和函数指针的结构 |
| 泛型上下文 | 只能用于 `fn` 类型参数 | 可用于实现了 `Fn` 系列 trait 的类型参数 |
| 创建和存储 | 易于创建、存储和传递 | 每个闭包是唯一类型，存储和传递可能需要 trait 对象或泛型 |

#### 发散函数

发散函数是永不返回的函数。在 Rust 中，这种函数的返回类型被标记为 `!`，称为 **never type** 或 **empty type**。

> 类型系统中的作用： `!` 类型是所有类型的子类型，**这意味着它可以被强制转换为任何其他类型**。

特征：

* 永不返回到调用者
* 可以用在需要任何类型的地方
* 通常用于终止程序执行或进入无限循环

`fn diverging_function() -> ! { 函数体 }`

常见用例：

* 程序终止（如 `panic!` 宏）
* 持续运行的进程（如服务器主循环）
* 错误处理

```rust
#![allow(dead_code)]

fn exit() -> ! {
    std::process::exit(1);
}

fn infinite_loop() -> ! {
    loop {
        // 无限循环
    }
}

// 与 match 结合
fn match_number(n: i32) -> bool {
    match n {
        0 => false,
        1 => true,
        _ => panic!("Unexpected number"), // 这里使用了发散函数
    }
}

// 与 Option<T> 或 Result<T,E> 结合
fn process(data: Option<i32>) -> i32 {
    match data {
        Some(x) => x,
        None => panic!("No data"), // 发散函数用作返回值
    }
}
```

在异步编程中的应用： 在异步函数中，发散函数可以用来表示永不完成的 future。

注意事项：

* 编译器会检查发散函数是否真的不会返回
* 发散函数仍然可以包含 `return` 语句，但这些语句必须也是发散的

## 模块系统

### mod 关键字

模块可以分配到**文件**/**目录**的层次结构中。

```rust
// 此声明将会查找名为 `my.rs` 或 `my/mod.rs` 的文件，
// 并将该文件的内容放到此作用域中一个名为 `my` 的模块里面。
mod my;

// 类似地，`mod inaccessible` 和 `mod nested` 将找到 `nested.rs` 和
// `inaccessible.rs` 文件，并在它们放到各自的模块中。
mod inaccessible;
pub mod nested;
```

### use 语句

use 声明可以将一个完整的路径绑定到一个新的名字，从而更容易访问。

```rust
// 将 `deeply::nested::function` 路径绑定到 `other_function`。
use deeply::nested::function as other_function;

fn function() {
    println!("called `function()`");
}

mod deeply {
    pub mod nested {
        pub fn function() {
            println!("called `deeply::nested::function()`")
        }
    }
}

fn main() {
    // 更容易访问 `deeply::nested::function`
    other_function();
    println!("Entering block");
    {
        // 这和 `use deeply::nested::function as function` 等价。
        // 此 `function()` 将遮蔽外部的同名函数。
        use deeply::nested::function;
        function();
        // `use` 绑定拥有局部作用域。在这个例子中，`function()`
        // 的遮蔽只存在这个代码块中。
        println!("Leaving block");
    }
    function();
}

```

### pub 关键字

默认情况下，模块中的项拥有私有的可见性（private visibility），不过可以加上 `pub` 修饰语来重载这一行为。模块中只有公有的（public）项可以从模块外的作用域访问。

## 属性

* 当属性作用于整个 crate 时，它们的语法为 `#![crate_attribute]`，
* 当属性作用于模块或项时，语法为 `#[item_attribute]`（注意少了感叹号 !）。

属性可以接受参数，有不同的语法形式：

```rust
#[attribute = "value"]
#[attribute(key = "value")]
#[attribute(value,value2)]
```

### cfg

条件编译可能通过两种不同的操作符实现：

* cfg 属性：在属性位置中使用 `#[cfg(...)]`
* cfg! 宏：在布尔表达式中使用 `cfg!(...)`

两种形式使用的参数语法都相同。

```rust
// 这个函数仅当目标系统是 Linux 的时候才会编译
#[cfg(target_os = "linux")]
fn are_you_on_linux() {
    println!("You are running linux!")
}

// 而这个函数仅当目标系统 不是 Linux 时才会编译
#[cfg(not(target_os = "linux"))]
fn are_you_on_linux() {
    println!("You are *not* running linux!")
}

fn main() {
    are_you_on_linux();
    
    println!("Are you sure?");
    if cfg!(target_os = "linux") {
        println!("Yes. It's definitely linux!");
    } else {
        println!("Yes. It's definitely *not* linux!");
    }
}
```

## 泛型

泛型极大地减少了代码的重复，采用泛型意味着指定**泛型类型具体化**时，什么样的具体类型是合法的。泛型最简单和常用的用法是用于**类型参数**。

泛型的**类型参数**是使用**尖括号**和**大驼峰命名**的名称：`<Aaa, Bbb, ...>` 来指定的。泛型类型参数一般用 `<T>` 来表示。

```rust
// 泛型函数的定义
fn foo<T>(arg: T) { ... }

// 泛型函数的调用
fun::<A, B, ...>();

// 泛型方法
struct generic<T> {
    value: T,
}

impl<T> generic<T> {
    fn value(&self) -> &T {
        &self.value
    }
}

// 泛型特性
trait MyTrait<T> {
    fn foo(self);
}

impl<T, U> MyTrait<T> for U {
    fn foo(self) {}
}

fn main() {
    // 调用泛型方法
    let g = generic { value: 42 };
    println!("{}", g.value());
    // 调用泛型特性
    <i32 as MyTrait<i32>>::foo(42);
    <&str as MyTrait<&str>>::foo("hello");
}
```

在使用泛型时，类型参数常常必须使用 `trait` 作为约束（`bound`）来明确规定类型应实现哪些功能。

约束的工作机制会产生这样的效果：

* **即使一个 `trait` 不包含任何功能，你仍然可以用它作为约束**。标准库中的 `Eq` 和 `Ord` 就是这样的 trait。
* 如果有多重约束时，可以用 `+` 连接。和平常一样，类型之间使用 `,` 隔开。

```rust
fn printer<T: std::fmt::Display>(arg: T) {
    println!("{}", arg);
}

struct Example;
trait Empty {}
impl Empty for Example {}

fn demo<T: Empty>(arg: T) -> &'static str {
    "empty"
}

fn main() {
    let e = Example;
    demo::<Example>(e);

    let ee = "";
    // &str 没有实现 Empty trait 下一行会报错。
    // demo::<&str>(ee);
}
```

约束也可以使用 `where` 分句来表达，它放在 `{` 的前面，而不需写在类型第一次出现之前。另外 `where` 从句可以用于任意类型的限定，而不局限于类型参数本身。

where 在下面一些情况下很有用：

* 当分别指定泛型的类型和约束会更清晰时
* 当使用 where 从句比正常语法更有表现力时

```rust
use std::fmt::Debug;

trait PrintInOption {
    fn print_in_option(self);
}

// 这里需要一个 `where` 从句，否则就要表达成 `T: Debug`（这样意思就变了），
// 或者改用另一种间接的方法。
impl<T> PrintInOption for T where
    Option<T>: Debug {
    // 我们要将 `Option<T>: Debug` 作为约束，因为那是要打印的内容。
    // 否则我们会给出错误的约束。
    fn print_in_option(self) {
        println!("{:?}", Some(self));
    }
}

fn main() {
    let vec = vec![1, 2, 3];
    vec.print_in_option();
}
```

### 关联项

Rust 中的 `trait` 可以包含关联项，包括**关联类型**、**关联常量**和**关联函数**。这些关联项为 `trait` 提供了更强大和灵活的功能。

```rust
// 关联类型
trait Container {
    type Item; // 占位符，实现 trait 时指定具体类型
    fn add(&mut self, item: Self::Item);
    fn get(&self) -> Option<&Self::Item>;
}

// 关联常量
trait HasArea {
    const PI: f64 = 3.14159;
    fn area(&self) -> f64;
}

// 关联函数
trait Creatable {
    fn new() -> Self;
    fn with_value(value: i32) -> Self;
}
```

这些关联项可以在实现 `trait` 时被具体化或重写。它们提供了一种方式来定义与 `trait` 相关的类型、常量和函数，而不需要指定具体的实现细节。这增加了 `trait` 的抽象能力和复用性。

使用关联项可以使 `trait` 更加通用和灵活，允许不同的实现者根据自己的需求来定义具体的类型、常量或函数。

## 作用域规则

作用域在**所有权**（ownership）、**借用**（borrow）和**生命周期**（lifetime）中起着重要作用。

Rust 的变量不只是在栈中保存数据：它们也占有资源，比如 `Box<T>` 占有堆（heap）中的内存。Rust 强制实行 **RAII**（Resource Acquisition Is Initialization，资源获取即初始化），所以任何对象在离开作用域时，它的析构函数（destructor）就被调用，然后它占有的资源就被释放。

> Rust 中的析构函数概念是通过 `Drop` trait 提供的。当资源离开作用域，就调用析构函数。无需为每种类型都实现 `Drop` trait，只要为那些需要自己的析构函数逻辑的类型实现就可以了。

### 所有权规则

因为变量要负责释放它们拥有的资源，所以资源只能拥有一个所有者。这也防止了资源的重复释放。注意并非所有变量都拥有资源（例如**引用**）。

在进行赋值（`let x = y`）或通过值来传递函数参数（`foo(x)`）的时候，资源的所有权（ownership）会发生转移。按照 Rust 的说法，这被称为资源的**移动**（move）。

在移动资源之后，原来的所有者不能再被使用，这可避免悬挂指针（dangling pointer）的产生。

* 当所有权转移时，数据的可变性可能发生改变。
* 在单个变量的解构内，可以同时使用 **by-move** 和 **by-reference** 模式绑定。这样做将导致变量的**部分移动**（partial move），这意味着变量的某些部分将被移动，而其他部分将保留。**在这种情况下，后面不能整体使用父级变量，但是仍然可以使用只引用（而不移动）的部分。**

```rust
fn main() {
    // 所有权转移时，可变性发生变化
    let immutable_box = Box::new(5u32);
    let mut mutable_box = immutable_box;
    *mutable_box = 4;
    println!("{}", mutable_box);

    // 部分移动
    struct Person {
        name: String,
        age: u8,
    }

    let person = Person {
        name: String::from("Alice"),
        age: 30,
    };

    let Person { name, ref age } = person;
    println!("{}", person.age);
}
```

### 引用和借用

多数情况下，我们更希望**能访问数据**，**同时不取得其所有权**。为实现这点，Rust 使用了**借用**（borrowing）机制。对象可以通过**引用**（`&T`）来传递，从而取代通过值（`T`）来传递。

编译器（通过借用检查）静态地保证了引用总是指向有效的对象。也就是说，**当存在引用指向一个对象时，该对象不能被销毁**。

* `&mut T` 通过可变引用（mutable reference）来借用数据，使借用者可以读/写数据。
* `&T` 通过不可变引用（immutable reference）来借用数据，借用者可以读数据而不能更改数据。

> 数据可以**多次不可变借用**，但是在不可变借用的同时，原始数据不能使用可变借用。或者说，**同一时间内只允许一次可变借用**。仅当最后一次使用可变引用之后，原始数据才可以再次借用。

在通过 `let` 绑定来进行模式匹配或解构时，`ref` 关键字可用来创建**结构体**/**元组**的字段的引用。

```rust
fn main() {
    let var = "var";
    // _ref_var_1 和 _ref_var_2 等价
    let ref _ref_var_1 = var;
    let _ref_var_2 = &var;
}
```

### 生命周期

编译器（中的借用检查器）用生命周期来保证所有的借用都是有效的。**一个变量的生命周期在它创建的时候开始，在它销毁的时候结束**。

> 虽然生命周期和作用域经常被一起提到，但它们并不相同。
> 通过 `&` 来借用一个变量。该借用拥有一个**生命周期**，此生命周期由它声明的位置决定。只要该借用在出借者（lender）被销毁前结束，借用就是有效的。然而，借用的**作用域**则是由使用引用的位置决定的。

#### 结构体

```rust
// 显示标注
struct foo<'a>(&'a i32) // foo 的生命周期不超过 'a
struct bar<'a, 'b>{ &'a i32, &'b i32 } // bar 的生命周期不超过 'a 或 'b 中任意一个
```

#### 函数/方法

带上生命周期的**函数/方法**签名有一些限制：

* 任何引用都必须拥有标注好的生命周期。
* 任何被返回的引用都必须有和某个输入量相同的生命周期或是静态类型（static）。

```rust
// 参数 x 和 y 的有各自的生命周期，且'b 不能比 'a 活得更久
fn max<'a, 'b: 'a>(x: &'a i32, y: &'b i32) -> &'a i32 {
    match x > y {
        true => x,
        false => y,
    }
}
```

**注意，如果没有输入的函数返回引用，有时会导致返回的引用指向无效数据，这种情况下禁止它返回这样的引用。**

#### trait

```rust
trait Default {
    fn default() -> Self;
}

struct Borrowed<'a>(&'a i32);

impl<'a> Borrowed<'a> {
    fn new(x: &'a i32) -> Self {
        Borrowed(x)
    }
}

impl<'a> Default for Borrowed<'a> {
    fn default() -> Self {
        Borrowed(&0)
    }
}
```

#### 约束

就如泛型类型能够被约束一样，生命周期（它们本身就是泛型）也可以使用约束。`:` 字符的意义在这里稍微有些不同，不过 `+` 是相同的。注意下面的说明：

* `T: 'a`：在 `T` 中的所有引用都必须比生命周期 `'a` 活得更长。
* `T: Trait + 'a`：`T` 类型必须实现 `Trait` trait，并且在 `T` 中的所有引用都必须比 `'a` 活得更长。

```rust
// `Ref` 包含一个指向泛型类型 `T` 的引用，其中 `T` 拥有一个未知的生命周期 `'a`。
// `T` 拥有生命周期限制， `T` 中的任何引用都必须比 `'a` 活得更长。
// `Ref` 的生命周期也不能超出 `'a`。
struct Ref<'a, T: 'a>(&'a T);

// 接受一个指向 `T` 的引用，
// 其中 `T` 实现了 `Debug` trait，并且在 `T` 中的所有引用都必须比 `'a'` 存活时间更长。
// `'a` 要比函数活得更长。
fn print_ref<'a, T: 'a>(x: Ref<'a, T>)
where
    T: std::fmt::Debug + 'a,
{
    println!("{:?}", x.0);
}
```

#### 强制转换

一个**较长**的生命周期可以强制转成一个**较短**的生命周期，使它在一个通常情况下不能工作的作用域内也能正常工作。

* 强制转换可以由**编译器隐式地推导**并执行，
* 强制转换可以通过**声明不同的生命周期**的形式实现。

```rust
// `x` 和 `y` 有不同的生命周期，编译器推导并强制将它们转换为更短的生命周期。
fn multiply<'a>(x: &'a i32, y: &'a i32) -> i32 {
    x * y
}

// 'a 的生命周期至少和 'b 的生命周期一样长。这是生命周期约束的限制条件。
// 显式的声明将 'a 的生命周期强制转换为更短的 'b 生命周期。
fn choose_first<'a: 'b, 'b>(x: &'a i32, _y: &'b i32) -> &'b i32 {
    x
}
```

#### static

`'static` 生命周期是可能的生命周期中**最长**的，它会**在整个程序运行的时期中存在**。`'static` 生命周期也可被强制转换成一个更短的生命周期。有两种方式使变量拥有 `'static` 生命周期，它们都把数据保存在可执行文件的**只读内存区**：

* 使用 `static` 声明来产生常量（`constant`）。
* 产生一个拥有 `&'static str` 类型的 `string` 字面量。

```rust
// 创建一个生命周期是 static 的常量
static NUM: i32 = 18;

// 强制 static 的生命周期转换为 coerce_static() 入参的生命周期。
fn coerce_static<'a>(_: &'a i32) -> &'a i32 {
    &NUM
}
```

#### 省略

有些生命周期的模式**太常用了**，所以**借用检查器**将会**隐式**地添加它们以减少程序输入量和增强可读性。

这种隐式添加生命周期的过程称为**省略**（elision）。在 Rust 使用省略仅仅是因为**这些模式太普遍了**。

```rust
//  elided_input 和 annotated_input等价
fn elided_input(x: &i32) {
    println!("{}", x);
}

fn annotated_input<'a>(x: &'a i32) {
    println!("{}", x);
}

// elided_pass 和 annotated_pass 等价
fn elided_pass(x: &i32) -> &i32 {
    x
}

fn annotated_pass<'a>(x: &'a i32) -> &'a i32 {
    x
}
```

## 特征（`trait`）

`trait` 是对未知类型 `Self` 定义的方法集。该类型也可以访问同一个 `trait` 中定义的其他方法。对任何数据类型都可以实现 `trait`。

### 派生

通过 `#[derive]` 属性，编译器能够提供某些 `trait` 的**基本实现**。如果需要更复杂的行为，这些 `trait` 也可以**手动实现**，查看各个属性的标准库，了解手动实现的函数。

下面是可以**自动派生**的 `trait`：

* `[Eq](https://rustwiki.org/zh-CN/std/cmp/trait.Eq.html)`、`[PartialEq](https://rustwiki.org/zh-CN/std/cmp/trait.PartialEq.html)`、`[Ord](https://rustwiki.org/zh-CN/std/cmp/trait.Ord.html)`、`[PartialOrd](https://rustwiki.org/zh-CN/std/cmp/trait.PartialOrd.html)`：用于比较。
* `[Clone](https://rustwiki.org/zh-CN/std/clone/trait.Clone.html)`：用来从 `&T` 创建副本 `T`。
* `[Copy](https://rustwiki.org/zh-CN/core/marker/trait.Copy.html)`：使类型具有 “**复制**语义”（copy semantics）而非 “**移动**语义”（move semantics）。**当处理资源时，默认的行为是在赋值或函数调用的同时将它们转移。但是我们有时候也需要把资源复制一份。**
* `[Hash](https://rustwiki.org/zh-CN/std/hash/trait.Hash.html)`：从 `&T` 计算哈希值（hash）。
* `[Default](https://rustwiki.org/zh-CN/std/default/trait.Default.html)`：创建数据类型的一个空实例。
* `[Debug](https://rustwiki.org/zh-CN/std/fmt/trait.Debug.html)`：使用 `{:?}` formatter 来格式化一个值。

### 返回 `dyn Trait` / `impl Trait`

Rust 编译器需要知道**每个函数的返回类型需要多少空间**。这意味着所有函数都必须返回一个具体类型。与其他语言不同，不能直接返回 trait，因为其不同的实现将需要不同的内存量。

在这种情况下，我们需要使用 `Box<dyn Animal>`。这个函数返回一个包含一些 `Animal` 的 Box（只是对堆中某些内存的引用）。因为引用的大小是静态已知的，并且编译器可以保证引用指向已分配的堆内存，所以可以从函数中返回 `trait`。

> 每当在堆上分配内存时，Rust 都会尝试尽可能明确。因此，如果函数以这种方式返回指向堆的 `trait` 指针，则需要使用 `dyn` 关键字编写返回类型，例如 `Box<dyn Animal>`。

```rust
trait Animal {
    fn make_sound(&self) -> String;
}

struct Dog;
impl Animal for Dog {
    fn make_sound(&self) -> String {
        "Woof!".to_string()
    }
}

struct Cat;
impl Animal for Cat {
    fn make_sound(&self) -> String {
        "Meow!".to_string()
    }
}

fn get_animal(is_dog: bool) -> Box<dyn Animal> {
    if is_dog {
        Box::new(Dog)
    } else {
        Box::new(Cat)
    }
}
```

对于闭包这种场景，或者是没有显式定义 Trait 的情况下，需要作为函数的返回值，则使用 `impl Trait`。

```rust
fn generate_random_numbers() -> impl Iterator<Item = u32> {
    use rand::Rng;
    let mut rng = rand::thread_rng();
    std::iter::from_fn(move || Some(rng.gen_range(1..100)))
}

fn main() {
    let numbers = generate_random_numbers().take(5).collect::<Vec<_>>();
    println!("{:?}", numbers);
}
```

还可以使用 `impl Trait` 返回使用 `map` 或 `filter` 闭包的迭代器！这使得使用 `map` 和 `filter` 更容易。因为闭包类型没有名称，所以如果函数返回带闭包的迭代器，则无法写出显式的返回类型。

```rust
fn double_positives<'a>(numbers: &'a Vec<i32>) -> impl Iterator<Item = i32> + 'a {
    numbers.iter().filter(|x| x > &&0).map(|x| x * 2)
}
```

| 特性 | dyn Trait | impl Trait |
|------|-----------|------------|
| 语法 | `Box<dyn Trait>` | `impl Trait` |
| 类型擦除 | 完全类型擦除 | 部分类型擦除 |
| 运行时开销 | 有动态分发开销 | 无动态分发开销 |
| 内存分配 | 在堆上分配 | 通常在栈上 |
| 返回类型灵活性 | 可以在运行时返回不同类型 | 编译时确定具体类型 |
| 适用场景 | 需要运行时多态 | 静态分发，隐藏具体类型 |
| `trait` 对象安全要求 | 必须满足对象安全 | 无对象安全限制 |
| 代码可读性 | 对复杂类型可能更清晰 | 对简单情况更简洁 |
| 编译时类型检查 | 部分延迟到运行时 | 完全在编译时进行 |
| 性能 | 相对较低 | 通常更高 |

> `impl trait` 的限制主要在于它不能用于表示不同的具体类型。如果需要根据条件返回不同的具体类型，那么就需要使用 `trait` 对象（如 `Box<dyn Trait>`）而不是 `impl trait`。

### 运算符重载

在 Rust 中，很多[运算符](https://rustwiki.org/zh-CN/core/ops/#traits)可以通过 `trait` 来重载。这些运算符可以根据输入参数来完成不同的任务。

> Rust 中的**运算符是方法调用的语法糖**。
>
> 例如，`a + b` 中的 `+` 运算符会调用 `add` 方法（也就是 `a.add(b)`）。这个 `add` 方法是 `Add trait` 的一部分。因此，`+` 运算符可以被任何 `Add trait` 的实现者使用。

```rust
use std::ops::{Add, Sub};

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Self;

    fn add(self, other: Self) -> Self {
        Self {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

impl Sub for Point {
    type Output = Self;

    fn sub(self, other: Self) -> Self {
        Self {
            x: self.x - other.x,
            y: self.y - other.y,
        }
    }
}

assert_eq!(
    Point { x: 3, y: 3 },
    Point { x: 1, y: 0 } + Point { x: 2, y: 3 }
);
assert_eq!(
    Point { x: -1, y: -3 },
    Point { x: 1, y: 0 } - Point { x: 2, y: 3 }
);
```

### [Drop](https://rustwiki.org/zh-CN/std/ops/trait.Drop.html)

`Drop` trait 只有一个方法：`drop`，当不再需要某个值时，Rust 将对该值运行 “析构函数”。 不再需要值的最常见方法是离开作用域。

此析构函数由两个组件组成：

* 如果为此类型实现了特殊的 `Drop` trait，则对该值调用 `Drop::drop`。
* 自动生成的 `drop glue` 递归调用该值的所有字段的析构函数。

> 由于 Rust 自动调用所有包含字段的析构函数，因此在大多数情况下，无需实现 `Drop`。
> 但是在某些情况下它很有用，例如对于直接管理资源的类型。 该资源可能是**内存**，可能是**文件描述符**，可能是**网络套接字**。 一旦不再使用该类型的值，则应通过释放内存或关闭文件或套接字 “`clean up`” 资源。

```rust
struct HasDrop;
impl Drop for HasDrop {
    fn drop(&mut self) {
        println!("Dropping HasDrop!");
    }
}

struct HasTwoDrops {
    one: HasDrop,
    two: HasDrop,
}
impl Drop for HasTwoDrops {
    fn drop(&mut self) {
        println!("Dropping HasTwoDrops!");
    }
}

fn main() {
    let _x = HasTwoDrops {
        one: HasDrop,
        two: HasDrop,
    };
    println!("Running!");
}
// 输出：
// Running!
// Dropping HasTwoDrops!
// Dropping HasDrop!
// Dropping HasDrop!
```

### [Iterator](https://rustwiki.org/zh-CN/core/iter/trait.Iterator.html)

`Iterator` trait 用来对集合（collection）类型（比如数组）实现迭代器。

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

创建自己的迭代器涉及两个步骤：创建一个 `struct` 来保存迭代器的状态，然后为该 `struct` 实现 `Iterator`。

```rust
struct CountDown {
    count: u32,
}
impl Iterator for CountDown {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        if self.count == 0 {
            None
        } else {
            self.count -= 1;
            Some(self.count)
        }
    }
}

fn main() {
    let countdown = CountDown { count: 5 };
    for i in countdown {
        println!("{}", i);
    }
}
```

Rust 的 `for` 循环语法实际上是迭代器的语法糖。为方便起见，`for` 结构会使用 `.into_iter()` 方法将一些集合类型转换为迭代器。

```rust
let values = vec![1, 2, 3, 4, 5];

for x in values {
    println!("{x}");
}
```

### 父 trait

Rust 没有“继承”，但是可以将一个 `trait` 定义为另一个 `trait` 的超集（即父 `trait`）。

```rust
// Person 是 Student 的父 trait。
trait Person {
    fn name(&self) -> String;
}

// 实现 Student 需要同时也 impl Person。
trait Student: Person {
    fn university(&self) -> String;
}

trait Programmer {
    fn fav_language(&self) -> String;
}

// CompSciStudent 是 Programmer 和 Student 两者的子类。
// 实现 CompSciStudent 需要同时 impl 两个父 trait。
trait CompSciStudent: Programmer + Student {
    fn git_username(&self) -> String;
}

fn comp_sci_student_greeting(student: &dyn CompSciStudent) -> String {
    format!(
        "My name is {} and I attend {}. My favorite language is {}. My Git username is {}",
        student.name(),
        student.university(),
        student.fav_language(),
        student.git_username()
    )
}
```

### 消除重叠 trait

一个类型可以实现许多不同的 `trait`。如果两个 trait 都需要相同的名称怎么办？例如，许多 trait 可能拥有名为 `get()` 的方法。他们甚至可能有不同的返回类型！

* 由于每个 `trait` 实现都有自己的 `impl` 块，因此很清楚要实现哪个 `trait` 的 `get` 方法。
* 为了消除它们之间的歧义，必须使用完全限定语法（Fully Qualified Syntax）来进行方法调用。

```rust
trait UsernameWidget {
    fn get(&self) -> String;
}

trait AgeWidget {
    fn get(&self) -> u8;
}

struct Form {
    username: String,
    age: u8,
}

impl UsernameWidget for Form {
    fn get(&self) -> String {
        self.username.clone()
    }
}

impl AgeWidget for Form {
    fn get(&self) -> u8 {
        self.age
    }
}

fn main() {
    let form = Form {
        username: "rustacean".to_owned(),
        age: 28,
    };

    let username = <Form as UsernameWidget>::get(&form);
    assert_eq!("rustacean".to_owned(), username);
    let age = <Form as AgeWidget>::get(&form);
    assert_eq!(28, age);
}
```

## 宏

Rust 提供了一个强大的宏系统，可进行**元编程**（metaprogramming）。

* 宏看起来和函数很像，只不过名称末尾有一个感叹号`!`。
* 宏并不产生函数调用，而是展开成源码，并和程序的其余部分一起被编译。

> Rust 的宏会展开为**抽象语法树**（AST，abstract syntax tree），而不是像字符串预处理那样直接替换成代码，这样就**不会产生无法预料的优先权错误**。

为什么宏是有用的？

* **不写重复代码**（DRY，Don't repeat yourself.）。很多时候需要在一些地方针对不同 的类型实现类似的功能，这时常常可以使用宏来避免重复代码。

* **领域专用语言**（DSL，domain-specific language）。宏允许为特定的目的创造特定的语法。

* **可变接口**（variadic interface）。有时需要能够接受不定数目参数的接口，比如 `println!`，根据格式化字符串的不同，它需要接受任意多的参数。

### `macro_rules!`

宏是通过 `macro_rules!` 宏来创建的。

```rust
// 这是一个简单的宏，名为 `say_hello`。
macro_rules! say_hello {
    () => {
        // `()` 表示此宏不接受任何参数。
        println!("Hello!");
    };
}

fn main() {
    say_hello!() // 这个调用将会展开成 `println("Hello");`!
}
```

### 指示符

* `block`
* `expr` 用于表达式
* `ident` 用于变量名或函数名
* `item`
* `literal` 用于字面常量
* `pat` (模式 pattern)
* `path`
* `stmt` (语句 statement)
* `tt` (标记树 token tree)
* `ty` (类型 type)
* `vis` (可见性描述符)

宏的参数使用一个美元符号 `$` 作为前缀，并使用一个**指示符**（designator）来注明类型：

```rust
macro_rules! create_function {
    // 此宏接受一个 `ident` 指示符表示的参数，并创建一个名为 `$func_name` 的函数。
    // `ident` 指示符用于变量名或函数名
    ($func_name:ident) => {
        fn $func_name() {
            // `stringify!` 宏把 `ident` 转换成字符串。
            println!("You called {:?}()", stringify!($func_name))
        }
    };
}

macro_rules! print_result {
    // 此宏接受一个 `expr` 类型的表达式，并将它作为字符串，连同其结果一起
    // 打印出来。
    // `expr` 指示符表示表达式。
    ($expression:expr) => {
        // `stringify!` 把表达式*原样*转换成一个字符串。
        println!("{:?} = {:?}", stringify!($expression), $expression)
    };
}

fn main() {
    create_function!(foo);
    create_function!(bar);
    foo();
    bar();

    print_result!(1u32 + 1);
    // 代码块也是表达式！
    print_result!({
        let x = 1u32;
        x * x + 2 * x - 1
    });
}
```

### 重载

宏可以重载，从而接收不同的参数组合。在这方面，`macro_rules!` 的作用类似于匹配（`match`）代码块。

```rust
// 根据调用它的方式，`test!` 将以不同的方式来比较 `$left` 和 `$right`。
macro_rules! test {
    // 参数不需要使用逗号隔开。
    // 参数可以任意组合！
    ($left:expr; and $right:expr) => (
        println!("{:?} and {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left && $right)
    );
    // ^ 每个分支都必须以分号结束。
    ($left:expr; or $right:expr) => (
        println!("{:?} or {:?} is {:?}",
                 stringify!($left),
                 stringify!($right),
                 $left || $right)
    );
}

fn main() {
    test!(1i32 + 1 == 2i32; and 2i32 * 2 == 4i32);
    test!(true; or false);
}
```

### 重复

宏在**参数列表**中：

* 使用 `+` 来表示该参数可能出现**一次或多次**，
* 使用 `*` 来表示该参数可能出现**零次或多次**。

把模式`$(...),+` 包围起来，就可以匹配一个或多个用逗号隔开的表达式。

> 宏定义的最后一个分支可以不用分号作为结束。

```rust
// `find_min!` 将求出任意数量的参数的最小值。
macro_rules! find_min {
    // 基本情形：
    ($x:expr) => ($x);
    // `$x` 后面跟着至少一个 `$y,`
    ($x:expr, $($y:expr),+) => (
        // 对 `$x` 后面的 `$y` 们调用 `find_min!` 
        std::cmp::min($x, find_min!($($y),+))
    )
}

fn main() {
    println!("{}", find_min!(1u32));
    println!("{}", find_min!(1u32 + 2 , 2u32));
    println!("{}", find_min!(5u32, 2u32 * 3, 4u32));
}
```

## 错误处理

错误处理是处理可能发生的失败情况的过程，显式地处理错误可以避免程序的其他部分产生潜在的问题。

在 Rust 中，有多种处理错误的方式：

|方式|描述|示例|
|---|---|---|
|`panic!`|1、测试<br> 2、处理不可恢复的错误|1、原型开发中使用`unimplemented`<br> 2、在测试中是用`panic!`显式地失败|
|`Option`|1、值可选<br> 2、缺少值并非错误|1、测试或者原型开发时，可以 `unwrap` 然后 `expect`<br> 2、如寻找父目录时，`/` 和 `C:` 目录没有父目录，这并不是一个错误。|
|`Result`|错误可能发生且应由调用者处理|在测试或者原型开发时，可以 `unwrap` 然后 `expect`|

### `panic!`

```rust
// exit 是一个发散函数，永远不会返回的函数。
fn exit() -> ! {
    // 这会导致程序崩溃并终止执行。
    panic!("crash and burn");
}
```

> 发散函数的一些常见用例包括：程序终止（如这个例子），无限循环，系统级别的错误处理。
> 使用 `!` 作为返回类型可以**让编译器知道这个函数不会正常返回**，这在某些情况下可以帮助编译器进行优化或进行更严格的类型检查。

### `Option`

在标准库（[std](https://doc.rust-lang.org/std/option/enum.Option.html)）中有个叫做 `Option<T>`的枚举类型，用于有 “不存在” 的可能性的情况。它表现为以下两个 “option”（选项）中的一个：

```rust
pub enum Option<T> {
    None,   // 找不到相应的元素
    Some(T),// 找到一个属于 T 类型的元素
}
```

处理 Option 的方式：

1. 使用 `match` 显式地处理，
2. 使用 `unwrap` 隐式地处理；要么返回 `Some` 内部的元素，要么 `panic`。

> 也可以手动使用 `expect` 方法自定义 `panic` 信息，但相比显式处理，`unwrap` 的输出仍显得不太有意义。

```rust
fn main() {
    let some_value: Option<i32> = Some(42);
    let none_value: Option<i32> = None;

    // 使用 match
    match some_value {
        Some(x) => println!("match: Value is {}", x),
        None => println!("match: No value"),
    }

    // 使用 unwrap
    let unwrapped = some_value.unwrap();
    println!("unwrap: Value is {}", unwrapped);

    // 使用 expect
    let expected = some_value.expect("Expected a value");
    println!("expect: Value is {}", expected);

    // 注意：对 None 使用 unwrap 或 expect 会导致 panic
    // none_value.unwrap(); // 这会 panic
    // none_value.expect("This will panic"); // 这也会 panic，但有自定义错误信息
}
```

**因此，对于 `Option`，只有在原型开发或者测试用例中才会是用 `unwrap` 和 `except`。**

访问 `Option` 变量的方式：

1. `match`
2. `if let`：
3. `?`：如果 `Option` 是 `Some`，返回 `Some` 内部的元素，否则无论函数是否正在执行都将终止且返回 `None`
4. 组合算子 `map`：`Option` 有一个内置方法 `map()`，可用于 `Some -> Some` 和 `None -> None` 这样的简单映射。多个不同的 `map()` 调用可以串起来，这样更加灵活。
5. `unwarp_or`
6. `and_then`: 也称为 flatmap，使用被 `Option` 包裹的值来调用其输入函数并返回结果。 如果 `Option` 是 `None`，那么它返回 `None`。

```rust
struct User {
    id: u32,
    name: String,
    age: Option<u32>,
}

fn is_adult(age: u32) -> Option<u32> {
    if age >= 18 {
        Some(age)
    } else {
        None
    }
}

fn recommend_activity(age: u32) -> Option<String> {
    match age {
        18..=25 => Some(String::from("Join our young adults program!")),
        26..=35 => Some(String::from("Try our professional networking events!")),
        36..=55 => Some(String::from("Consider our family-friendly activities!")),
        56.. => Some(String::from("Check out our seniors' social club!")),
        _ => None,
    }
}

fn print_user_info(user: &User) {
    // 使用 match 来处理 Option
    match user.age {
        Some(age) => println!("Age: {} years old", age),
        None => println!("Age: Not provided"),
    }

    // 使用 if let 来处理 Option
    if let Some(age) = user.age {
        if age >= 18 {
            println!("This user is an adult.");
        } else {
            println!("This user is a minor.");
        }
    }

    // 使用 ? 运算符来提取 age 值
    let age = user.age?;

    // 使用 map 方法来转换 Option 中的值
    let age_next_year = user.age.map(|age| age + 1);
    println!("Age next year: {:?}", age_next_year);

    // 使用 unwrap_or 来提供默认值
    let can_vote = user.age.unwrap_or(0) >= 18;
    println!("Can vote: {}", can_vote);

    // 使用 and_then 来链接多个操作:
    // 1. 首先检查 user.age 是否存在。
    // 2. 如果存在，则调用 is_adult 函数。
    // 3. 如果 is_adult 返回 Some，则继续调用 recommend_activity。
    // 4. 只有当所有步骤都成功（返回 Some）时，activity_recommendation 才会是 Some，否则就是 None。
    let activity_recommendation = user.age.and_then(is_adult).and_then(recommend_activity);

    match activity_recommendation {
        Some(activity) => println!("Recommended activity: {}", activity),
        None => println!("No activity recommendation available."),
    }
}
```

### [`Result<T, E>`](https://rustwiki.org/zh-CN/std/result/enum.Result.html)

`Result` 是 `Option` 类型的更丰富的版本，**描述的是可能的错误而不是可能的不存在**。

`Result<T，E>` 可以有两个结果的其中一个：

* `Ok<T>`：找到 `T` 元素
* `Err<E>`：找到 `E` 元素，`E` 即表示错误的类型。

按照约定，**预期结果**是 “`Ok`”，而**意外结果**是 “`Err`”。

> `Result` 有很多类似 `Option` 的方法。
> 在处理 Result 时，`?` 几乎就等于一个会返回 `Err` 而不是 `panic` 的 `unwrap`。

```rust
#[derive(Debug)]
enum UserError {
    InvalidUsername,
    InvalidPassword,
    UserCreationFailed,
}

// 别名
type UserResult<T> = Result<T, UserError>;

fn validate_username(username: &str) -> UserResult<String> {
    if username.len() >= 3 {
        Ok(username.to_string())
    } else {
        Err(UserError::InvalidUsername)
    }
}

fn validate_password(password: &str) -> UserResult<String> {
    if password.len() >= 8 {
        Ok(password.to_string())
    } else {
        Err(UserError::InvalidPassword)
    }
}

fn create_user(username: String, password: String) -> UserResult<User> {
    if username == "admin" {
        Err(UserError::UserCreationFailed)
    } else {
        Ok(User { username, password })
    }
}

// 使用 map 和 and_then
fn register_user_map(username: &str, password: &str) -> UserResult<User> {
    validate_username(username)
        .and_then(|valid_username| {
            validate_password(password).map(|valid_password| (valid_username, valid_password))
        })
        .and_then(|(username, password)| create_user(username, password))
}

// 使用类型别名和模式匹配和问号
fn register_user_alias(username: &str, password: &str) -> UserResult<User> {
    let valid_username = validate_username(username)?;
    let valid_password = validate_password(password)?;
    create_user(valid_username, valid_password)
}

// 使用提前返回
fn register_user_early_return(username: &str, password: &str) -> UserResult<User> {
    let valid_username = match validate_username(username) {
        Ok(username) => username,
        Err(e) => return Err(e),
    };

    let valid_password = match validate_password(password) {
        Ok(password) => password,
        Err(e) => return Err(e),
    };

    create_user(valid_username, valid_password)
}
```

## 标准库

| 类别 | 扩充类型 |
|------|----------|
| 字符串类型 | - `String`：可增长的、堆分配的 UTF-8 编码字符串<br>- `str`：不可变的 UTF-8 编码字符串切片 |
| 集合类型 | - `Vec<T>`：可增长的数组<br>- `VecDeque<T>`：双端队列<br>- `LinkedList<T>`：双向链表<br>- `HashMap<K, V>`：哈希表<br>- `BTreeMap<K, V>`：有序映射<br>- `HashSet<T>`：哈希集合<br>- `BTreeSet<T>`：有序集合 |
| 智能指针 | - `Box<T>`：堆分配的值<br>- `Rc<T>`：引用计数指针<br>- `Arc<T>`：原子引用计数指针<br>- `Cell<T>` 和 `RefCell<T>`：内部可变性 |
| 同步类型 | - `Mutex<T>`：互斥锁<br>- `RwLock<T>`：读写锁<br>- `Condvar`：条件变量<br>- `Once`：一次性初始化<br>- `Barrier`：同步屏障 |
| 错误处理 | - `Option<T>`：可选值<br>- `Result<T, E>`：可能的错误结果 |
| 迭代器相关 | - `Iterator` trait<br>- `IntoIterator` trait |
| 其他常用类型 | - `Path` 和 `PathBuf`：文件系统路径<br>- `OsString` 和 `OsStr`：操作系统字符串<br>- `CString` 和 `CStr`：C 兼容字符串 |
| 包装类型 | - `Wrapping<T>`：包装算术运算以处理溢出<br>- `Reverse<T>`：反转比较顺序的包装器 |
| 固定大小的数组类型 | - `[T; N]`：编译时固定大小的数组 |
| 切片类型 | - `&[T]` 和 `&mut [T]`：数组或 Vec 的视图 |

### 字符串类型

Rust 中有两种字符串类型：`String` 和 `&str`。

* `String` 被存储为由字节组成的 vector（`Vec<u8>`），但保证了它一定是一个有效的 UTF-8 序列。`String` 是堆分配的，可增长的，且不是零结尾的（null terminated）。
* `&str` 是一个总是指向有效 UTF-8 序列的切片（`&[u8]`），并可用来查看 `String` 的内容，就如同 `&[T]` 是 `Vec<T>` 的全部或部分引用。

### 集合类型

#### `HashMap`

`HashMap`（散列表）通过键（key）来存储值。键可以是:

* 布尔型、整型、字符串
* 任意实现了 `Eq` 和 `Hash` trait 的类型，加上`#[derive(PartialEq, Eq, Hash)]`

> 对于所有的集合类（collection class），如果它们包含的类型都分别实现了 `Eq` 和 `Hash`，那么这些集合类也就实现了 `Eq` 和 `Hash`。
>
> 注意：
> `f32` 和 `f64` 没有实现 `Hash`，**由于若使用浮点数作为散列表的键，浮点精度误差会很容易导致错误**。

`HashMap` 也是可增长的，但在占据了多余空间时还可以缩小自己。

* 使用 `HashMap::with_capacity(unit)` 创建具有一定初始容量的 `HashMap`
* 使用 `HashMap::new()` 来获得一个带有默认初始容量的 `HashMap`（**推荐**）

```rust
use std::collections::HashMap;

fn main() {
    // 创建一个新的 HashMap
    let mut fruit_prices = HashMap::new();

    // 添加键值对到 HashMap
    fruit_prices.insert(String::from("Apple"), 2);
    fruit_prices.insert(String::from("Banana"), 1);
    fruit_prices.insert(String::from("Orange"), 3);

    // 获取特定键的值
    match fruit_prices.get("Apple") {
        Some(&price) => println!("Price of Apple: ${}", price),
        None => println!("Apple not found"),
    }

    // 检查键是否存在
    println!("Do we have Mango? {}", fruit_prices.contains_key("Mango"));

    // 更新值
    *fruit_prices.entry(String::from("Banana")).or_insert(0) = 2;
    println!("Updated price of Banana: ${}", fruit_prices["Banana"]);

    // 删除键值对
    fruit_prices.remove("Orange");

    // 遍历 HashMap
    println!("All fruits and their prices:");
    for (fruit, price) in &fruit_prices {
        println!("{}: ${}", fruit, price);
    }

    // HashMap 的大小
    println!("Total number of fruits: {}", fruit_prices.len());
}
```

#### `HashSet`

`HashSet<T>` 实际上只是对 `HashMap<T, ()>` 的封装，**只关心其中的键而非值**。

> `HashSet` 的独特之处在于，它保证了**不会出现重复的元素**。这是任何 set 集合类型（set collection）遵循的规定。

集合（set）拥有 4 种基本操作（**下面的调用全部都返回一个迭代器**）：

* `union`（并集）：获得两个集合中的所有元素（不含重复值）。
* `difference`（差集）：获取属于第一个集合而不属于第二集合的所有元素。
* `intersection`（交集）：获取同时属于两个集合的所有元素。
* `symmetric_difference`（对称差）：获取所有只属于其中一个集合，而不同时属于两个集合的所有元素。

```rust
use std::collections::HashSet;

fn main() {
    let mut a: HashSet<i32> = vec![1i32, 2, 3].into_iter().collect();
    let mut b: HashSet<i32> = vec![2i32, 3, 4].into_iter().collect();

    println!("A: {:?}", a);
    println!("B: {:?}", b);

    // 乱序打印 [1, 2, 3, 4, 5]。
    println!("Union: {:?}", a.union(&b).collect::<Vec<&i32>>());

    // 这将会打印出 [1]
    println!("Difference: {:?}", a.difference(&b).collect::<Vec<&i32>>());

    // 乱序打印 [2, 3, 4]。
    println!(
        "Intersection: {:?}",
        a.intersection(&b).collect::<Vec<&i32>>()
    );

    // 打印 [1, 5]
    println!(
        "Symmetric Difference: {:?}",
        a.symmetric_difference(&b).collect::<Vec<&i32>>()
    );
}
```

### 智能指针

#### `Box<T>`

在 Rust 中，所有值默认都是**栈分配的**。通过创建 `Box<T>`，可以把值装箱（boxed）来使它**在堆上分配**。箱子（box，即 `Box<T>` 类型的实例）是一个智能指针，指向堆分配的 `T` 类型的值。

当箱子离开作用域时，它的析构函数会被调用，内部的对象会被销毁，堆上分配的内存也会被释放。

被装箱的值可以使用 `*` 运算符进行解引用；这会移除掉一层装箱。

```rust
#![allow(dead_code)]

use std::mem;

#[derive(Debug, Clone, Copy)]
struct Point {
    x: f64,
    y: f64,
}

impl Point {
    fn new(x: f64, y: f64) -> Point {
        Point { x: x, y: y }
    }
    fn origin() -> Point {
        Point { x: 0.0, y: 0.0 }
    }
    fn boxed_origin() -> Box<Point> {
        Box::new(Point { x: 0.0, y: 0.0 })
    }
}

fn main() {
    let point_stack = Point::origin();
    let point_heap = Point::boxed_origin();
    let boxed_point_heap = Box::new(Point::boxed_origin());

    println!(
        "Point occupies {} bytes in stack",
        mem::size_of_val(&point_stack)
    );
    // Box 的宽度就是指针的宽度
    println!(
        "Point occupies {} bytes in heap",
        mem::size_of_val(&point_heap)
    );
    println!(
        "Boxed Point occupies {} bytes in heap",
        mem::size_of_val(&boxed_point_heap)
    );

    // 使用 * 解引用，移除一层装箱
    let unboxed_point_heap = *boxed_point_heap;
    println!(
        "Unboxed Point occupies {} bytes in heap",
        mem::size_of_val(&unboxed_point_heap)
    );

    // output:
    // Point occupies 16 bytes in stack
    // Point occupies 8 bytes in heap
    // Boxed Point occupies 8 bytes in heap
    // Unboxed Point occupies 8 bytes in heap
}
```

#### `Rc<T>`

当需要多个所有权时，可以使用 `Rc`（引用计数，Reference Counting 缩写）。`Rc` 跟踪引用的数量，这相当于包裹在 `Rc` 值的所有者的数量.

> * 当克隆一个 `Rc` 时，Rc 的引用计数就会增加 1，
> * 当克隆得到的 `Rc` 退出作用域时，引用计数就会减少 1。
> * 当 `Rc` 的引用计数变为 0 时，这意味着已经没有所有者，`Rc` 和值两者都将被删除。

**克隆 `Rc` 从不执行深拷贝。克隆只创建另一个指向包裹值的指针，并增加计数。**

```rust
use std::rc::Rc;

fn main() {
    let rc_examples = "Rc examples".to_string();
    {
        println!("--- rc_a is created ---");
        // `rc_examples` 已经移入 `rc_a`
        let rc_a: Rc<String> = Rc::new(rc_examples);
        // Reference Count of rc_a: 1
        println!("Reference Count of rc_a: {}", Rc::strong_count(&rc_a));
        {
            println!("--- rc_a is cloned to rc_b ---");
            let rc_b: Rc<String> = Rc::clone(&rc_a);
            // Reference Count of rc_b: 2
            println!("Reference Count of rc_b: {}", Rc::strong_count(&rc_b));
            // Reference Count of rc_a: 2            
            println!("Reference Count of rc_a: {}", Rc::strong_count(&rc_a));
            // rc_a and rc_b are equal: true
            println!("rc_a and rc_b are equal: {}", rc_a.eq(&rc_b));
            // Length of the value inside rc_a: 11
            println!("Length of the value inside rc_a: {}", rc_a.len());
            // Value of rc_b: Rc examples
            println!("Value of rc_b: {}", rc_b);
            println!("--- rc_b is dropped out of scope ---");
        }
        // Reference Count of rc_a: 1
        println!("Reference Count of rc_a: {}", Rc::strong_count(&rc_a));
        println!("--- rc_a is dropped out of scope ---");
    }
    // 当 `rc_a` 被删时，`rc_examples` 也被一起删除。
}
```

### `Arc<T>`

当线程之间所有权需要共享时，可以使用`Arc`（共享引用计数，Atomic Reference Counted 缩写）可以使用。

* 通过 `Clone`，为堆内存中的值的位置，创建一个**引用指针**，同时增加**引用计数器**。
* 在线程之间共享所有权，因此当指向某个值的最后一个引用指针退出作用域时，该变量将被删除。

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let apple = Arc::new("the same apple");
    for _ in 0..10 {
        let apple = Arc::clone(&apple);
        thread::spawn(move || {
            println!("{:?}", apple);
        });
    }
    thread::sleep(std::time::Duration::from_secs(1));
}
```

## 其他常用标准库

### 路径

`Path` 结构体代表了底层文件系统的文件路径，分为两种：

* `posix::Path`，针对类 UNIX 系统
* `windows::Path`，针对 Windows

`prelude` 会选择并输出符合平台类型的 `Path` 种类。

> `prelude` 是 Rust 自动地在每个程序中导入的一些通用的东西，这样就不必每写一个程序就手动导入一番。

```rust
use std::path::Path;

fn main() {
    let path = Path::new(".");
    let display = path.display();
    println!("Display: {}", display);
    let new_path = path.join("a").join("b");
    match new_path.to_str() {
        None => panic!("new path is not a valid UTF-8 sequence"),
        Some(s) => println!("new path is {}", s),
    }
}
```

注意 `Path` 在内部并不是用 UTF-8 字符串表示的，而是存储为若干字节（`Vec<u8>`）的 vector。因此，将 Path 转化成 `&str` 并非零开销的（free），且可能失败（因此它返回一个 `Option`）。

### 文件输入输出

`File` 结构体表示一个**被打开的文件**（它包裹了一个文件描述符），并赋予了对所表示的文件的读写能力。

由于在进行文件 I/O（输入/输出）操作时可能出现各种错误，因此 `File` 的所有方法都返回 `io::Result<T>` 类型，它是 `Result<T, io::Error>` 的别名。

这使得所有 I/O 操作的失败都变成显式的。借助这点，可以看到所有的失败路径，并被鼓励主动地处理这些情形。

```rust
use std::fs::File;
use std::io::{self, BufRead, Read, Write};
use std::path::Path;

static LOREM_IPSUM: &'static str =
    "Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.";

static FILE_PATH: &'static str = "out/lorem_ipsum.txt";
fn main() {
    println!("Write file content by create.");
    let input = Path::new(FILE_PATH);
    let display = input.display();

    // create 静态方法以只写模式（write-only mode）打开一个文件。
    // 若文件已经存在，则旧内容将被销毁。否则，将创建一个新文件。
    let mut file = match File::create(&input) {
        Err(why) => panic!("couldn't create {}: {}", display, why),
        Ok(file) => file,
    };

    match file.write_all(LOREM_IPSUM.as_bytes()) {
        Err(why) => panic!("couldn't write to {}: {}", display, why),
        Ok(_) => println!("successfully wrote to {}", display),
    }

    println!("\nRead file content by open.");
    let output = Path::new(FILE_PATH);
    let display = output.display();
    // open 静态方法以只读模式（read-only mode）打开一个文件。
    let mut file = match std::fs::File::open(&output) {
        Err(why) => panic!("couldn't open {}: {}", display, why),
        Ok(file) => file,
    };

    let mut s = String::new();
    match file.read_to_string(&mut s) {
        Err(why) => panic!("couldn't read {}: {}", display, why),
        Ok(_) => println!("{} contains:\n{}", display, s),
    }

    println!("\nRead file content by BufRead.");
    if let Ok(lines) = read_lines(output) {
        for line in lines {
            if let Ok(ipsum) = line {
                println!("{}", ipsum);
            }
        }
    }
}

// read_lines 从文件中读取每一行，返回一个 `io::Result<io::Lines<io::BufReader<File>>>`
fn read_lines<T>(filename: T) -> io::Result<io::Lines<io::BufReader<File>>>
where
    T: AsRef<Path>,
{
    let file = File::open(filename)?;
    // lines 在文件行上返回一个迭代器。
    Ok(io::BufReader::new(file).lines())
}
```

### 程序参数

命令行参数可使用 `std::env::args` 进行接收，这将返回一个迭代器，该迭代器会对每个参数举出一个字符串。

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    // 第一个参数是调用本程序的路径
    println!("My path is {}.", args[0]);
    // 其余的参数是被传递给程序的命令行参数。
    // 请这样调用程序：
    //   $ ./args arg1 arg2
    println!("I got {:?} arguments: {:?}.", args.len() - 1, &args[1..]);
}
```

## 异步编程

### 线程

Rust 通过 `spawn` 函数提供了创建本地操作系统（native OS）线程的机制，该函数的参数是一个通过**值捕获变量的闭包**（moving closure）。

> 这些线程由操作系统调度（schedule）。

```rust
use std::thread;
static NTHREADS: i32 = 10;

fn main() {
    let mut children = vec![];
    for i in 0..NTHREADS {
        children.push(thread::spawn(move || println!("this is thread {}", i)))
    }

    for child in children {
        let _ = child.join();
    }
}
```

### 通道

Rust 为线程之间的通信提供了**异步**的通道（`channel`）。通道允许两个端点之间信息的单向流动：`Sender`（发送端） 和 `Receiver`（接收端）。

```rust
use std::sync::mpsc;
use std::thread;

static NTHREADS: usize = 3;

fn main() {
    let (tx, rx) = mpsc::channel();
    for id in 0..NTHREADS {
        // sender 端可以被复制。
        let thread_tx = tx.clone();
        thread::spawn(move || {
            // 新建的线程取得 thread_tx 的所有权。
            thread_tx.send(id).unwrap();
            println!("thread {} finished", id);
        });
    }

    let mut ids = Vec::with_capacity(NTHREADS as usize);
    for _ in 0..NTHREADS {
        // recv() 从通道中拿一个消息，若无消息则阻塞当前线程。
        ids.push(rx.recv());
    }
    println!("received ids: {:?}", ids);
}
```

### 子进程

* `process::Command` 结构体：表示是一个进程创建者（process builder）。
* `process::Output` 结构体：表示已结束的子进程的输出。

```rust
use std::process::Command;

fn main() {
    let output = Command::new("rustc")
        .arg("--version")
        .output()
        .unwrap_or_else(|e| panic!("failed to execute process: {}", e));

    if output.status.success() {
        let s = String::from_utf8_lossy(&output.stdout);
        println!("rustc successed and stdout was:\n{}", s);
    } else {
        let s = String::from_utf8_lossy(&output.stderr);
        println!("rustc failed and stderr was:\n{}", s);
    }
}
```

* `std::Child` 结构体：表示一个正在运行的子进程，它暴露了 `stdin`（标准输入），`stdout`（标准输出）和 `stderr`（标准错误）句柄，从而可以通过管道与所代表的进程交互。

```rust
use std::io::{Read, Write};
use std::process::{Command, Stdio};

static PANGRAM: &'static str = "the quick brown fox jumped over the lazy dog\n";

fn main() {
    let process: Child = match Command::new("wc")
        .stdin(Stdio::piped())
        .stdout(Stdio::piped())
        .spawn()
    {
        Err(why) => panic!("couldn't spawn wc: {}", why),
        Ok(process) => process,
    };

    match process.stdin.unwrap().write_all(PANGRAM.as_bytes()) {
        Err(why) => panic!("couldn't write to wc stdin: {}", why),
        Ok(_) => println!("sent pangram to wc"),
    }

    let mut s = String::new();
    match process.stdout.unwrap().read_to_string(&mut s) {
        Err(why) => panic!("couldn't read wc stdout: {:?}", why),
        Ok(_) => print!("wc responded with:\n{}", s),
    }
}
```

* 要等待一个 `process::Child` 完成，就必须调用 `Child::wait`，这会返回一个 `process::ExitStatus`。

```rust
use std::process::Command;

fn main() {
    let mut child = Command::new("sleep").arg("5").spawn().unwrap();
    match child.wait() {
        Ok(status) => {
            println!("child exited with: {}", status);
        }
        Err(why) => println!("error waiting for child: {}", why),
    }
    println!("reached end of main")
}
```

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

## 所有权和借用

所有权规则
引用和借用
生命周期

## 特征（Traits）

trait 定义
impl Trait for Type

## 错误处理

Result<T, E>
Option
panic!

## 泛型

函数中的泛型
结构体和枚举中的泛型

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

## 常用标准库功能

Vec, String 操作
文件 I/O
并发（线程）

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

## 宏

macro_rules!
常用宏: println!, vec!

## 异步编程

async/await
Future trait

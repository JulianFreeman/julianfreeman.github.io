---
title: Rust 学习笔记
date: 2023-12-02 23:39:00 -0500
categories: [学习笔记, Rust]
tags: [programming, rust]
description: The Rust Programming Language 的阅读笔记
---

## 1. 入门指南

### 1.1 安装

Rust 安装见 https://kaisery.github.io/trpl-zh-cn/ch01-01-installation.html 。

#### 更新和卸载

Rust 是可以更新的，不像 Python 那样每个版本都需要重新装。

```sh
rustup update
```
{: .nolineno}

卸载 Rust 用如下命令

```sh
rustup self uninstall
```
{: .nolineno}

查看离线文档

```sh
rustup doc
```
{: .nolineno}

### 1.2 Hello, World!

#### `rustc` 编译文件

创建一个文件 `main.rs`，编辑如下内容

```rust
fn main() {
    println!("Hello rust");
}
```

在命令行输入

```sh
rustc main.rs
```
{: .nolineno}

会生成一个可执行文件，运行该文件即可打印 `Hello rust`。

### 1.3 Hello, Cargo!

使用 `rustc` 来编译源码，可以，但有点繁琐。Cargo 是 Rust 的构建系统和包管理器，适合复杂项目。

#### 创建项目

```shell
cargo new hello_cargo --vcs=none
```
{: .nolineno}

该命令创建一个名为 `hello_cargo` 的文件夹，里面包含一些必要文件。默认情况下也会添加 git 版本控制，添加 `--vcs=none` 可以取消。

#### 构建项目

```shell
cd hello_cargo
cargo build
```
{: .nolineno}

编译项目，默认是 Debug 模式，会在 `target/debug` 下生成项目同名的可执行文件。

```shell
cargo run
```
{: .nolineno}

编译项目之后运行可执行文件，更方便，因此比 `cargo build` 更常用。

```shell
cargo check
```
{: .nolineno}

检查代码是否可编译但不实际编译项目，因此速度比 `cargo build` 更快。

#### 发布构建

```shell
cargo build --release
```
{: .nolineno}

在 `target/release` 下生成可用于发布的可执行文件。相比于 Debug 模式，编译时间可能更久（因为需要执行一些运行优化），但运行更快（因为有运行优化）。

## 3. 常见编程概念

### 3.1 变量与可变性

#### 变量

Rust 的变量用 `let` 声明，当变量类型可以被推断时，可以不用显式指明类型，当类型不可推断时，需要显式指明类型。

```rust
let a = 1; //可以推断是 i32 类型
let b: i32; //只声明未赋值，不能推断类型，需要指明
```

Rust 的变量默认都是不可变的，即赋值后便不可更改，要创建可变的变量需要用 `let mut` 声明。

```rust
let mut c = 3;
c += 2; //赋新值或者自增等都属于改变
```

#### 常量

Rust 的常量用 `const` 声明，永不可变，不能加 `mut` 修饰。

常量 **必须** 指明类型。

常量值必须在编译时就能确定，不能赋一个运行时才能知道的值，即只能是常量表达式。

```rust
const a: i32 = 3;
```

常量可以定义在全局作用域，变量只能定义在函数里。

```rust
const a: i32 = 3; //ok
let b = 3; //error

fn main() {
    let c = 4; //ok
}
```

#### 隐藏

Rust 可以用 `let` 多次声明同一变量，这一点与 JavaScript 不同。

```rust
let x = 3;
let x = x + 4;
```

这里第一行的 `x` 依然是不可变的，只不过第二行用 `let` 重新声明后这个 `x` 与第一行的 `x` 就是两个完全不同的变量了，只不过恰好名称相同。这一操作叫隐藏（Shadowing）。

因为是两个不同的变量了，所以类型啥的也没什么要求，下面的代码是可行的

```rust
let x = 3;
let x = "hello";
```

其中第一行的 `x` 的类型是 `i32`，而第二行的是 `&str`。两个变量除了名称相同外没什么联系。

但如下代码是不可行的

```rust
let mut x = 3;
x = "hello";
```

第一行声明了 `x` 这一变量为 `i32` 类型的，虽然是可变的，但这只代表其值可变，类型是不能变的，因此第二行试图赋值一个字符串时会失败。

因为第二行的 `x` 没有用 `let` 重新声明，因此它与第一行的 `x` 是同一个变量，虽然可变，但必须是 `i32` 类型的。

### 3.2 数据类型

#### 整型

|长度|有符号|无符号|
|--|--|--|
|8-bit|`i8`|`u8`
|16-bit|`i16`|`u16`
|32-bit|`i32`|`u32`
|64-bit|`i64`|`u64`
|128-bit|`i128`|`u128`
|arch|`isize`|`usize`

`isize` 在 32 位操作系统上是 `i32`，在 64 位操作系统上是 `i64`，`usize` 同理。

|数字字面值|例子|
|--|--|
|Decimal (十进制)|`98_222`|
|Hex (十六进制)|`0xff`|
|Octal (八进制)|`0o77`|
|Binary (二进制)|`0b1111_0000`|
|Byte (单字节字符)(仅限于u8)|`b'A'`|

数字字面值中可以添加下划线来分组，如 `1_000`。

Rust 的整型字面值默认是 `i32` 类型的，但可以通过添加类型后缀来指明类型

```rust
let a = 24;
let b = 24u8;
```

变量 `a` 是 `i32` 类型的，变量 `b` 是 `u8` 类型的。变量 `b` 的字面值也可以写作 `24_u8`。

整型溢出在 Debug 模式下会使程序 panic，在 Release 模式下则不会。

#### 浮点型

Rust 的浮点型有 `f32` 和 `f64` 两种。字面值默认是 `f64` 的。

```rust
let a = 0.4; //f64
let b: f32 = 0.4;
let c = 0.4f32;
let d = 0.4_f32;
```

后三个变量都是 `f32` 类型的。

> 注意，没有 `fsize` 这种类型。
{: .prompt-tip}

Rust 的整数除法与 C 是一样的，不会产生浮点数，而是向 0 舍入到最近的整数。

```rust
let a = 7 / 2; //3
let b = 7.0 / 2.0; //3.0
```

> 貌似不能让浮点数与整数进行运算。
{: .prompt-warning}

#### 布尔型

Rust 的布尔类型是 `bool`，只有两个值：`true` 和 `false`。

#### 字符类型

Rust 的字符类型是 `char`，大小是 4 个字节，代表一个 Unicode 标量值，因此 Rust 的一个字符可以是任意 Unicode 字符。

#### 元组类型

元组类型，长度固定，元素随意。

```rust
let a: (i32, f64, char) = (30, 2.4, 'a');
```
元组可以解构

```rust
let t = (30, 2.4, 'a');
let (i, f, c) = t;
println!("i: {i}, f: {f}, c: {c}");
```

用点和索引可以访问元组元素

```rust
let t = (30, 2.4, 'a');
let i = t.0;
```

单元（unit）元组是一种特殊值，写作 `()`，类型也是 `()`，表示空。如果一个函数没有显式返回任何值，则会隐式返回 `()`。

#### 数组类型

数组类型，长度 **固定**，元素类型也必须相同。

```rust
let a: [i32; 3] = [1, 2, 3];
let b: [f64; 3] = [4.0; 3];
```

第二行是声明 3 个 `4.0` 的快捷方式。

> 此处的类型注解不是必须的，只是为了说明数组类型的注解方式。
{: .prompt-tip}

使用方括号索引数组元素

```rust
let a = [1, 2, 3];
let t = a[0];
```

### 3.3 函数

Rust 的函数要求 **必须声明每个参数的类型**。

```rust
fn main() {
   say_age('B', 34);
}

fn say_age(name: char, age: i32) {
   println!("{name} is {age} years old.");
}
```

#### 语句和表达式

Rust 是一门基于表达式的语言。

**表达式** 计算并产生一个值，**语句** 执行一些操作，但不返回值。

`let a = 3;` 是一个语句，函数定义也是语句。

`5 + 6` 是一个表达式，`5` 也是一个表达式。函数调用是表达式，宏调用也是表达式，大括号建立的块作用域也是表达式

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };
    println!("y: {y}");
}
```

注意，上述 `x + 1` 后没有分号，表示它是一个表达式，也是整个块作用域的返回值。

#### 有返回值的函数

函数的返回值等于函数体最后一个表达式的值。如果使用 `return` 可以提前返回。

```rust
fn five() -> i32 {
    5 //这里没有分号
}

fn main() {
    let f = five();
    println!("{f}");
}
```

函数返回值需要声明类型。

### 3.5 控制流

#### `if` 表达式

`if` 是一个表达式，因此 `if` 的结果可以返回给一个变量

```rust
let n = 10;
let a = if n > 4 {
    n + 1
} else {
    n - 2
};
println!("n: {n}, a: {a}");
```

注意，`n + 1` 和 `n - 2` 后都没有分号，表示它们都是表达式，其结果会赋值给 `a`。

形如

```rust
fn is_even(num: i32) -> bool {
    if num % 2 == 0 {
        true
    } else {
        false
    }
}
```

这样的函数也可能常见，一个函数体的最后是一个 `if` 表达式，每个 `if` 分支的最后都是一个表达式，某个符合条件的表达式就会作为返回值返回。虽然看上去不是很直观，但的确是这么回事。

**`if` 表达式的判定条件必须是 `bool` 值**，像数字啥的就不行。

三元运算可以像下面这样表达

```rust
let r = 1;
let a = if r == -1 { "No" } else { "Yes" };
```

**`if` 每个分支中返回的值的类型必须相同**。

#### 循环

Rust 有三种循环：`loop` 、`while`、`for`。

`loop` 循环没啥条件，只能用 `break` 终止。

`loop` 是一个表达式，因此也可以返回值，`break` 也是个表达式，可以像用 `return` 一样使用，可以理解为中断循环时返回一个值。

```rust
let mut counter = 0;
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;
    }
};
println!("result: {result}");
```

当多层循环嵌套时，可以给循环加标签，然后在 `break` 的时候指明要跳出哪层循环

```rust
let mut count = 0;
'outer: loop {
    println!("count = {count}");
    let mut remain = 10;
    loop {
        println!("remain = {remain}");
        if remain == 9 {
            break;
        }
        if count == 2 {
            break 'outer;
        }
        remain -= 1;
    }
    count += 1;
}
```

`break` 把标签和返回值同时使用时返回值需要在后

```rust
let mut count = 0;
let res = 'outer: loop {
    println!("count = {count}");
    let mut remain = 10;
    loop {
        println!("remain = {remain}");
        if remain == 9 {
            break;
        }
        if count == 2 {
            break 'outer count;
        }
        remain -= 1;
    }
    count += 1;
};
println!("res: {res}");
```

`while` 没啥新东西，不说了。

`for` 循环的话举两个例子就清楚了

```rust
let a = [1, 2, 3, 5, 6];
for i in a {
    print!("{i} ");
}
print!("\n");
for j in 2..5 {
    print!("{j} "); // 2 3 4
}
print!("\n");
```

其中 `2..5` 不包括 `5`。

## 4. 认识所有权

这里先简单记一点关于权限的东西，稍后再细节整理：

> `R`：可读取数据，可拷贝数据  
`W`：可原地更改数据  
`O`：可转交所有权，可释放内存

当一个不可变变量创建的时候（包括用 `let` 声明赋值和初始化函数参数），它拥有 `R-O` 权限，如果是一个可变的变量，或者可变函数参数，则拥有 `RWO` 权限。

当一个变量被不可变引用时，失去 `-WO` 权限，如果被可变引用，则失去 `RWO` 权限。

一个引用的变量，其解引用的权限，可以根据类型判断，如果是 `&T` 则解引用后只有 `R--` 权限，如果是 `&mut T` 则解引用后有 `RW-` 权限。因为是引用，所以都没有 `--O` 权限。

为什么要关心一个引用变量解引用后的权限？因为我们操作一个引用变量的时候，大部分时候都是在操作其解引用后的数据，只不过 Rust 自动隐式地帮我们解引用我们不用直接明写出来，但还是需要考虑的。

> 详细的内容请查看 [Rust 所有权]({{site.url}}/posts/rust-ownership)。
{: .prompt-info}


## 5. 使用结构体组织相关联的数据

### 5.1 结构体的定义和实例化

#### 结构体基本使用

定义结构体

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

创建实例

```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };
}
```

实例的字段顺序不需要与定义时的一致。

使用结构体的字段

```rust
println!("name: {}", user1.username);
```

如果结构体实例是可变的，则可以修改字段

```rust
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("username123"),
        email: String::from("userone@example.com"),
        sign_in_count: 1,
    };

    user1.sign_in_count += 1;
    println!("{} has signed in {} times", user1.username, user1.sign_in_count);
}
```

> 注意，不能只指定结构体的某个字段可变。要变整个都变。
{: .prompt-tip}

结构体实例也是表达式，因此可以放在函数最后返回

```rust
fn build_user(username: String, email: String) -> User {
    User { 
        active: true,
        username: username,
        email: email, 
        sign_in_count: 1
    }
}

fn main() {
    let mut user1 = build_user(
        String::from("username123"), 
        String::from("userone@example.com")
    );

    user1.sign_in_count += 1;
    println!("{} has signed in {} times", user1.username, user1.sign_in_count);
}
```

上述函数中，结构体字段名和函数参数名相同，这种情况也很常见，因此有一种简写语法

```rust
fn build_user(username: String, email: String) -> User {
    User { 
        active: true,
        username,
        email, 
        sign_in_count: 1
    }
}
```

有时候我们创建的新结构体实例的大部分字段与已有的相同，只有部分不同，就可以用结构体更新语法

```rust
let user2 = User {
    email: String::from("another@example.com"),
    ..user1
};
println!("{} 's email is {}", user2.username, user2.email);
```

注意，`..user1` 必须在结构体所有字段的最后，且之后不能有逗号。

`..user1` 这样的更新语法等同于 `=` 赋值，因此也可能会转移所有权。比如上例中，`user1.username` 就被转移了所有权，在创建 `user2` 之后就不能使用 `user1.username` 了，但是 `user.active` 或者 `user.sign_in_count` 还可以使用，因为它们不是堆数据，支持 Copy。

#### 元组结构体

元组结构体的字段只有类型而没有名称，使用方法类似元组

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let grey = Color(127, 127, 127);
    let center = Point(5, 10, 5);

    println!("R: {}, y: {}", grey.0, center.1);
}
```

#### 类单元结构体

类单元结构体就跟单元元组类似，没有任何字段，通常用于定义一个不需要存储任何数据只需要实现 trait 的结构体（之后讨论）

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```
### 5.2 结构体示例程序

### 5.3 方法语法

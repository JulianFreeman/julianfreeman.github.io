---
title: Rust 杂记
date: 2024-07-04 19:17:00 -0400
categories: [笔记, Rust]
tags: [rust]
description: 本文记录一些自己在学习 Rust 过程中的理解
math: true
---

## 枚举的基本使用

枚举的定义比较简单：

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right,
}
```

使用 `match` 对枚举进行匹配的时候，需要匹配所有的可能性：

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

fn which_way(go: Direction) -> String {
    match go {
        Direction::Down => "Down".to_string(),
        Direction::Left => "Left".to_string(),
        Direction::Right => "Right".to_string(),
        Direction::Up => "Up".to_string(),
    }
}
```

如果有些可能性无关紧要，可以用一个变量代替：

```rust
#[derive(Debug)]
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

fn which_way(go: Direction) -> String {
    match go {
        Direction::Down => "Down".to_string(),
        Direction::Left => "Left".to_string(),
        any_var => format!("{:?} is irrelevent", any_var),
    }
}
```

> 这里为了使用 `any_var` 这个变量，并返回一个字符串，需要给 `Direction` 这个枚举加一个 `Debug` 特征。总之，这里的意思是，如果 `go` 是 `Direction::Right` 或者 `Direction::Up` 那么都会转到 `any_var` 这个分支上去，并且赋值给 `any_var` 这个变量。
{: .prompt-tip}

如果这个变量也用不到，则可以用 `_` 代替：

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

fn which_way(go: Direction) -> String {
    match go {
        Direction::Down => "Down".to_string(),
        Direction::Left => "Left".to_string(),
        _ => "I don't care".to_string(),
    }
}
```

但是，默认来说，枚举值是不能进行比较的。如果要进行比较，需要加 `PartialEq` 特征：

```rust
#[derive(PartialEq)]
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

fn is_left(go: Direction) -> bool {
    if go == Direction::Left {
        true
    } else {
        false
    }
}
```

不过这个样子好像也挺不专业的，其实完全可以用 `match` 代替了：

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

fn is_left(go: Direction) -> bool {
    match go {
        Direction::Left => true,
        _ => false,
    }
}
```

另外枚举没有 `Copy` 特征，所以二次赋值或者传参会转交所有权。除非加上 `Copy` 特征：

```rust
#![allow(dead_code)]

#[derive(Clone, Copy)]
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

fn which_way(go: Direction) -> String {
    match go {
        Direction::Down => "Down".to_string(),
        Direction::Left => "Left".to_string(),
        Direction::Right => "Right".to_string(),
        Direction::Up => "Up".to_string(),
    }
}

fn is_left(go: Direction) -> bool {
    match go {
        Direction::Left => true,
        _ => false,
    }
}

fn main() {
    let go = Direction::Right;
    println!("{}", which_way(go));
    println!("{}", is_left(go));
}
```

## 枚举的高级用法

使用 [官方文档](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html) 的例子，我们可以这样定义一个 IP 地址：

```rust
#![allow(dead_code, unused_variables)]

enum IpAddrKind {
    V4,
    V6,
}

struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

fn main() {
    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: "127.0.0.1".to_string(),
    };
}
```

但是多用一个结构体来绑定 IP 类型和地址，有点冗余，枚举本身也支持给变体（Variant）绑定数据：

```rust
#![allow(dead_code, unused_variables)]

enum IpAddrKind {
    V4(String),
    V6(String),
}

fn main() {
    let home = IpAddrKind::V4("127.0.0.1".to_string());
}
```

此时 `IpAddrKind::V4` 就变成了一个函数，接收一个 `String` 参数。

不同的变体可以接收不同的参数：

```rust
#![allow(dead_code, unused_variables)]

enum IpAddrKind {
    V4(u8, u8, u8, u8),
    V6(String),
}

fn main() {
    let home = IpAddrKind::V4(127, 0, 0, 1);
    let loopback = IpAddrKind::V6(String::from("::1"));
}

```

有数据的变体和没有数据的变体可以共存：

```rust
#![allow(dead_code, unused_variables)]

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let quit_msg = Message::Quit;
    let move_msg = Message::Move { x: 10, y: 20 };
    let write_msg = Message::Write("Hello".to_string());
    let change_msg = Message::ChangeColor(144, 234, 4);
}
```

高级匹配：

```rust
#![allow(dead_code)]

enum Discount {
    Percent(f64),
    Flat(i32),
}

fn main() {
    let flat4 = Discount::Flat(4);
    match flat4 {
        Discount::Flat(2) => println!("it's flat 2"),
        Discount::Flat(4) => println!("it's flat 4"),
        _ => println!("Don't care"),
    }
}
```

匹配第二个分支，而不会匹配第一个分支。也可以使用如下方式：

```rust
#![allow(dead_code)]

enum Discount {
    Percent(f64),
    Flat(i32),
}

fn main() {
    let flat4 = Discount::Flat(4);
    match flat4 {
        Discount::Flat(num) => println!("it's flat {}", num),
        _ => println!("Don't care"),
    }
}
```

## 结构体的定义和使用

```rust
#![allow(dead_code)]

struct ShippingBox {
    depth: i32,
    width: i32,
    height: i32,
}

fn main() {
    let my_box = ShippingBox {
        depth: 3,
        width: 2,
        height: 5,
    };
    let tall = my_box.height;
    print!("the box is {} units tall", tall);
}
```

注意，如果 `my_box.height` 的类型没有 `Copy` 特征，那么 `let tall = my_box.height` 这种赋值操作也会转交所有权。

如果变量与结构体的域（field）相同，可以使用简单写法：

```rust
struct Position {
    x: i32,
    y: i32,
}

fn main() {
    let x = 10;
    let y = 20;
    let p = Position { x, y };
    //而不必
    //let p = Position { x: x, y: y };
    println!("({}, {})", p.x, p.y);
}
```

结构体也可以用 `match` 匹配：

```rust
struct Worker {
    age: i32,
    salary: i32,
}

fn main() {
    let w = Worker {
        age: 32,
        salary: 10,
    };
    match w {
        Worker { salary: 10, age } => println!("age: {age}, salary: 10"),
        Worker { age, .. } => println!("The age is {}", age),
    }
}
```

第一个分支中给定了 `salary` 的值，因此该分支下并没有 `salary` 这个变量，只有 `age` 变量。

第二个分支只关心 `age` 这个数据，其他的不关心，可以用 `..` 代替。

## 元组的定义和解构

```rust
fn main() {
    let person_info: (String, i32) = (String::from("John"), 34);
    let (name, age): (String, i32) = person_info;
    println!("{} is {} years old.", name, age);
    // println!("{}", person_info.0);
    println!("age: {}", person_info.1);
}
```

元组的类型声明就是 **用圆括号括起固定数量的类型**。

元组的值就是 **用圆括号括起固定数量的值**。

解构的过程就是在等号左边用圆括号括起固定数量的变量名，以匹配等号右侧的元组值。

解构也相当于赋值，因此如果某些值没有 `Copy` 特征，则解构后，那一部分值就无法再使用。比如上例中的 `person_info.0` 便无法获取，但这不影响 `person_info.1` 的正常获取。

## impl 的基本使用

一段 C 风格的代码如下：

```rust
struct Temperature {
    degree: f64,
}

fn show_temp(temp: Temperature) {
    println!("The temperature is {}", temp.degree);
}

fn main() {
    let hot = Temperature { degree: 99.9 };
    show_temp(hot)
}
```

此时结构体 `Temperature` 和函数 `show_temp` 其实没什么关系，只不过后者的参数是前者。

`impl` 关键词可以把两者结合起来，如下：

```rust
struct Temperature {
    degree: f64,
}

impl Temperature {
    fn show_temp(temp: Temperature) {
        println!("The temperature is {}", temp.degree);
    }
}

fn main() {
    let hot = Temperature { degree: 99.9 };
    Temperature::show_temp(hot)
}
```

注意，此时 `show_temp` 属于 `Temperature` 域了，要调用需要加前缀，但是除此之外也没啥大的区别。

如果要有点区别，可以把 `show_temp` 的第一个参数改为 `self` ：

```rust
struct Temperature {
    degree: f64,
}

impl Temperature {
    fn show_temp(self: Temperature) {
        println!("The temperature is {}", self.degree);
    }
}

fn main() {
    let hot = Temperature { degree: 99.9 };
    // Temperature::show_temp(hot)
    hot.show_temp();
}
```

当改成 `self` 之后，就可以使用类似 `host.show_temp()` 的方式调用函数了。当然，`Temperature::show_temp(hot)` 的方式也管用。

在 `impl` 中的函数有个方便之处是，可以用 `Self` 类型代替结构体类型：

```rust
struct Temperature {
    degree: f64,
}

impl Temperature {
    fn show_temp(self: Self) {
        println!("The temperature is {}", self.degree);
    }
}

fn main() {
    let hot = Temperature { degree: 99.9 };
    hot.show_temp();
}
```

虽然 `Self` 是个特殊的类型，但是用起来也跟其他的类型没区别，在这里它只是 `Temperature` 的替身而已。所以，因为 `Temperature` 没有 `Copy` 特征，所以这个传参会转交所有权，`hot.show_temp()` 之后我们就无法使用 `hot` 这个变量了。通常这不是我们希望的情况，解决办法也跟普通类型没区别，借用就好了：

```rust
struct Temperature {
    degree: f64,
}

impl Temperature {
    fn show_temp(self: &Self) {
        println!("The temperature is {}", self.degree);
    }
}

fn main() {
    let hot = Temperature { degree: 99.9 };
    hot.show_temp();
    println!("temp: {}", hot.degree);
}
```

此时语法糖又来了，我们可以用 `&self` 代替 `self: &Self` ，所以这就是我们常见的 `impl` 中函数的样子了：

```rust
struct Temperature {
    degree: f64,
}

impl Temperature {
    fn show_temp(&self) {
        println!("The temperature is {}", self.degree);
    }
}

fn main() {
    let hot = Temperature { degree: 99.9 };
    hot.show_temp();
    println!("temp: {}", hot.degree);
}
```

虽然这个样子不是很直观，但是我们也要知道，这里的 `&self` 是 `Temperature` 示例（也就是 `hot`）的一个不可变借用。

如果要使用可变借用，可以使用 `&mut self`，也就是 `self: &mut Self` 的语法糖。

当然，要允许拥有可变调用，实例本身也必须是可变的：

```rust
struct Temperature {
    degree: f64,
}

impl Temperature {
    fn show_temp(&mut self) {
        println!("The temperature is {}", self.degree);
    }
}

fn main() {
    let mut hot = Temperature { degree: 99.9 };
    hot.show_temp();
    println!("temp: {}", hot.degree);
}
```

## 列表的定义与使用

```rust
fn main() {
    let a: [i32; 3] = [1, 2, 4];
    println!("{:?}", a);
    println!("second: {}", a[1]);
    let [c, d, e]: [i32; 3] = a;
    println!("[{c}, {d}, {e}]");
}
```

列表没有实现 `std::fmt::Display` 特征，所以无法使用 `"{}"` 打印，但是可以用 `"{:?}"` 打印。

列表获取索引的方式与元组不同，列表用方括号。

解构则类似。

列表是统一类型，固定长度的，不能 `push` 或者 `pop` 。

快速生成列表：

```rust
fn main() {
    let a = [1; 3];
    println!("{:?}", a);
}
```

## Vector 的创建和使用

创建方式一：

```rust
fn main() {
    let mut v1: Vec<i32> = Vec::new();
    v1.push(1);
    v1.push(2);
    v1.push(3);
    println!("{:?}", v1);
}
```

创建方式二：

```rust
fn main() {
    let v2 = Vec::from([1, 2, 3]);
    println!("{:?}", v2);
}
```

创建方式三：

```rust
fn main() {
    let v3 = vec![1, 2, 3];
    println!("{:?}", v3);
}
```

数组的容量和长度是不一样的：

```rust
fn main() {
    let mut v4: Vec<i32> = Vec::with_capacity(5);
    println!("{:?}, {}", v4, v4.len());
    v4.resize(v4.capacity(), 1);
    println!("{:?}, {}", v4, v4.len());
}
```

遍历数组：

```rust
fn main() {
    let v = vec![2, 4, 6, 8];
    for n in v {
        println!("{}", n);
    }
}
```

这里的遍历隐式调用了 `v.into_iter()`，而该函数的签名是 `fn into_iter(self) -> Self::IntoIter` ，传参并不是引用，所以会直接转移数组的所有权（因为 `Vec` 没有 `Copy` 特征）。所以在遍历之后，就不能再使用 `v` 变量了。

解决办法：

```rust
fn main() {
    let v = vec![2, 4, 6, 8];
    let mut c: usize = 0;
    for n in &v {
        println!("as value: {}", n);
        println!("as addr : {:p}", n);
        println!("its addr: {:p}", &n);
        println!("idx addr: {:p}", &v[c]);
        println!("-----------");
        c += 1;
    }
}
```

遍历时用 `&v` 就好了，这样只会借用，而不是转交所有权了。

此时 `n` 的类型是 `&i32` ，也就是数组中每个元素的引用。

遍历下来，每次循环中，第一行打印发生了隐式的从 `&i32` 到 `i32` 的转换，实际上 `n` 的值是一个地址，因为其类型是引用。这个值与 `&v[c]` 相同，即数组中每个元素的地址。

在所有循环中，`&n` 的值都不变，也就是 `n` 的地址没变。

> 这里的 `c` 必须是 `usize` 类型，这是 `v[c]` 的要求。
{: .prompt-tip}

## 对 & 的理解

在讲 `&` 之前，先讲讲数学中的根号：

$$ \sqrt{x} $$

我上学的时候听到过这么一句话，平方根的结果有正有负，算术平方根的结果只有正。

小时候就把这句话这么记住了，但是却不明白，为什么算术平方根的结果只有正？为什么会有平方根和算术平方根的区别？

不管区别是什么吧，后来我明白了一件事，平方根这个符号，有两个含义。

第一种含义是 **动词性质** 的含义，意思是要取某个值的平方根，它代表的是一种运算过程，也就是说平方根这个符号在这里是一个运算符，比如：

$$ \sqrt{4} = \pm2 $$

在这里，根号 4 是一个运算过程，其结果是 2 或者 -2。

第二种含义是 **形容词性质** 的含义，意思是某个值的平方根，它仅仅代表一个值，是一种数值的写法，比如：

$$ \sqrt{7} $$

这里根号 7 并不是要运算 7 的平方根，而仅仅是根号 7 这个数值的写法，就如同 -2 就是 -2，是这个数字的写法，并不表示减去 2。

因此这里根号 7 就是正的，而不是既有正又有负。

回到 `&` ，这个符号也有两种含义：

一种是动词性质的，表示取某个变量的地址，比如：

```rust
let a = 10;
let b = &a;
```

第二种是形容词性质的，表示某个类型是个引用类型，比如：

```rust
fn say_hi(name: &String) {}
```

## &str 的基本使用

`to_string` 可以把一些类型转换为 `String` 类型：

```rust
fn main() {
    let s: &str = "hello";
    let n: i32 = 10;
    let ss: String = s.to_string();
    let nn: String = n.to_string();
    println!("{}, {}", ss, nn);
}
```

`to_owned` 可以把引用类型通过 Clone 转为有所有权的类型：

```rust
fn main() {
    let s: &str = "hello";
    let n: &i32 = &10;
    let ss: String = s.to_owned();
    let nn: i32 = n.to_owned();
    println!("{}, {}", ss, nn);
}
```

对于 `&str` 来说，两者作用差不多。

向函数传递字符串参数时，推荐使用 `&str` ，因为更快：

```rust
fn say_hello(name: &str) {
    println!("Hello, {}", name);
}

fn main() {
    say_hello("John");
}
```

在结构体中使用 `&str` 是一件比较麻烦的事情，因为需要保证结构体中 `&str` 类型的变量至少要活得跟结构体实例一样久，因此需要指定生命周期：

```rust
struct Person<'a> {
    name: &'a str,
}

fn a_person(name: &str) {
    let p = Person { name };
    println!("hello, {}", p.name);
}

fn main() {
    let name = "John";
    a_person(name);
    println!("{} is still here", name);
}
```

所以，结构体中保存字符串，建议使用 `String` 。

## Option 的基本使用

`Option` 是一个枚举，因此需要先了解 [Rust enum](#枚举的基本使用) 。

使用场景一：

```rust
struct Person {
    age: i32,
    name: String,
}

fn find_age(name: &str, persons: &Vec<Person>) -> Option<i32> {
    for person in persons {
        if person.name == name {
            return Some(person.age);
        }
    }
    None
}

fn main() {
    let persons_info = vec![
        Person { name: "John".to_string(), age: 32 },
        Person { name: "Karl".to_string(), age: 45 },
        Person { name: "Liz".to_string(), age: 24 },
    ];
    let age = find_age("Karl", &persons_info);
    match age {
        None => println!("age not found"),
        Some(a) => println!("the age is {a}"),
    }
}
```

使用场景二：

```rust
fn main() {
    let mut nums = vec![1, 2, 3];
    loop {
        let n = nums.pop();
        match n {
            None => {
                println!("no more numbers");
                break;
            }
            Some(i) => {
                println!("{i} is popped");
            }
        }
    }
}
```

## Result 的基本使用

`Result` 是一个枚举，因此需要先了解 [Rust enum](#枚举的基本使用) 。

`Result` 的定义：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E)
}
```

错误类型 `E` 也是一个泛型，因此可以是任何类型，数字、字符串，或者更复杂的结构体。

### 使用场景

使用场景一：

```rust
struct SoundData {
    kind: String,
}

impl SoundData {
    fn new(kind: &str) -> Self {
        Self { kind: kind.to_string() }
    }
}

fn get_sound(kind: &str) -> Result<SoundData, String> {
    if kind == "alert" {
        Ok(SoundData::new("alert"))
    } else {
        Err("Unable to find sound data".to_string())
    }
}

fn main() {
    let sound = get_sound("alert");
    match sound {
        Ok(sd) => println!("sound data located: {}", sd.kind),
        Err(e) => println!("error: {e}"),
    }
}
```

使用场景二：

```rust
#[derive(Debug)]
enum MenuChoice {
    MainMenu,
    Start,
    Quit,
}

fn get_choice(input: &str) -> Result<MenuChoice, String> {
    match input {
        "mainmenu" => Ok(MenuChoice::MainMenu),
        "start" => Ok(MenuChoice::Start),
        "quit" => Ok(MenuChoice::Quit),
        c => Err(format!("menu choice [{c}] not found")),
    }
}

fn main() {
    let choice = get_choice("start");
    match choice {
        Ok(mc) => println!("found a choice: {mc:?}"),
        Err(e) => println!("error: {e}"),
    }
}
```

使用场景三：

```rust
struct Adult {
    name: String,
    age: u8,
}

impl Adult {
    fn new(name: &str, age: u8) -> Result<Self, String> {
        if age >= 21 {
            Ok(Self { name: name.to_string(), age })
        } else {
            Err("the age should be older than 21".to_string())
        }
    }
}

fn show(a: &Result<Adult, String>) {
    match a {
        Ok(adult) => println!("{} is {} years old", adult.name, adult.age),
        Err(err) => println!("error: {err}"),
    }
}

fn main() {
    let a1 = Adult::new("John", 14);
    let a2 = Adult::new("Karl", 34);
    show(&a1);
    show(&a2);
}
```

### ? 的使用

```rust
#[derive(Debug)]
enum MenuChoice {
    MainMenu,
    Start,
    Quit,
}

fn get_choice(input: &str) -> Result<MenuChoice, String> {
    match input {
        "mainmenu" => Ok(MenuChoice::MainMenu),
        "start" => Ok(MenuChoice::Start),
        "quit" => Ok(MenuChoice::Quit),
        c => Err(format!("menu choice [{c}] not found")),
    }
}

fn print_choice(input: &str) -> Result<(), String> {
    let choice: MenuChoice = get_choice(input)?;
    println!("the choice: {choice:?}");
    Ok(())
    // let choice: Result<MenuChoice, String> = get_choice(input);
    // match choice {
    //     Ok(c) => {
    //         println!("the choice: {c:?}");
    //         Ok(())
    //     }
    //     Err(e) => Err(e),
    // }
}

fn main() {
    let r = print_choice("quit");
    match r {
        Ok(_) => {}
        Err(e) => println!("error: {e}"),
    }
}
```

`print_choice` 函数中，没有注释的内容与注释的内容作用是相同的，但是更简洁。

`get_choice(input)?` 会在遇到 `Err` 的时候 **直接返回** `Err`，如果不是，则将 `Ok` 中的数据返回给 `choice`。

因此，两个 `choice` 的类型是不一样的。

## 对所有权转移的理解

看如下代码：

```rust
struct Position {
    x: i32,
    y: i32,
}

impl Position {
    fn origin() -> Self {
        Self { x: 0, y: 0 }
    }

    fn show(&self) {
        println!("({}, {})", self.x, self.y);
    }
}

fn main() {
    let o = Position::origin();
    o.show();
}
```

在 `origin` 中，返回值毫无疑问发生了所有权转移，但是这个转移具体发生了什么，需要理解一下。

实际上，move 和 copy 发生的操作差不多，都是把数据从一个地方转移到了另一个地方，不过区别是，move 之后的原数据就不能用了，copy 之后的原数据还可以继续用。

我们打印一下地址看看：

```rust
#![allow(dead_code)]

struct Position {
    x: i32,
    y: i32,
}

impl Position {
    fn origin() -> Self {
        let ori = Self { x: 0, y: 0 };
        println!("in  stack: {:p}", &ori);
        ori
    }
}

fn main() {
    let ori = Position::origin();
    println!("out stack: {:p}", &ori);
}
```

输出类似于：

```console
in  stack: 0xda1f32fa38
out stack: 0xda1f32fad8
```

地址并不一样。

所以转移所有权也会发生地址变动，可以将其理解为拷贝吧，只不过拷贝之后原地址就不可用了。

除非重新赋值：

```rust
fn main() {
    let mut s = String::from("hello");
    println!("{:p}", &s);
    let t = s;
    println!("{:p}", &t);
    s = String::from("world");
    println!("{:p}", &s);
}
```

输出类似于：

```console
0x41d26ff570
0x41d26ff5d0
0x41d26ff570
```

当然，要允许重新赋值，就得保证 `s` 是可变的，如果不可变的话，连重新赋值也不能了。

## Rust Book 的第一个例子

### 1. 获取一行输入

```rust
use std::io;

fn main() {
    println!("猜测一个数");
    let mut guess = String::new();
    io::stdin().read_line(&mut guess).expect("无法读取行");
    println!("您猜测的数是：{}", guess);
}
```

首先创建一个字符串变量用来保存读取到的内容，这个变量需要是 **可变** 的，使用 `String::new()` 创建。

然后使用 `io:stdin().read_line` 来读取，参数是一个 **可变引用** 类型。

读取可能会出错，因此在后面接一个 `expect` 函数，参数是错误提示信息。

### 2. expect 的表现

```rust
fn main() {
    let e: Result<i32, &str> = Err("错误信息");
    e.expect("出现错误");
}
```

报错信息的格式为：`出现错误: "错误信息"` 。

这个 `Result` 全称是 `std::result::Result` ，是较通用的类型，还有其他特定的类型比如 `std::io::Result` 等。

### 3. 生成随机数

首先在项目下运行 `cargo add rand` ，添加这个库。

```rust
use rand::Rng;

fn main() {
    let num = rand::thread_rng().gen_range(1..101);
    println!("The number: {}", num);
}
```

`rand::thread_rng` 是一个随机数生成器， 包含在 `rand::Rng` 这个 Trait 里面。

其中 `gen_range` 用于在指定的范围里生成一个随机数。

`1..101` 表示 `1 <= x < 101` ，`1..=100` 表示 `1 <= x <= 100` 。

### 4. 用 match 比较大小

```rust
use std::cmp::Ordering;

fn main() {
    let a = 10;
    let b = 20;
    match a.cmp(&b) {
        Ordering::Less => println!("小了"),
        Ordering::Greater => println!("大了"),
        Ordering::Equal => println!("正好"),
    }
}
```

### 5. 用 match 替换 expect

使用 `expect` 会在出错时直接使程序崩溃，比如：

```rust
fn main() {
    println!("输入一个数字");
    let mut num = String::new();
    std::io::stdin().read_line(&mut num).expect("无法读取行");
    let num: i32 = num.trim().parse().expect("无法转换为数字");
    println!("转换后：{}", num);
}
```

如果输入的字符串不是纯数字，程序会在第 5 行崩溃，崩溃后的代码都不会运行。但有时候我们不想这样，出现错误就使程序崩溃不是一种正确的错误处理办法。

此时我们可以改成这样：

```rust
fn main() {
    loop {
        println!("输入一个数字");
        let mut num = String::new();
        std::io::stdin().read_line(&mut num).expect("无法读取行");
        let num: i32 = match num.trim().parse() {
            Ok(n) => n,
            Err(_) => {
                println!("输入的不是一个纯数字！");
                continue
            }
        };
        println!("转换后：{}", num);
        break
    }
}
```

这是一种常见的错误处理方式。

同样，对于读取字符串过程中的错误处理，也可以改成如下：

```rust
fn main() {
    loop {
        println!("输入一个数字");
        let mut num = String::new();
        match std::io::stdin().read_line(&mut num) {
            Ok(_) => {},
            Err(err) => {
                println!("无法读取行：{}", err);
                break
            },
        };
        
        let num: i32 = match num.trim().parse() {
            Ok(n) => n,
            Err(_) => {
                println!("输入的不是一个纯数字！");
                continue
            }
        };
        println!("转换后：{}", num);
        break
    }
}
```

其中 `read_line` 成功后会返回读取到的字节数，是一个 `usize` 类型，不过这里我们不关心，因此直接用 `Ok(_) => {}` ，如果出错的话，就打印错误信息，然后中止循环。

### 6. for 循环遍历固定次数

```rust
fn main() {
    println!("正向：");
    for i in 1..5 {
        println!("{i}");
    }
    println!("逆向：");
    for j in (1..5).rev() {
        println!("{j}");
    }
}
```

## 处理 JSON 数据

添加 JSON 库：

```shell
cargo add serde_json
```

官方 API 文档：https://docs.rs/serde_json/latest/serde_json/

### 基础

在 Rust 中处理 JSON 数据的三种方式：

- **作为文本数据**。意思就是你拿到的 JSON 数据是一段纯文本，比如直接从文本读取的等。
- **作为一种无类型形式的数据**。把 JSON 文本转换为没有严格的类型限定的 Rust 数据格式进行处理。
- **作为强类型数据结构**。把 JSON 文本转换为严格类型限定的 Rust 数据格式。

### 处理无类型 JSON 数据

任何合法的 JSON 文本都可以转换成由如下枚举递归而成的数据形式：

```rust
enum Value {
    Null,
    Bool(bool),
    Number(Number),
    String(String),
    Array(Vec<Value>),
    Object(Map<String, Value>),
}
```

这个数据结构叫 `serde_json::Value`。

可以用 `serde_json::from_str` 函数从文本读取 JSON 数据，然后转换为 `serde_json::Value` 格式。

```rust
use serde_json::{Result, Value};

fn untyped_example() -> Result<()> {
    let data = r#"
        {
            "name": "John Doe",
            "age": 43,
            "phones": [
                "+44 12456",
                "+44 24352"
            ]
        }
    "#;
    let v: Value = serde_json::from_str(data)?;
    println!("Please call {} at the number {}", v["name"], v["phones"][0]);

    Ok(())
}
```

其中 `v["name"]` 的类型是 `&Value`，因此打印出来是形如 `"John Doe"` 的样子，带着一个双引号。

要去除这个双引号，可以使用 `as_str()` 方法，这个方法返回一个 `Option<&str>` 类型，之所以是 `Option` 类型，是因为并不是所有的 `&Value` 都能转成字符串，比如 `v["age"]` 就不行，可以转就是一个 `Some(&str)`，转不了就是一个 `None`。

如果说有的键值不存在，比如 `v["abc"]`，则会返回 `Value::Null`。

### 处理强类型 JSON 数据

需要安装 `serde` 并且开启 `derive` 功能：

```shell
cargo add serde -F derive
```
{: .nolineno}

```rust
use serde::{Deserialize, Serialize};
use serde_json::Result;

#[derive(Serialize, Deserialize)]
struct Person {
    name: String,
    age: u8,
    phones: Vec<String>,
}

fn typed_example() -> Result<()> {
    let data = r#"
        {
            "name": "John Doe",
            "age": 43,
            "phones": [
                "+44 12456",
                "+44 24352"
            ]
        }
    "#;
    let p: Person = serde_json::from_str(data)?;
    println!("Please call {} at the number {}", p.name, p.phones[0]);

    Ok(())
}
```

### 构建 JSON 数据

`serde_json` 提供了 `json!` 宏来构建 `serde_json::Value` 对象：

```rust
use serde_json::json;

fn main() {
    let john = json!({
        "name": "John Doe",
        "age": 43,
        "phones": [
            "+44 12345",
            "+44 34243"
        ]
    });

    println!("First phone number: {}", john["phones"][0]);
    println!("{}", john.to_string());
}
```

`to_string` 函数可以把 `serde_json::Value` 对象转成字符串。`json!` 宏内也可以直接写变量，只要能序列化的就行。比如 `"name": full_name`。

但是问题依旧，这种方式并没有类型提示。

### 序列化数据结构

```rust
use serde::{Deserialize, Serialize};
use serde_json::Result;

#[derive(Serialize, Deserialize)]
struct Address {
    street: String,
    city: String,
}

fn print_an_address() -> Result<()> {
    let address = Address {
        street: "10 Downing Street".to_owned(),
        city: "London".to_owned(),
    };

    let j = serde_json::to_string(&address)?;
    println!("{}", j);
    Ok(())
}
```

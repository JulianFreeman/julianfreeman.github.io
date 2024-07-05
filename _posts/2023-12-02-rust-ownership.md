---
title: Rust 所有权
date: 2023-12-02 21:31:00 -0500
categories: [翻译, Rust]
tags: [rust]
description: Rust Book 第四章的完全翻译（省略图示功能）（个人翻译，不作权威）
---

> 原文链接：https://rust-book.cs.brown.edu/
{: .prompt-info}

## 理解所有权

所有权是 Rust 最独特的功能，其对 Rust 语言的其他部分有着深刻的影响。所有权保证了 Rust 可以在不需要垃圾回收器的情况下保证内存安全，所以理解所有权是如何工作的就十分重要。在这一章，我们会讨论所有权以及几个相关特性：借用、切片，以及 Rust 如何在内存中排列数据。

## 什么是所有权？

所有权是一种保证 Rust 程序 **安全** 的机制。要理解所有权，我们需要先理解什么会导致 Rust
程序不安全。

### 安全即没有未定义行为

让我们先举一个例子。下面的程序是可以安全执行的：

```rust
fn read(y: bool) {
    if y {
        println!("y is true!");
    }
}

fn main() {
    let x = true;
    read(x);
}
```

要让这个程序变得不安全，我们可以把 `read` 调用挪到 `x` 的定义之上：

```rust
fn read(y: bool) {
    if y {
        println!("y is true!");
    }
}

fn main() {
    read(x); // oh no! x isn't defined!
    let x = true;
}
```

> 这段代码无法通过编译
{: .prompt-warning}

第二个程序是不安全的，因为 `read(x)` 期望 `x` 有一个 `bool` 类型的值，但是此时 `x` 还没有值。

如果一个解释器执行类似上述的代码，在 `x` 定义之前试图读取就会引发一个异常，比如在 Python 中是 `NameError`，在 JavaScript 中是 `ReferenceError`。但是想引发异常是有代价的，每次程序想要读取一个变量，解释器就得检查这个变量是不是已经定义了。

Rust 的目标是编译出一个高效的二进制程序，能尽可能少的做运行时检查。因此 Rust 不会在 *运行时* 检查一个变量在使用前有没有被定义。相反，Rust 在 *编译时* 检查。如果你尝试编译这段不安全的程序，你会看到如下报错：

```console
error[E0425]: cannot find value `x` in this scope
 --> src/main.rs:8:10
  |
8 |     read(x); // oh no! x isn't defined!
  |          ^ not found in this scope
```

大概你也能感觉到，Rust 保证一个变量在使用前是被定义的，这是一件好事。但是为何要如此呢？为了论证这一点，我们需要知道：**如果 Rust 允许这样一个不安全的程序编译，会发生什么？**

让我们先来看一下一个安全的程序是如何编译并执行的。在一台使用 x86 架构处理器的计算机上，Rust 编译上述安全程序的 `main` 函数并生成了如下汇编代码（[点击查看完整代码](https://rust.godbolt.org/z/xnT1fzsqv)）：

```nasm
main:
    ; ...
    mov     edi, 1
    call    read
    ; ...
```

> 如果你不熟悉汇编代码，没有关系。这一部分会涉及到几个汇编代码的例子，但只是为了向你展示 Rust 底层是如何工作的。汇编不是学习 Rust 的必需工具。
{: .prompt-info}

这段汇编代码做了如下几件事：

- 将数字 1，也就是 `true`，移动到一个叫 `edi` 的寄存器（一种汇编语言的变量）中。
- 调用 `call` 函数，同时期望能在 `edi` 这个寄存器中找到参数 `y` 的值。

如果那段不安全的代码被允许编译，那么生成的汇编代码可能会像下面这样：

```nasm
main:
    ; ...
    call    read
    mov     edi, 1    ; mov is after call
    ; ...
```

这个程序是不安全的，因为 `read` 希望 `edi` 中会有一个布尔值，要么是 `0` 要么是 `1`，但是此时 `edi` 里可能是任意值：`2`，`100`，`0x1337BEEF` 等等。当 `read` 想使用参数 `y` 的时候，就会立刻触发 **未定义行为**！

Rust 没有规定当你运行 `if y { .. }` 但 `y` 既不是 `true` 也不是 `false` 的时候会发生什么。这种行为，或者说执行了这段代码之后会发生什么，是 *未定义的*。什么都可能发生，比如：

- 程序执行得好好的，没有崩溃，没人发现任何问题。
- 程序立刻因为[存储器区段错误](https://zh.wikipedia.org/wiki/%E8%A8%98%E6%86%B6%E9%AB%94%E5%8D%80%E6%AE%B5%E9%8C%AF%E8%AA%A4)或其他操作系统错误而崩溃。
- 程序没有崩溃，直到某一天一个别有用心的人借此找到了一个漏洞，然后删掉了你的生产数据库，重写了你的备份，还偷走了你的午饭钱。

**Rust 一个最基本的目标是保证你的程序永远没有未定义行为。** 这就是“安全”的含义。未定义行为对于能直接操作内存的低级语言来说特别危险。在[报道出来的有关低级系统的安全漏洞](https://msrc.microsoft.com/blog/2019/07/a-proactive-approach-to-more-secure-code/)中，大概有 70% 是由内存损坏引起的，而内存损坏就是其中一种未定义行为。

Rust 的第二个目标就是在 *编译时* 就阻断未定义行为，而不是在 *运行时*。这样做有两个原因：

1. 在编译时发现 bug，意味着在发布后能避免这些 bug，这样提高了软件的稳定性。
2. 在编译时发现 bug，意味着能少一些运行时检查，这样提高了软件的性能。

Rust 不能防止所有的 bug。如果一个程序暴露了一个公有的无需授权的“删除生产数据库”端点，那么别有用心的人就不需要尝试什么 `if` 语句来删除数据库了。但是 Rust 的保护机制相比于其他少有保护机制的语言，还是能让程序更安全些的。[谷歌的安卓团队](https://security.googleblog.com/2022/12/memory-safe-languages-in-android-13.html)是这么说的。

### 所有权，一种内存安全机制

既然安全指的是没有未定义行为，而所有权是有关安全的，那我们就需要理解所有权如何防止未定义行为的。Rust 参考手册中维护了一个“[被认为是未定义行为](https://doc.rust-lang.org/reference/behavior-considered-undefined.html)”的长长的列表。但目前为止，我们只需要关注其中一个方面：内存操作。

内存是程序执行时用来存储数据的地方。我们可以用很多种方式来思考内存：

- 如果你不熟悉系统编程，你可能会在一个较高的抽象上把内存理解为电脑里的内存条，或者一个我加载数据越多就会变得越少的东西。
- 如果你熟悉系统编程，你可能会在一个较低的抽象上把内存理解为一个字节数组，或者我调用 `malloc` 时拿回来的指针。

上述两种内存模型都是合理的，但是用它们来思考 Rust 的运作模式不是很有用。较高的抽象，太高了，因为你还是需要理解指针的概念的。较低的抽象，太低了，毕竟 Rust 不允许你把内存解析为一串字节数组。

Rust 用一种特别的方式来思考内存。所有权就是在这种思考方式下，用来安全使用内存的机制。本章剩余的部分会解释 Rust 的内存模型。

### 栈中的变量

下面是一段代码，定义了一个数字 `n`，然后用 `n` 调用函数 `plus_one`。

```rust
fn main() {
    let n = 5; //L1
    let y = plus_one(n); //L3
    println!("The value of y is: {y}");
}

fn plus_one(x: i32) -> i32 {
    //L2
    x + 1
}
```

变量存在于 **栈帧** 中。栈帧是一种在单一作用域（比如函数）下从变量到值的映射。举例如下：

- L1 位置下，`main` 的栈帧中保存了 `n = 5`。
- L2 位置下，`plus_one` 的栈帧中保存了 `x = 5`。
- L3 位置下，`main` 的栈帧中保存了 `n = 5; y = 6`。

栈帧被放在一个当前调用函数的 **栈** 中。比如说，在 L2 位置，`main` 的栈帧在 `plus_one` 的栈帧之上。当函数返回时，Rust 就会释放这个函数的栈帧。这样一个栈帧的序列被称为一个栈，因为最后加入的帧是最先被释放的。

> 这个内存模型并不完全符合 Rust 的工作模式。就像我们之前在汇编代码那里看到的，Rust 编译器也可能会把 `n` 或者 
`x` 放入一个寄存器而不是栈帧。但这些区别都是具体的实现细节了，不影响我们理解 Rust 的安全机制。所以我们还是只关注栈帧中的变量这种简单的情况。
{: .prompt-info}

当一个表达式要读取一个变量的时候，程序就会从栈帧中把这个变量的值拷贝一份。比如说我们运行这段程序：

```rust
let a = 5;
let mut b = a; // b = 5
b += 1; // a = 5; b = 6
```

`a` 的值被拷贝了一份给 `b`，在 `b` 自增之后，`a` 依然保持不变。

### 堆上的 Box

然而，拷贝数据会占用大量内存。比如说，下面是一个稍微修改过的程序，这个程序拷贝了一个有一百万个值的数组。

```rust
let a = [0; 1_000_000];
let b = a;
```

可以看到，把 `a` 拷贝到 `b` 之后，`main` 的栈帧上保存了两百万个值。

如果想转移数据但又不想拷贝，Rust 使用 **指针** 来达到这一点。指针是一个描述内存中某个位置的值。那个被指针指向的值就被叫做被指者。一个常用的获取指针的方式是在 **堆** 上分配内存。堆是内存中一块特别的区域，在这里数据可以不限期地保留。堆上的数据不会跟某个特定的栈帧绑定。Rust 提供了一个叫 `Box` 的结构来把数据放在堆上。比如说，我们可以把那有一百万个值的数组放进 `Box::new` 中：

```rust
let a = Box::new([0; 1_000_000]); // L1
let b = a; // L2
```

现在可以看到，此时只有一个数组存在了。在 L1，`a` 的值是一个指针，指向了堆上的一个数组。语句 `let b = a` 把 `a` 保存的指针拷贝给了 `b`，但是被指向的数据并没有拷贝。注意，现在 `a` 实际是被转移了，我们稍后会解释这是什么意思。

### Rust 不允许手动管理内存

内存管理是一个分配内存和释放内存的过程。换句话说，内存管理就是寻找那些不再使用的内存并把它们返还。栈帧是由 Rust 自动管理的。当一个函数被调用的时候，Rust 会分配一块栈帧给这个函数，当这个调用结束的时候，Rust 会释放这个栈帧。

我们在前面看到，堆上的数据是我们调用 `Box::new(..)` 的时候分配的，那什么时候释放呢？假如说 Rust 有一个叫 `free()` 的函数用来释放堆上的内存，想象一下 Rust 让程序员来决定他们何时要调用 `free`。这种手动的内存管理很容易就会导致 bug。比如说，我们可能会尝试读取一个指向已经被释放的内存的指针：

```rust
let b = Box::new([0; 100]); // L1
free(b); // L2
assert!(b[0] == 0); // L3
```

> 上述代码不可编译
{: .prompt-warning}

> 你可能会想，我们怎么执行一段 Rust 不让编译的程序呢？我们使用一些[特殊的工具](https://github.com/cognitive-engineering-lab/aquascope)来模拟 Rust 的借用检查器被禁用的情况，当然，为了教学目的。这样我们就可以回答一些假设的问题，比如：假设 Rust 可以让不安全的程序通过编译？
{: .prompt-info}

这里，我们在堆上分配了一个数组，然后我们调用了 `free(b)`，这使得 `b` 的堆内存被释放了。因此 `b` 的值就变成一个指向非法内存的指针了。目前未知还没有任何未定义行为发生！在 L2 的位置，程序尚且安全。保留一个非法指针并不是啥大问题。

未定义行为发生在当我们想通过 `b[0]` 使用这个指针的时候。这个行为试图获取非法内存，可能就会导致程序崩溃，或者可能更糟，程序没有崩溃，而是返回了任意数据。因此这个程序是 **不安全** 的。

Rust 不允许程序手动释放内存。这种政策就避免了上述未定义行为的发生。

### Box 的所有者负责释放内存

相反，Rust 会 *自动* 释放 box 的堆内存。这是一个关于 Rust 释放 box 的 *几乎* 准确的描述：

> **Box 的释放原则（几乎准确）**：如果一个变量绑定了一个 box，当 Rust 释放该变量所在的栈帧时，也会一同释放这个 box 的堆内存。

举个例子，我们来跟踪一下下面这个程序是如何分配和释放 box 的：

```rust
fn main() {
    let a_num = 4; // L1
    make_and_drop(); // L3
}

fn make_and_drop() {
    let a_box = Box::new(5); // L2
}
```

在 L1，调用 `make_and_drop` 之前，内存的状态就只有 `main` 的栈帧。在 L2，调用 `make_and_drop` 的时候，`a_box` 指向了堆上的 `5`。当 `make_and_drop` 结束的时候，Rust 释放它的栈帧。因为 `make_and_drop` 保存了变量 `a_box`，因此 Rust 也释放了 `a_box` 所指向的堆数据。因此到 L3 的时候，堆上又是空的了。

Box 的堆内存管理一切正常。但是如果我们使坏怎么办？回到之前的一个例子上，如果我们把一个 box 绑定到两个变量会发生什么？

```rust
let a = Box::new([0; 1_000_000]);
let b = a;
```

现在堆上的数组被同时绑定到了 `a` 和 `b` 上。根据我们“几乎准确”的原则，Rust 会尝试释放 box 的堆内存 *两次*。这也是一个未定义行为！

为了避免这个问题，所有权终于登场了。当 `a` 绑定到 `Box::new([0; 1_000_000])` 的时候，我们称 `a` **拥有** 这个 box。语句 `let b = a` 把这个 box 的所有权从 `a` **转移** 给了 `b`。根据这些说明，我们可以更加准确地描述 Rust 释放 box 的政策：

> **Box 的释放原则（完全准确）**：如果一个变量拥有一个 box，当 Rust 释放这个变量的栈帧时，也会一同释放这个 box 的堆内存。

在上述例子中，`b` 拥有数组的 box，因此当这个作用域结束时，Rust 只会因为 `b` 而释放这块内存一次。

### 集合也使用 Box

Rust 的数据结构[^1]，比如 `Vec`、`String`、`HashMap` 等，使用 Box 来保存可变数量的值。比如说，下面这段程序可以创建、移动、修改一个字符串：

```rust
fn main() {
    let first = String::from("Ferris"); // L1
    let full = add_suffix(first); // L4
    println!("{full}");
}

fn add_suffix(mut name: String) -> String {
    // L2
    name.push_str(" Jr."); // L3
    name
}
```

这段程序涉及到很多，所以跟上了：

1. 在 L1，堆上分配了字符串 `"Ferris"`，由 `first` 拥有。
2. 在 L2，函数 `add_suffix(first)` 被调用了。这导致字符串的所有权从 `first` 转移到了 `name`。字符串本身的数据没有被拷贝，但是指向这个字符串的指针被拷贝了。
3. 在 L3，函数 `name.push_str(" Jr.")` 调整了字符串在堆上的大小。这里包含了三件事。一，分配一块新的更大的内存；二，把 `"Ferris Jr."` 写入这块内存；三，释放原来的内存。现在 `first` 指向了一个已经被释放的内存。
4. 在 L4，栈帧 `add_suffix` 没了，函数返回了 `name`，把字符串的所有权转交给了 `full`。

### 变量在被转移后就不能使用了

上面的字符串程序描述了一个关于所有权的重要安全原则。想象一下如果在调用 `add_suffix` 之后 `first` 在 `main` 函数里又被使用了，会怎样。我们可以模拟一下这个程序，看看会导致什么未定义行为：

```rust
fn main() {
    let first = String::from("Ferris");
    let full = add_suffix(first);
    println!("{full}, originally {first}"); // L1 // first is now used here
}

fn add_suffix(mut name: String) -> String {
    name.push_str(" Jr.");
    name
}
```

> 上述代码不可编译
{: .prompt-warning}

在调用 `add_suffix` 之后，`first` 指向了一块被释放的内存。在 `println!` 中试图读取 `first` 是一种对内存安全的侵犯（未定义行为）。请记住：`first` 是一个指向被释放内存的指针，这不是什么问题，问题在于当它变成非法指针之后我们依旧尝试 *使用* 它。

好在，Rust 会拒绝编译这段程序，并给出如下错误：

```console
error[E0382]: borrow of moved value: `first`
 --> test.rs:4:35
  |
2 |     let first = String::from("Ferris");
  |         ----- move occurs because `first` has type `String`, which does not implement the `Copy` trait
3 |     let full = add_suffix(first);
  |                           ----- value moved here
4 |     println!("{full}, originally {first}"); // first is now used here
  |                                   ^^^^^ value borrowed here after move
```

让我们细看看这个错误。Rust 说在第 3 行调用 `add_suffix(first)` 的时候，`first` 已经被转移了。这个错误说 `first` 之所以被转移是因为它的类型是 `String`，没有实现 `Copy`。我们稍后会谈论这个 `Copy`，简单来说，你只需要知道，如果把这里的 `String` 换成 `i32` 就不会报这个错误了。最后，这个错误说我们在 `first` 被转移（其实是“借用”，我们会在下一部分讨论这个）后还使用了它。

所以，如果你转移了一个变量，Rust 就不让你用它了。更准确来说，编译器遵守如下原则：

> **堆数据转移原则**：如果一个变量 `x` 把堆数据的所有权转移给了另一个变量 `y`，那转移之后 `x` 就不能被使用了。

现在你应该明白所有权、转移和安全之间的关系了。转移堆数据的所有权避免了读取被释放内存的这种未定义行为。

### 不想转移就拷贝

其中一种避免转移数据的方法是使用 `.clone()` 方法来拷贝数据。比如说，我们可以用拷贝来修复上一个程序的安全问题：

```rust
fn main() {
    let first = String::from("Ferris");
    let first_clone = first.clone(); // L1
    let full = add_suffix(first_clone); // L2
    println!("{full}, originally {first}");
}

fn add_suffix(mut name: String) -> String {
    name.push_str(" Jr.");
    name
}
```

在 L1，`first_clone` 不只是浅拷贝了 `first` 的指针，而是深拷贝了字符串的数据到一块新的堆内存上。因此在 L2，当 `first_clone` 因为调用 `add_suffix` 而被转移并失效之后，原来的 `first` 并没有变。继续使用 `first` 是安全的。

### 总结

所有权主要是一种堆管理机制。[^2]

- 所有的堆数据只能被一个变量拥有。
- 当一块堆数据的所有者离开作用域后，这块堆数据就被释放了。
- 所有权可以通过转移而改变，比如在变量赋值或者函数调用时。
- 堆数据只能被它的当前所有者拥有，上一任所有者不行。

我们不仅着重介绍了 Rust 的安全机制是 *如何* 工作的，也解释了 *为什么* 它能避免未定义行为。如果你的 Rust 编译器报了一个错，而你不理解为什么会有这个错的话，你会很容易不知所措。上面这些基本概念应该可以帮助你理解 Rust 的错误信息。它们也应该能帮助你设计更符合 Rust 理念的 API。

## 引用和借用

所有权，box 和转移提供了一个使用堆进行安全编程的基础。然而，只能转移的话，挺不方便的。比如说，你想读取某个字符串两遍：

```rust
fn main() {
    let m1 = String::from("Hello");
    let m2 = String::from("world");
    greet(m1, m2);
    let s = format!("{} {}", m1, m2); // Error: m1 and m2 are moved
}

fn greet(g1: String, g2: String) {
    println!("{} {}!", g1, g2);
}
```

> 上述代码无法编译
{: .prompt-warning}

在这个例子中，调用 `greet` 把数据从 `m1` 和 `m2` 转移到了 `greet` 的参数中。两个字符串都在 `greet` 的结尾被释放了，因此就不能再在 `main` 中被使用了。如果我们尝试使用 `format!(..)` 这样的操作去读取它们，就会引发一个未定义行为。因此 Rust 编译器拒绝编译该程序，并报出一个跟上一节一样的错误：

```console
error[E0382]: borrow of moved value: `m1`
 --> test.rs:5:30
 (...rest of the error...)
```

这个转移操作实在是太不方便了。程序经常会需要用到一个字符串两次以上，一种解决办法是我们可以让 `greet` 返回字符串的所有权，像这样：

```rust
fn main() {
    let m1 = String::from("Hello");
    let m2 = String::from("world");
    let (m1_again, m2_again) = greet(m1, m2);
    let s = format!("{} {}", m1_again, m2_again);
}

fn greet(g1: String, g2: String) -> (String, String) {
    println!("{} {}!", g1, g2);
    (g1, g2)
}
```

然而，这种形式的程序有点冗余了。Rust 提供了一种更加简便地读取和写入的操作方式：引用。

### 引用是不拥有数据的指针

**引用** 是一种指针。下面这个例子使用引用来重写我们的 `greet` 程序，让其看起来更加简洁：

```rust
fn main() {
    let m1 = String::from("Hello");
    let m2 = String::from("world"); // L1
    greet(&m1, &m2); // L3 // note the ampersands
    let s = format!("{} {}", m1, m2);
}

fn greet(g1: &String, g2: &String) { // note the ampersands
    // L2
    println!("{} {}!", g1, g2);
}
```

表达式 `&m1` 使用与操作符创建了一个对 `m1` 的引用（或者说“借用”）。`greet` 的参数 `g1` 的类型改为了 `&String`，意思是“这是一个对 `String` 的引用”。

观察 L2，从 `g1` 到字符串 `"Hello"` 有两步。`g1` 是一个指向栈上的 `m1` 的引用，而 `m1` 是一个字符串 box，指向了堆上的 `"Hello"`。

当 `m1` 拥有堆上的数据 `"Hello"` 的时候，`g1` 既 *不* 拥有 `m1` 也 *不* 拥有 `"Hello"`。因此当 `greet` 结束之后，程序到达了 L3 的位置，没有堆数据被释放。只有 `greet` 的栈帧没了。这个现象与我们的 *Box 的释放原则* 相符。因为 `g1` 不拥有 `"Hello"`，所以 Rust 不会因为 `g1` 而释放 `"Hello"`。

引用是 **不拥有数据的指针**，因为它们不拥有它们所指向的数据。

### 解引用一个指针来获取它的数据

前一个例子没有展示 Rust 是如何追踪一个指针到它指向的数据的。比如说，这个 `println!` 宏莫名其妙地可以同时适用于字符串本身 `String` 和对字符串的引用 `&String`。这个底层机制是 **解引用** 操作，写作星号 `*`。比如说，这是一个用各种不同方式使用解引用的程序：

```rust
let mut x: Box<i32> = Box::new(1);
let a: i32 = *x;         // *x reads the heap value, so a = 1
*x += 1;                 // *x on the left-side modifies the heap value,
                         //     so x points to the value 2

let r1: &Box<i32> = &x;  // r1 points to x on the stack
let b: i32 = **r1;       // two dereferences get us to the heap value

let r2: &i32 = &*x;      // r2 points to the heap value directly
let c: i32 = *r2;    // so only one dereference is needed to read it
```

注意看，`r1` 指向的是栈上的 `x`，而 `r2` 指向的是堆上的数值 `2`。

如果写 Rust 代码的话你大概率是见不到太多解引用操作符的。Rust 会在一些特定场合下隐式地插入解引用或者引用，比如说当你用点操作符调用一个方法的时候。举个例子，下面这个程序展示了两种等同的调用 `i32:abs` 和 `str::len` 的方法：

```rust
let x: Box<i32> = Box::new(-1);
let x_abs1 = i32::abs(*x); // explicit dereference
let x_abs2 = x.abs();      // implicit dereference
assert_eq!(x_abs1, x_abs2);

let r: &Box<i32> = &x;
let r_abs1 = i32::abs(**r); // explicit dereference (twice)
let r_abs2 = r.abs();       // implicit dereference (twice)
assert_eq!(r_abs1, r_abs2);

let s = String::from("Hello");
let s_len1 = str::len(&s); // explicit reference
let s_len2 = s.len();      // implicit reference
assert_eq!(s_len1, s_len2);
```

这个例子展示了三种隐式转换：

1. 函数 `i32::abs` 期望得到一个 `i32` 类型的输入，如果想用 `Box<i32>` 调用 `abs`，你需要显式地解引用，像 `i32::abs(*x)`。你也可以通过调用方法函数的语法 `x.abs()` 来隐式地解引用。这个点语法是一种函数调用的语法糖。
2. 这种隐式转换也适用于多层指针。比如说用 `r: &Box<i32>` 调用 `abs` 会插入两次解引用。
3. 这种隐式转换也适用于相反的方向。函数 `str::len` 期望一个 `&str` 的引用类型。如果你在一个有所有权的 `String` 上调用 `len`，Rust 会插入一个借用操作符。（当然，这里还有一层从 `String` 到 `str` 的转换）

我们会在接下来的几章中继续讨论更多关于方法调用和隐式转换的内容。就目前而言，需要记住的就是这种转换会发生在方法调用和宏调用时。我们想要揭开 Rust 的这层面纱，以对 Rust 是如何运作的有一个更清晰的构想模型。

### Rust 禁止同时拥有别名和进行更改

指针是一种非常强大也非常危险的特性，因为它支持别名（aliasing）。别名是指通过不同的变量获取同一个数据。仅对于它自己而言，别名是无害的。但跟 **更改（mutation）** 结合在一起，那可就遭老罪了。一个变量可以用多种方式“搞死”其他的变量，比如：

1. 释放所指向的数据，让其他别名指向一块已经释放的内存。
2. 改变所指向的数据，搞乱其他变量所期望的运行时属性。
3. 多变量同时更改所指向的数据，造成一种不可预知结果的数据更改竞赛。

接下来的例子里，我们要来看一个使用了向量数据结构 `Vec` 的程序。跟固定长度的数组不同，向量结构在堆中存储数据的长度是可变的。比如说，`Vec::push` 可以往向量的结尾添加一个值：

```rust
let mut v: Vec<i32> = vec![1, 2, 3];
v.push(4);
```

宏 `vec!` 创建了一个向量，包含了方括号内的值。向量 `v` 的类型是 `Vec<i32>`。`<i32>` 的意思是这个向量中的值的类型是 `i32`。

有一个重要的细节是 `v` 在堆上申请了一块固定 **容量** 的数组。注意看这个向量的长度是 3，容量也是 3。向量的容量满了。所以当我们执行 `push` 操作的时候，向量就需要申请一块更大一点的内存，然后把所有的值都拷贝过去，再把原来的堆内存释放掉。

说回内存安全问题，让我们把引用加进来。比如说我们创建了一个对向量堆数据的引用，然后这个引用在 `push` 之后就无效了，模拟如下：

```rust
let mut v: Vec<i32> = vec![1, 2, 3];
let num: &i32 = &v[2]; // L1
v.push(4); // L2
println!("Third element is {}", *num); // L3
```

> 上述代码无法编译
{: .prompt-warning}

最开始，`v` 指向了一个堆上的长度为 3 的数组。然后创建了一个变量 `num` 引用了数组的第三个值，这是 L1 发生的事。然而，`v.push(4)` 的操作调整了 `v` 的大小。这个调整使得之前的数组内存被释放了，然后申请了一块新的更大的数组。因此在 L3，解引用 `*num` 读取的是一块非法内存，会造成未定义行为。

> **指针安全原则**：数据不应该同时拥有别名和进行更改。

数据可以有别名。数据也可以被更改。但数据不能 *同时* 有别名和被更改。比如说，Rust 禁止 Box（拥有数据的指针）拥有别名。当把一个 box 从一个变量赋值给另一个变量的时候，所有权也会转移，这使得之前的变量无效了。被拥有的数据只能由其所有者获取，其他别名不行。

然而，因为引用是不拥有数据的指针，它们需要一些跟 box 不同的规定来保证 *指针安全原则*。在设计上，引用意味着临时创建别名。在后面的部分中，我们会解释一些 Rust 如何通过 **借用检查器** 来确保引用安全的基本内容。

### 引用会更改左值的权限

借用检查器的核心概念是变量有三种操作它们的数据的 **权限**：

- **读取权（R）**：数据可以被拷贝到另一个位置，
- **写入权（W）**：数据可以被原地修改。
- **所有权（O）**：数据可以被移动或释放。

这些权限不是运行时存在的，只在编译时才有。它们描述了在程序执行前，编译器如何思考你的程序。

默认情况下，一个变量对其数据拥有 RO 权限，如果一个变量是用 `let mut` 声明的，那它也有 W 权限。关键的一点是，**引用可以临时移除这些权限**。

要理解上述概念，让我们来看一下上例的一个安全版本的代码中的权限，这个例子中 `push` 被移动到了 `println!` 之后。

```rust
let mut v: Vec<i32> = vec![1, 2, 3];
let num: &i32 = &v[2];
println!("Third element is {}", *num);
v.push(4);
```

让我们一行一行看：

1. 在 `let mut v = (...)` 之后，变量 `v` 初始化，拥有 RWO 权限。
2. 在 `let num = &v[2]` 之后，`v` 的数据被 `num` **借** 走了。这一步中发生了三件事：
    - 这次借用移除了 `v` 的 WO 权限，`v` 不能再被修改或者移动了，但是依然可读。
    - 变量 `num` 获得了 RO 权限，它是不可改的，因为没有用 `let mut` 声明。
    - **左值** `*num` 有 R 权限。
3. 在 `println!(...)` 之后，`num` 不再使用了，因此 `v` 不再被借用了，因此：
    - `v` 重新获得了 WO 权限。
    - `num` 和 `*num` 失去了它们的权限。
4. 在 `v.push(4)` 之后，`v` 也不再使用了，也失去了它的所有权限。

下一步，让我们甚久一点细节。首先，我们为什么要同时考虑 `num` 和 `*num`？因为通过引用获取数据和操作引用本身是不一样的。比如说，我们用 `let mut` 声明一个指向数字和引用：

```rust
let x = 0;
let mut x_ref = &x;
```

注意，`x_ref` 有 W 权限，而 `*x_ref` 没有。这就是说，我们可以把 `x_ref` 赋值给另一个不同的引用（比如 `x_ref = &y`），但我们不能更改被指向的数据（比如 `*x_ref += 1`）。

换句话说，权限是设定在 **左值** 而不是变量上的。左值就是任何你可以放在赋值操作左边的东西，比如说：

- 变量，像 `a`
- 解引用，像 `*a`
- 数组取值，像 `a[0]`
- 取元素，像元组的 `a.0` 或者结构的 `a.field`
- 以上内容的随意结合，像 `*((*a)[0].1)`

其次，为什么左值再不被使用后就失去了权限？因为有些权限相互排斥。如果 `num = &v[2]`，那么在 `num` 被使用期间 `v` 就不能被修改或者释放。但是这不代表我们只能用 `num` 一点点时间。如果我们在上一段程序中添加一个 `println`，`num` 就能再多活一会：

```rust
let mut v: Vec<i32> = vec![1, 2, 3];
let num: &i32 = &v[2];
println!("Third element is {}", *num);
println!("Again, the third element is {}", *num);
v.push(4);
```

### 借用检查器能找到权限违规行为

再回想一下 *指针安全原则*：数据不能同时有别名和被修改。这三种权限的目的就是确保数据如果有别名，就不能被修改。对一个数据创建引用（借用）会导致数据暂时性的变为只读状态，直到引用不再被使用为止。

Rust 在 **借用检查器** 中使用这些权限。借用检查器寻找那些涉及引用的可能会不安全的操作。让我们回到我们之前看的那个不安全的程序：

```rust
let mut v: Vec<i32> = vec![1, 2, 3];
let num: &i32 = &v[2];
v.push(4);
println!("Third element is {}", *num);
```

每当一个左值被使用时，Rust 期望这个左值能有满足其行为的权限。比如说，`&v[2]` 需要 `v` 是可读的。与之相对的，`v.push(4)` 这个更改操作需要 `v` 同时是可读和可写的。然而，此时 `v` 并没有写权限，因为 `num` 借用了它。

如果你尝试编译这段程序，Rust 编译器会报如下错误：

```text
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> test.rs:4:1
  |
3 | let num: &i32 = &v[2];
  |                  - immutable borrow occurs here
4 | v.push(4);
  | ^^^^^^^^^ mutable borrow occurs here
5 | println!("Third element is {}", *num);
  |                                 ---- immutable borrow later used here
```

错误信息说，引用 `num` 还在使用中时，`v` 不能被更改。这是表层的原因，底层的原因是 `push` 可能会使 `num` 指向无效内存。Rust 发现了这个可能会违反内存安全的问题。

### 可变引用提供了唯一获取数据但不拥有数据的权限

目前为止我们看到的引用都是只读 **不可变引用**（也叫 **共享引用**）。不可变引用允许有别名但不允许更改。然而，有时候临时改一下数据，但不移动它，也是需要的。

这种技术叫 **可变引用**（也叫 **唯一引用**）。这有一个关于可变引用的简单例子：

```rust
let mut v: Vec<i32> = vec![1, 2, 3];
let num: &mut i32 = &mut v[2];
*num += 1;
println!("Third element is {}", *num);
println!("Vector is now {:?}", v);
```

可变引用用 `&mut` 操作符创建。`num` 的类型是 `&mut i32`。与不可变引用相比，你可以看到在权限方面有两大不同：

1. 当 `num` 是不可变引用时，`v` 依然有 R 权限，当 `num` 是一个可变引用时，`v` 就啥权限都没了（在 `num` 可用期间）。
2. 当 `num` 是不可变引用时，`*num` 只有 R 权限，当 `num` 是可变引用时，`*num` 还有 W 权限。

第一个项就是保证可变引用 *安全* 的方法。可变引用允许修改但不允许别名。`v` 被借走后临时不可用了，因此也就起不到别名的作用了。

第二项保证了让可变引用 *有用*。`v[2]` 可以通过 `*num` 更改。比如说 `*num += 1` 会更改 `v[2]`。注意，`*num` 有 W 权限，但 `num` 没有。也就是说，`num` 不能再被赋值给另一个可变引用。

可变引用也可以被临时“降级”为一个只读引用，比如：

```rust
let mut v: Vec<i32> = vec![1, 2, 3];
let num: &mut i32 = &mut v[2];
let num2: &i32 = &*num;
println!("{} {}", *num, *num2);
```

在这个程序里，`&*num` 这个借用移除了 `*num` 的 W 权限但保留了 R 权限，所以 `println!(..)` 可以同时读取 `*num` 和 `*num2`。

### 权限在引用的生命周期的最后被返还

我们前面说，引用会在它被“使用”期间更改权限。这个“使用”指的是一个引用的 **生命周期**，或者说是这个变量从创建到最后一次使用的这个代码范围。

比如说，在这段程序中，`y` 的生命周期从 `let y = &x` 开始，到 `let z = *y` 结束：

```rust
let mut x = 1;
let y = &x;
let z = *y;
x += z;
```

`y` 生命周期结束后，`x` 的 W 权限就被还回来了。

前面我们说，生命周期是一段连续的代码区域，但是，在我们介绍了控制流之后，事情就不一定了。比如说，下面这个函数会把一个 ASCII 字符的向量数组中的第一个字母大写：

```rust
fn ascii_capitalize(v: &mut Vec<char>) {
    let c = &v[0];
    if c.is_ascii_lowercase() {
        let up = c.to_ascii_uppercase();
        v[0] = up;
    } else {
        println!("Already capitalized: {:?}", v);
    }
}
```

变量 `c` 在不同的分支上有不同的生命周期。在 `if` 分支中，`c` 在 `c.to_ascii_uppercase()` 中被使用了，因此 `*v` 在这一行之前都没有 W 权限。

但是在 `else` 分支中，`c` 没有使用，`*v` 在程序进去 `else` 分支块后立即就重获了 W 权限。

### 数据必须比它所有的引用都活得长

作为 *指针安全原则* 的一部分，借用检查器确保 **数据比它所有的引用都活得长**。Rust 用两种方式确保此事。第一种处理在一个函数的作用域内创建和释放的引用。比如，我们尝试释放一个有引用的字符串：

```rust
let s = String::from("Hello world");
let s_ref = &s;
drop(s);
println!("{}", s_ref);
```

> 上述代码不能编译
{: .prompt-warning}

要发现这样的错误，Rust 使用我们前面讨论的权限机制。`&s` 这个借用移除了 `s` 的 O 权限，然后 `drop` 需要有 O 权限，这就不匹配了。

这里的关键点是，Rust 知道 `s_ref` 活了多长。但是如果 Rust 不知道，它就需要另一种机制来确保此事了。特别是当一个引用既不是函数的输入也不是函数的输出时。比如说，下面是一个返回一个向量第一个元素的安全函数：

```rust
fn first(strings: &Vec<String>) -> &String {
    let s_ref = &strings[0];
    s_ref
}
```

这里有一种新的权限，F 权限。当一个表达式使用了一个输入的引用（比如 `&string[0]`）或者返回了一个引用（比如 `return s_ref`）时，F 权限必须存在。

不同于 RWO 权限，F 在函数体内不会发生改变。如果一个引用被允许在一个特定的表达式中使用，那么它就有 F 权限。比如说，我们把 `first` 函数改为一个新函数 `first_or`，接收一个 `default` 参数：

```rust
fn first_or(strings: &Vec<String>, default: &String) -> &String {
    if strings.len() > 0 {
        &strings[0]
    } else {
        default
    }
}
```

> 上述代码不能编译
{: .prompt-warning}

这个函数不能编译，因为表达式 `&string[0]` 和 `default` 缺少用来返回的 F 权限。但是为什么？Rust 给出了如下错误：

```console
error[E0106]: missing lifetime specifier
 --> test.rs:1:57
  |
1 | fn first_or(strings: &Vec<String>, default: &String) -> &String {
  |                      ------------           -------     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `strings` or `default`
```

这里的“缺少生命周期指示符”可能有点让人困惑，不过下面的帮助信息倒很有用。如果 Rust 只看这个函数签名的话，它不知道返回的 `&String` 是 `strings` 的引用还是 `default` 的引用。要理解为什么这个很关键，我们这样：

```rust
fn main() {
    let strings = vec![];
    let default = String::from("default");
    let s = first_or(&strings, &default);
    drop(default);
    println!("{}", s);
}
```

如果程序允许 `default` 返回，那么它就不安全。就跟上例一样，`drop` 会让 `s` 指向无效内存。Rust 只有确定了 `default` *绝对* 不会被返回，才能允许这个程序编译。

要指定 `default` 是否能被返回，Rust 提供了一种叫 *生命周期参数* 的机制。我们会在第 10 章介绍这个功能。现在的话，我们只需要知道：

1. 输入和输出引用跟函数体内的引用不一样；
2. Rust 使用 F 权限检查这些引用是否安全。

我们再看一下另一种情境下的 F 权限，比如说你想返回一个栈中变量的引用：

```rust
fn return_a_string() -> &String {
    let s = String::from("Hello world");
    let s_ref = &s;
    s_ref
}
```

> 上述代码不能编译
{: .prompt-warning}

上述程序不安全，因为当函数返回后，`&s` 就无效了。Rust 会以与之前类似的错误拒绝编译程序。现在你知道了，这个错误指的是 `s_ref` 缺少 F 权限。

### 总结

引用提供了一种不转移数据所有权的同时还能读写数据的能力。引用通过借用创建（`&` 和 `&mut`），通过解引用（`*`）使用（通常是隐式的）。

然后，引用也很容易被误用。Rust 的借用检查器通过一系列权限确保引用都被安全使用：

- 所有变量都能读、拥有数据，有时能写数据。
- 创建引用会把原左值的权限转移到引用上。
- 引用的生命周期结束后权限会被返还。
- 数据必须比它所有的引用都活得长。

在这一节，你可能会感觉我们讨论了更多关于 Rust *不能* 做什么得内容。没错，就是这样。Rust 的一个核心特性就是允许你在没有垃圾回收机制的情况下使用指针，同时还能避免未定义行为。现在理解这些安全规则可以帮助你之后更好地理解编译器的报错。

## 解决所有权错误

学会如何解决所有权错误是 Rust 的一项核心技能。当借用检查器拒绝编译你的代码时，你该怎么做？在这一章节，我们会讨论几个常见的所有权错误的案例。每一个案例分析都会包含一个无法通过编译的函数，然后我们会解释为什么 Rust 拒绝编译这个函数，并且给出几种解决的方式。

一个常见的情况是理解一个函数 *实际上* 是否是安全的。Rust 总是会拒绝编译不安全的程序[^3]。但是有时候，Rust 也会拒绝安全的程序。这些案例分析会向你展示如何处理这两种情况。

### 修正不安全程序：返回栈上的引用

我们的第一个案例是关于返回栈上的引用的，就像我们在“[数据必须比它所有的引用都活得长](#数据必须比它所有的引用都活得长)”那一节中讨论的那样。下面是这个函数：

```rust
fn return_a_string() -> &String {
    let s = String::from("Hello world");
    &s
}
```

> 上述代码无法编译
{: .prompt-warning}

在思考如何修正这个函数之前，我们要先思考：**为什么这个程序是不安全的**？这里问题的关键在于被引用数据的生命周期上。如果你传递了一个字符串的引用，那你就得保证这个字符串活得得比这个引用长。

根据不同的情况，有四种方式可以延长字符串的生命周期。第一种是把字符串的所有权转交到函数之外，即把 `&String` 改为 `String`：

```rust
fn return_a_string() -> String {
    let s = String::from("Hello world");
    s
}
```

第二种是返回一个字符串字面量，这种数据会永远存活（用 `'static` 声明）。当我们不打算修改这个字符串时，也没必要申请堆内存，就可以用这种方式：

```rust
fn return_a_string() -> &'static str {
    "Hello world"    
}
```

第三种是使用垃圾回收把借用检查推迟到运行时。比如说，你可以使用[引用计数指针](https://doc.rust-lang.org/std/rc/index.html)：

```rust
use std::rc::Rc;
fn return_a_string() -> Rc<String> {
    let s = Rc::new(String::from("Hello world"));
    Rc::clone(&s)
}
```

我们会在第 15 章 [`Rc<T>` 引用计数智能指针](https://rust-book.cs.brown.edu/ch15-04-rc.html)讨论更多关于引用计数的话题。简单来说，`Rc::clone` 只拷贝了指向 `s` 的指针，而没有拷贝数据本身。在运行时，`Rc` 会在最后一个指向数据的 `Rc` 被释放后释放数据。

第四种是让调用者提供一个使用可变引用的字符串“槽”：

```rust
fn return_a_string(output: &mut String) {
    output.replace_range(.., "Hello world");
}
```

使用这种方式，调用者需要负责为字符串申请空间。这种方式可能会有点繁琐，但是如果调用者需要仔细控制什么时候发生内存申请，那么这种方式可能会更有效。

上述四种方式哪种最合适，会根据背景的不同而不同。但是关键的一点是识别出在这个表面的所有权错误背后的根本问题。我的字符串应该存活多久？谁应该负责释放它？当你对这些问题有一个清晰的答案时，剩下的就是选择什么 API 来解决问题了。

### 修正不安全程序：权限不足

另一个常见的问题是试图修改一个只读数据，或者尝试在引用后释放数据。比如说，我们想要写一个 `stringify_name_with_title` 的函数，这个函数应该使用一个名称向量数组创建一个全名，外加一个头衔：

```rust
fn stringify_name_with_title(name: &Vec<String>) -> String {
    name.push(String::from("Esq."));
    let full = name.join(" ");
    full
}

// ideally: ["Ferris", "Jr."] => "Ferris Jr. Esq."
```

> 上述代码无法编译
{: .prompt-warning}

编译器会拒绝编译这个程序，因为 `name` 是一个不可变引用，但是 `name.push(..)` 需要有 W 权限。这个程序不安全，是因为 `push` 可能会导致该函数之外的 `name` 的其它引用无效化，像这样：

```rust
fn main() {
    let name = vec![String::from("Ferris")];
    let first = &name[0];
    stringify_name_with_title(&name);
    println!("{}", first);
}
```

> 上述代码无法编译
{: .prompt-warning}

在这个例子中，`first` 对 `name[0]` 的引用是在 `stringify_name_with_title` 函数调用之前创建的，函数 `name.push(..)` 会重新申请 `name` 数据的内存，就会导致 `first` 无效化，导致 `println` 试图读取一块已释放的内存。

所以我们该怎么改？其中一种方式是把参数的类型从 `&Vec<String>` 改为 `&mut Vec<String>`：

```rust
fn stringify_name_with_title(name: &mut Vec<String>) -> String {
    name.push(String::from("Esq."));
    let full = name.join(" ");
    full
}
```

但这并不是一个好办法！**函数不应该在调用者不知晓的情况下修改其输入**。一个调用 `stringify_name_with_title` 的人大概不希望这个函数修改了他的向量数组。如果有另一个函数叫 `add_title_to_name` 可能会修改输入，但这不是我们要讨论的情况。

第二种方法是转移参数的所有权，即把 `&Vec<String>` 改为 `Vec<String>`：

```rust
fn stringify_name_with_title(mut name: Vec<String>) -> String {
    name.push(String::from("Esq."));
    let full = name.join(" ");
    full
}
```

但这也不是一个好办法！**很少会有 Rust 函数接管像 `Vec` 和 `String` 这样的堆数据结构的所有权**。上述函数会使得输入的 `name` 不可再使用，这种情况对于调用者来说是挺烦人的，我们在[引用和借用](#引用和借用)的开头讨论过这一点了。

所以说，使用 `&Vec` 其实是对的，这一点我们 *不* 想改变。相反，我们修改函数体本身。根据内存使用多少的不同，会有很多可行的办法，其中一种是把输入 `name` 拷贝一份：

```rust
fn stringify_name_with_title(name: &Vec<String>) -> String {
    let mut name_clone = name.clone();
    name_clone.push(String::from("Esq."));
    let full = name_clone.join(" ");
    full
}
```

通过拷贝 `name`，我们就可以修改本地拷贝的向量数组了，然而这样拷贝的话会把输入的每一个字符串都拷贝。我们可以通过之后添加后缀的方式避免不必要的拷贝：

```rust
fn stringify_name_with_title(name: &Vec<String>) -> String {
    let mut full = name.join(" ");
    full.push_str(" Esq.");
    full
}
```

总的来说，编写 Rust 函数需要非常认真地权衡如何给予 *合适* 的权限。对这个例子而言，我们通常希望参数 `name` 只有读权限。

### 修正不安全程序：对一个数据结构同时创建别名和进行修改

另一种不安全的操作是使用一个被其他别名释放的堆内存的引用。比如说，下面的函数会获取一个向量数组中最大的字符串的引用，然后在更改向量数组的同时还使用这个引用：

```rust
fn add_big_strings(dst: &mut Vec<String>, src: &[String]) {
    let largest: &String = 
      dst.iter().max_by_key(|s| s.len()).unwrap();
    for s in src {
        if s.len() > largest.len() {
            dst.push(s.clone());
        }
    }
}
```

> 这个例子使用了迭代器和闭包来简洁地找出最大字符串的引用。我们会在之后的章节中讨论这些功能，现在我们只会提供一些对这些功能的基本理解。
{: .prompt-tip}

这个程序会被借用检查器拒绝，因为 `let largest = ..` 移除了 `dst` 的 W 权限。然而 `dst.push(..)` 需要有 W 权限。这里我们又要问：**为什么这个程序不安全**？因为 `dst.push(..)` 会重新申请 `dst` 的内存，使得 `largest` 的引用无效化。

要解决这个程序，关键点在于我们要缩短 `largest` 的生命周期，以便不跟 `dst.push(..)` 重叠。其中一种方案是拷贝 `largest`：

```rust
fn add_big_strings(dst: &mut Vec<String>, src: &[String]) {
    let largest: String = dst.iter().max_by_key(|s| s.len()).unwrap().clone();
    for s in src {
        if s.len() > largest.len() {
            dst.push(s.clone());
        }
    }
}
```

然而，这可能会因为申请内存和拷贝字符串数据而造成性能问题。

另一种解决办法是先比较所有的长度，然后再修改 `dst`：

```rust
fn add_big_strings(dst: &mut Vec<String>, src: &[String]) {
    let largest: &String = dst.iter().max_by_key(|s| s.len()).unwrap();
    let to_add: Vec<String> = 
        src.iter().filter(|s| s.len() > largest.len()).cloned().collect();
    dst.extend(to_add);
}
```

然而这也会因为申请 `to_add` 向量而造成性能问题。

最终的解决办法是只拷贝 `largest` 的长度，因为我们并不需要 `largest` 的内容。这个方案是最常用且性能最高的：

```rust
fn add_big_strings(dst: &mut Vec<String>, src: &[String]) {
    let largest_len: usize = dst.iter().max_by_key(|s| s.len()).unwrap().len();
    for s in src {
        if s.len() > largest_len {
            dst.push(s.clone());
        }
    }
}
```

这些解决办法都有一个共同点：缩短 `dst` 借用的生命周期，以便不跟修改 `dst` 时重叠。

---

[^1]: 这些数据结构并不是使用字面上的 `Box` 类型。比如 `String` 是用 `Vec` 实现的，而 `Vec` 是用 `RawVec` 实现的，而不是 `Box`。但是像 `RawVec` 这种类型也与 box 类似，它们也在堆上拥有内存。

[^2]: 从另一方面说，所有权是 *指针* 管理的机制。但是我们还没有讲到如何在堆之外的地方获取指针。我们会在下一部分讨论。

[^3]: 这里说的是用 Rust 的“安全子集”编写的程序。如果你使用了 `unsafe` 代码或者引用了不安全的组件（比如说调用了 C 库），你必须采取额外的办法来避免未定义行为。

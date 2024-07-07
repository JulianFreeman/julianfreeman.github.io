---
title: Rust 处理文件
date: 2024-07-06 21:43:00 -0400
categories: [学习笔记, Rust]
tags: [rust, file]
description: 本文记录一些关于使用 Rust 处理文件的基础内容
---

## 读取二进制文件

`std::fs::read` 函数可以按照二进制读取整个文件。

```rust
use std::fs;

fn read_bytes() {
    let data: Vec<u8> = fs::read("./rsc/sheet.png").unwrap();
    let mark: String = String::from_utf8(data[1..4].to_vec()).unwrap();
    println!("the mark: {}", mark);
}
```

```console
the mark: PNG
```

## 读取文本文件

`std::fs::read_to_string` 函数可以按照文本读取文件。

```rust
use std::fs;

fn read_string() {
    let data: String = fs::read_to_string("./rsc/demo.txt").unwrap();
    println!("first word: {}", data[0..6].to_string());
}
```

```console
first word: Lorem
```

## 打开文件并读取

```rust
use std::fs::File;
use std::io::Read;

fn open_and_read() {
    let mut f: File = File::open("./rsc/demo.txt").unwrap();
    let mut s: String = String::new();
    f.read_to_string(&mut s).unwrap();
    println!("first word: {}", s[0..6].to_string());
}
```

`File::read_to_string` 的签名是

```rust
fn read_to_string(&mut self, buf: &mut String) -> io::Result<usize>
```
{: .nolineno}

所以变量 `f` 需要声明为可变的，因为该函数的第一个参数是 `&mut self`。

## 创建并写入

```rust
use std::fs::File;
use std::io::Write;

fn create_and_write() {
    let mut f: File = File::create("./rsc/test.txt").unwrap();
    let s: &str = "Hello world";
    f.write_all(s.as_bytes()).unwrap();
}
```

`write` 和 `write_all` 的区别是，前者不一定会一次性写入指定的所有内容，而后者一定会全部写入所有内容。

因为往硬盘写入内容都是有缓冲的，有时候缓冲没满就不一定会写入，后者则强制全部写入为止。


## OpenOptions 提供更多选项

```rust
use std::fs;
use std::fs::File;
use std::io::Read;

fn advance() {
    let mut file: File = fs::OpenOptions::new()
        .read(true)
        .write(true)
        .create(true)
        .open("./rsc/demo.txt").unwrap();
    let mut s: String = String::new();
    file.read_to_string(&mut s).unwrap();
    println!("first word: {}", s[0..6].to_string());
}
```

上述表示以读写模式打开文件，如果不存在就创建。详见 [文档](https://doc.rust-lang.org/std/fs/struct.OpenOptions.html)。

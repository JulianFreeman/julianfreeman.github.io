---
title: AWK 命令
date: 2023-12-04 23:32:00 -0500
categories: [翻译, AWK]
tags: [linux, awk]
description: 本文是 GeeksforGeeks 网站关于 AWK 的一篇文章的翻译
---

原文：https://www.geeksforgeeks.org/awk-command-unixlinux-examples/

## Unix/Linux 中的 AWK

AWK 是一个用来处理数据和生成报告的脚本语言。AWK 命令程序语言不需要编译，同时允许用户使用变量、数字函数、字符串函数和逻辑操作符。

AWK 是一个辅助工具，旨在帮助程序员编写小巧但有效的声明式程序，通过定义文本模式搜索一个文件中的每一行内容，并对一行中匹配成功的内容应用某种行为。AWK 最常用于模式扫描和处理。它可以扫描一个或多个文件中的每一行并寻找匹配特定模式的行并应用对应的行为。

AWK 这个名字取自它的开发者：Aho，Weinberger，和 Kernighan。

### AWK 能干什么？

#### 1. AWK 操作：

- 逐行扫描文件
- 把每一行分割成段
- 用模式匹配行/段
- 对匹配的行应用行为

#### 2. 目的：

- 转换数据文件
- 生成格式化的报告

#### 3. 编程结构

- 格式化输出行
- 算数和字符串操作
- 条件和循环

#### 语法

```shell
awk options 'selection_criteria {action}' input-file > output-file
```
{: .nolineno}

#### 选项

`options`

```plaintext
-f program-file: 从文件中读取 AWK 源程序，而非命令行的第一个参数
-F fs: 把 fs 作为输入的段分隔符
```

### 示例

下面的示例中将使用如下文件作为输入文件：

```plaintext
julian@ubuntu2204:~/temp$ cat employee.txt 
ajay manager account 45000
sunil clerk account 25000
varun manager sales 50000
amit manager account 47000
tarun peon sales 15000
deepak clerk sales 23000
sunil peon sales 13000
satvik director purchase 80000
```

#### 1. AWK 的默认行为

默认情况下 AWK 会打印文件的每一行数据

```plaintext
julian@ubuntu2204:~/temp$ awk '{print}' employee.txt 
ajay manager account 45000
sunil clerk account 25000
varun manager sales 50000
amit manager account 47000
tarun peon sales 15000
deepak clerk sales 23000
sunil peon sales 13000
satvik director purchase 80000
```

在上例中，没有指定任何模式。所以每一行都被应用了行为。`print` 行为在没有任何参数的情况下会打印一整行，所以这里 `print` 打印了文件的所有行。

#### 2. 打印匹配模式的行

```plaintext
julian@ubuntu2204:~/temp$ awk '/manager/ {print}' employee.txt 
ajay manager account 45000
varun manager sales 50000
amit manager account 47000
```

上例中，AWK 命令打印了所有匹配 `manager` 的行。

#### 3. 将行分成段

对于每一条记录，也就是每一行，AWK 默认会按照空白字符分割并存储在 `$n` 变量中。如果有一行有 4 个单词，那就会分别存在 `$1`，`$2`，`$3` 和 `$4` 中。`$0` 存储的是一整行。

```plaintext
julian@ubuntu2204:~/temp$ awk '{print $1,$4}' employee.txt 
ajay 45000
sunil 25000
varun 50000
amit 47000
tarun 15000
deepak 23000
sunil 13000
satvik 80000
```

在上例中，`$1` 和 `$4` 分别表示名称和薪资段。

##### AWK 的内置变量

AWK 的内置变量包括段变量 `$1`，`$2`，`$3` 等，它们把一行文本分割成独立的部分，称为段。

- `NR`：NR 保存了当前记录的序号，一般来说一条记录就是一行。AWK 每次会对文件中的一条记录匹配模式并应用行为。（Number of Records）
- `NF`：NF 保存了当前记录中的段数。（Number of Fields）
- `FS`：FS 保存了用来分割记录的段分隔符。默认是空白字符，也就是空格和制表符。FS 可以被指定为其他字符（一般在 `BEGIN` 中指定），以此修改段分隔符。（Field Separator）
- `RS`：RS 保存了当前的记录分隔符。因为默认情况下，一行就是一条记录，因此默认的记录分隔符是换行符。（Record Separator）
- `OFS`：OFS 保存了输出段分隔符，用于在 AWK 打印时分隔段。默认是一个空格。当 `print` 有多个用逗号分隔的参数时，AWK 就会在每两个打印的参数值之间插入一个 OFS。（Output Field Separator）
- `ORS`：ORS 保存了输出记录分隔符，用于 AWK 在输出多行时打印。默认是换行符。`print` 会自动在内容的最后添加一个 ORS。（Output Record Separator）

##### 使用内置变量 `NR`

```plaintext
julian@ubuntu2204:~/temp$ awk '{print NR,$0}' employee.txt 
1 ajay manager account 45000
2 sunil clerk account 25000
3 varun manager sales 50000
4 amit manager account 47000
5 tarun peon sales 15000
6 deepak clerk sales 23000
7 sunil peon sales 13000
8 satvik director purchase 80000
```

上例中，有 NR 的 AWK 命令在打印每一行时附带了行号。

##### 使用内置变量 `NF`

```plaintext
julian@ubuntu2204:~/temp$ awk '{print $1,$NF}' employee.txt 
ajay 45000
sunil 25000
varun 50000
amit 47000
tarun 15000
deepak 23000
sunil 13000
satvik 80000
```

上例中，`$1` 代表名称段，`$NF` 代表薪资段，因为 `$NF` 表示的是最后一段。

##### `NR` 的另一种用法

```plaintext
julian@ubuntu2204:~/temp$ awk 'NR==3,NR==6 {print NR,$0}' employee.txt 
3 varun manager sales 50000
4 amit manager account 47000
5 tarun peon sales 15000
6 deepak clerk sales 23000
```

### 更多示例

下面的实例使用如下文件

```plaintext
julian@ubuntu2204:~/temp$ cat geeksforgeeks.txt 
A    B    C
Tarun    A12    1
Man    B6    2
Praveen    M42    3
```

#### 1. 打印行号和每一行的第一个元素并用 `-` 隔开

```plaintext
julian@ubuntu2204:~/temp$ awk '{print NR "- " $1}' geeksforgeeks.txt 
1- A
2- Tarun
3- Man
4- Praveen
```

#### 2. 返回第二列

```plaintext
julian@ubuntu2204:~/temp$ awk '{print $2}' geeksforgeeks.txt 
B
A12
B6
M42
```

#### 3. 打印所有非空行

```plaintext
julian@ubuntu2204:~/temp$ awk 'NF > 0' geeksforgeeks.txt 
A    B    C
Tarun    A12    1
Man    B6    2
Praveen    M42    3
```

如果要打印所有空行外加行号用以区分，则

```plaintext
julian@ubuntu2204:~/temp$ awk 'NF <= 0 {print NR}' geeksforgeeks.txt 
3
6
```

#### 4. 找到长度最长的行

```plaintext
julian@ubuntu2204:~/temp$ awk '{if (length($0)>max) max=length($0)} END {print max}' geeksforgeeks.txt 19
```

#### 5. 计算文件的行数

```plaintext
julian@ubuntu2204:~/temp$ awk 'END {print NR}' geeksforgeeks.txt 
4
```

#### 6. 打印多余 14 个字符的行

```plaintext
julian@ubuntu2204:~/temp$ awk 'length($0)>14' geeksforgeeks.txt 
Tarun    A12    1
Praveen    M42    3
```

#### 7. 在任意列中查找任意字符串

```plaintext
julian@ubuntu2204:~/temp$ awk '{ if($2 == "B6") print $0;}' geeksforgeeks.txt 
Man    B6    2
```

#### 8. 从 1 到 6 打印平方

```plaintext
julian@ubuntu2204:~/temp$ awk 'BEGIN { for(i=1;i<=6;i++) print "square of",i,"is",i*i;}' geeksforgeeks.txt 
square of 1 is 1
square of 2 is 4
square of 3 is 9
square of 4 is 16
square of 5 is 25
square of 6 is 36
```


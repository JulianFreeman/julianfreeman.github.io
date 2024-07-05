---
title: Google 表格公式 - 去重
date: 2023-05-22 21:06:00 -0400
categories: [Goole 表格, 公式笔记]
tags: [google, sheet, formula]
description: 本文介绍数组去重
---

本文将一步步推演一个严谨的数组去重公式。注意，这里说的“数组去重”指的是去掉 **两个** 数组中相同的元素，所以其实更类似于“集合减法”。

样表：

|   | A | B |
|:-:|:-:|:-:|
| 1 | Cat  | Cat    |
| 2 | Dog  | Cheese |
| 3 | Cat  | Dog    |
| 4 | Milk | milk   |
| 5 | cat  |        |
| 6 |      |

## 一个简单的解决方案

在推演严谨（且复杂）的公式之前，先来看看简单的公式怎么解决。

两个数组去重的思路其实很简单，假设有 A 数组和 B 数组两个数组，要检查 B 数组中有哪些元素不在 A 数组中，那就对 B 数组的每一个值在 A 数组中进行查找，找到则有，找不到则无。这个查找的函数当然可以用 `MATCH`，但为了兼顾多种可能，我们可以用 `COUNTIF`，如果结果为 0，则表示该值不在 A 数组中。

`D1` 处填入公式 `=MAP(B1:B, LAMBDA(x, IF(x="", "", COUNTIF(A1:A, x))))`

|   | A | B | C | D |
|:-:|:-:|:-:|:-:|:-:|
| 1 | Cat  | Cat    | | 3 |
| 2 | Dog  | Cheese | | 0 |
| 3 | Cat  | Dog    | | 1 |
| 4 | Milk | milk   | | 1 |
| 5 | cat  |        |
| 6 |      |

看上去还行，不过就是有一个小小的问题：不管是 `MATCH` 还是 `COUNTIF`，比较时都不区分大小写。

## 严格比较

Google 貌似并没有提供一个什么查找函数是支持在比较时区分大小写的。`FIND` 区分大小写，但是它只能在字符串中查找，不能在数组中查找。所以我们只能另辟蹊径了。

`EXACT` 函数是严格比较字符串的，也就是区分大小写的。但这只是一个比较函数，并不是查找函数，毕竟也没个什么其他函数是支持在查找时传入一个比较函数作为参数的，所以要想严格比较并计数，一个思路就是把 `EXACT` 结合 `COUNTIF` 使用。

## 严格比较并计数

我们先把 `D1` 公式换为 `=MAP(A1:A, LAMBDA(x, IF(x="", FALSE, EXACT(x, B1))))`

|   | A | B | C | D |
|:-:|:-:|:-:|:-:|:-:|
| 1 | Cat  | Cat    | | TRUE |
| 2 | Dog  | Cheese | | FALSE |
| 3 | Cat  | Dog    | | TRUE |
| 4 | Milk | milk   | | FALSE |
| 5 | cat  |        | | FALSE |
| 6 |      | | | FALSE |

这里其实就是对 A 组中的每一个值都与 `B1` 的值进行严格比较，得出一个布尔数组，然后计算该数组中 `TRUE` 值的个数。

`E1` 处填入公式 `=COUNTIF(MAP(A1:A, LAMBDA(x, IF(x="", FALSE, EXACT(x, B1)))), TRUE)`

|   | A | B | C | D | E |
|:-:|:-:|:-:|:-:|:-:|:-:|
| 1 | Cat  | Cat    | | TRUE | 2 |
| 2 | Dog  | Cheese | | FALSE |
| 3 | Cat  | Dog    | | TRUE |
| 4 | Milk | milk   | | FALSE |
| 5 | cat  |        | | FALSE |
| 6 |      | | | FALSE |

结果找到 2 个。如果比较的是 `B2` 的值，结果就是 0。

然后，我们对 B 组的每一个值都进行该操作，`F1` 填入公式

`=MAP(B1:B, LAMBDA(y, IF(y="", "", COUNTIF(MAP(A1:A, LAMBDA(x, IF(x="", FALSE, EXACT(x, y)))), TRUE))))`

就出结果了。

|   | A | B | C | D | E | F |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| 1 | Cat  | Cat    | | TRUE | 2 | 2 |
| 2 | Dog  | Cheese | | FALSE | | 0 |
| 3 | Cat  | Dog    | | TRUE | | 1 |
| 4 | Milk | milk   | | FALSE | | 0 |
| 5 | cat  |        | | FALSE |
| 6 |      | | | FALSE |

## 去重

有了上述结果的计数数组，我们就可以用多种方式进行去重了。比如可以使用条件格式高亮标记计数为 0 或者大于 1 的值，或者也可以将上述计数数组作为过滤条件对原数组进行过滤，找出不存在于另一数组的值。

`G1` 处填入公式

`=FILTER(B1:B, MAP(B1:B, LAMBDA(y, IF(y="", FALSE, NOT(COUNTIF(MAP(A1:A, LAMBDA(x, IF(x="", FALSE, EXACT(x, y)))), TRUE))))))`

|   | A | B | C | D | E | F | G |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| 1 | Cat  | Cat    | | TRUE | 2 | 2 | Cheese |
| 2 | Dog  | Cheese | | FALSE | | 0 | milk |
| 3 | Cat  | Dog    | | TRUE | | 1 |
| 4 | Milk | milk   | | FALSE | | 0 |
| 5 | cat  |        | | FALSE |
| 6 |      | | | FALSE |

就找出了 B 组中不存在于 A 组的元素。

因为上述公式有两处用到了范围 `B1:B`，为了避免修改时的麻烦，可以使用 `LET`，将公式改为

`=LET(r, B1:B, FILTER(r, MAP(r, LAMBDA(y, IF(y="", FALSE, NOT(COUNTIF(MAP(A1:A, LAMBDA(x, IF(x="", FALSE, EXACT(x, y)))), TRUE)))))))`

结果是一样的。

要查找 A 组中不存在于 B 组的元素，将两个范围换一下就行，

`H1` 处填入公式

`=LET(r, A1:A, FILTER(r, MAP(r, LAMBDA(y, IF(y="", FALSE, NOT(COUNTIF(MAP(B1:B, LAMBDA(x, IF(x="", FALSE, EXACT(x, y)))), TRUE)))))))`

|   | A | B | C | D | E | F | G | H |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| 1 | Cat  | Cat    | | TRUE | 2 | 2 | Cheese | Milk |
| 2 | Dog  | Cheese | | FALSE | | 0 | milk | cat |
| 3 | Cat  | Dog    | | TRUE | | 1 |
| 4 | Milk | milk   | | FALSE | | 0 |
| 5 | cat  |        | | FALSE |
| 6 |      | | | FALSE |

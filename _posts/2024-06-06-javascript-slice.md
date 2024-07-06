---
title: JavaScript 数组方法 slice 的用法
date: 2024-06-06 21:16:00 -0400
categories: [笔记, JavaScript]
tags: [javascript, slice]
description: 本文记录 Array.prototype.slice 的基本用法
---

[`Array`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) 实例的 `slice()` 方法返回原数组中一部分的 [浅拷贝](https://developer.mozilla.org/en-US/docs/Glossary/Shallow_copy) 到一个新的数组，范围从 `start` 到 `end`（不包括 `end` ）。其中 `start` 和 `end` 是原数组的索引。此过程中原数组不会被修改。

## 举例

```js
const animals = ['ant', 'bison', 'camel', 'duck', 'elephant'];

console.log(animals.slice(2));
// Expected output: Array ["camel", "duck", "elephant"]

console.log(animals.slice(2, 4));
// Expected output: Array ["camel", "duck"]

console.log(animals.slice(1, 5));
// Expected output: Array ["bison", "camel", "duck", "elephant"]

console.log(animals.slice(-2));
// Expected output: Array ["duck", "elephant"]

console.log(animals.slice(2, -1));
// Expected output: Array ["camel", "duck"]

console.log(animals.slice());
// Expected output: Array ["ant", "bison", "camel", "duck", "elephant"]
```

## 语法

```js
slice()
slice(start)
slice(start, end)
```
{: .nolineno}

## 参数

### start

可选，从 0 开始的索引。

- 支持负索引：如果 `-array.length <= start < 0`，那么 `start` 就取 `start + array.length` 。（见举例中的第 4 例）
- 如果 `start < -array.length` 或者 `start` 没有指定，就取 `0`。（见举例中的最后 1 例）
- 如果 `start >= array.length`，则什么也不会截取到。

### end

可选，从 0 开始的索引。

- 支持负索引：如果 `-array.length <= end < 0`，那么 `end` 就取 `end + array.length` 。（见举例中的第 5 例）
- 如果 `end < -array.length`，就取 `0`。
- 如果 `end >= array.length` 或者 `end` 没有指定，就取 `array.length`，这样会取到剩下的所有元素。（见举例中的第 1 和第 3 例）
- 如果 `end` 指定的位置在 `start` 之前或相同，则什么也不会截取到。

## 返回值

包含截取元素的新数组。

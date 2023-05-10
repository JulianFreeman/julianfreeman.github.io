---
layout: post
title: Google Sheet 公式笔记 - 检索2
date: 2023-05-10 07:52:00 -0400
tags: [google sheet, formula]
description: 本文介绍使用 CHOOSEROWS 和 MATCH 检索 # Add post description (optional)
img: google-sheet.jpg  # Add image post (optional)
---
在 [前文][search1] 中我们介绍了使用 `MAP` 加 `FILTER` 代替 `VLOOKUP`，但是在那种情景中，实际上每次进行过滤的时候，只有一行会被过滤出来，但是理论上 `FILTER` 应该会检查所有要被过滤的内容的，有可能要比 `MATCH` 或 `VLOOKUP` 这种只输出匹配的第一个值的函数要慢一点。所以本文记录一下使用 `MATCH` 的替代方法。

# MATCH

`MATCH` 函数接受三个参数，第一个参数是要匹配的值，第二个参数是要匹配的范围，这个范围 **必须是一维的**，也就是说只能是单行或者单列，否则会报错。第三个参数通常为 0（表示范围未排序）。

`MATCH` 返回的是第一个匹配的值在范围内的 **索引**。

一般来说 `MATCH` 与 `INDEX` 配合使用多一些，但是 `INDEX` 只能返回单值，所以这里我们选择 `CHOOSEROWS` 与 `MATCH` 配合使用。

# 一个错误

最开始的思路是使用 `MAP` 加 `MATCH` 返回一个要检索的索引数组，然后直接传递给 `CHOOSEROWS` 来重组每个索引对应的行。但是实际情况下，有时候要检索的值的个数可能并不固定，这样有时候 `MATCH` 就会尝试检索一个空值，得到的结果就是找不到该值，然后返回一个错误，这个错误会与其他能找到的索引一起组成数组传递给 `CHOOSEROWS`，但是后者不能处理错误，所以直接整个报错。

# 修正

对于此错误，暂时也没什么好的处理办法，只能放弃传递索引数组给 `CHOOSEROWS` 的方法，转换思路。先用 `CHOOSEROWS` 和 `MATCH` 选出一行，再使用 `MAP` 对每一个要检索的值都进行该操作。如果遇到空值，直接排除掉。

如下样表，

|   | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q | R | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| 1 | **ID** | **Name** | **Age** | **Sex** |
| 2 | 1  | Tom   | 12  | M   |
| 3 | 2  | John  | 43  | M   |
| 4 | 2  | Alice | 45  | F   |
| 5 | 3  | Dom   | 23  | M   |
| 6 | 4  | Tina  | 34  | F   |

在 `G2` 中输入公式

`=MAP(F2:F6, LAMBDA(x, IF(x="", "", CHOOSEROWS({B2:B6,D2:D6}, MATCH(x, A2:A6, 0)))))`，

|   | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q | R | S | T |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| 1 | **ID** | **Name** | **Age** | **Sex** | | **Range** | **Name** | **Sex** |
| 2 | 1  | Tom   | 12  | M   |  | 1 | Tom  | M |
| 3 | 2  | John  | 43  | M   |  | 2 | John | M |
| 4 | 2  | Alice | 45  | F   |  | 4 | Tina | F |
| 5 | 3  | Dom   | 23  | m   |  |   |      |   |
| 6 | 4  | Tina  | 34  | F   |  |   |      |

可以看到，`MAP` 的范围是 `F2:F6`，虽然 `F5` 和 `F6` 是空值，但也不影响检索。

该方式相比于使用 `VLOOKUP` 同样拥有不限制检索列的位置，以及自动矫正范围列的移动的特性。只不过相比于使用 `FILTER` 的性能差异，没有去实际测试。

---

封面图：Photo by [Rubaitul Azad][cover-img-author] on [Unsplash][cover-img-src]

[search1]: {{site.url}}/gsf2-map-filter/

[cover-img-author]: https://unsplash.com/ja/@rubaitulazad?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
[cover-img-src]: https://unsplash.com/photos/TisvwNLLWA4?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText


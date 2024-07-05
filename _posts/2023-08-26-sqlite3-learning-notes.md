---
title: SQLite3 学习笔记
date: 2023-08-26 17:10:00 -0400
categories: [学习笔记, SQL]
tags: [programming, sqlite3]
description: 本文记录一些有关 SQLite3 的内容
---

## SQLite3 CLI

### 链接 SQLite 数据库

在终端中直接运行 `> sqlite3` 进入命令行模式。

此时 SQLite 会话会在内存中创建一个临时数据库，所有对此数据库的修改都会在会话关闭时丢失。

此时要打开一个数据库文件，可以使用 `.open FILENAME` 命令。

或者说也可以在开启会话之时就指定数据库 `> sqlite3 FILENAME` 。

如果开启会话时指定的数据库文件不存在，程序会自动创建一个。

### 显示所有可用命令

`.help` 命令可以提供帮助信息。

`.help COMMAND` 可以显示特定命令的帮助。比如 `.help mode` 。

### 显示当前链接中的所有数据库

使用 `.databases` 命令，至少会显示一个数据库 `main` 。

如果要添加额外的数据库到当前链接，使用

```sql
attach database "FILENAME" as ALIAS;
```

再次使用 `.databases` 就会显示添加的数据库了。

### 退出

`.exit` 退出 sqlite3 程序。

### 显示数据库中的数据表

`.tables` 命令。

可以按照 `LIKE` 的方式匹配某些数据表，比如 `.tables '%es'` 。

> 这里 `.tables` 和 `.table` 貌似没什么区别。

### 显示一个数据表的结构

`.schema TABLE` 命令。

如果不指定数据表名称，则显示所有数据表的结构。

### 显示索引

`.indexes` 命令，显示所有索引。

`.indexes TABLE` 显示指定数据表的索引。

### 保存查询结果到文件

`.output FILENAME` 命令。

一旦运行这个命令之后，之后所有的查询结果都会保存到这个文件。

如果只想保存下一次的查询结果，使用 `.once FILENAME` 命令。

如果从某一刻开始不想把结果保存到文件了，运行一下 `.output` 命令就可以了。

### 从文件中执行 SQL 语句

`.read FILENAME` 命令。

## SELECT

`SELECT` 的全部格式如下：

```sql
SELECT DISTINCT
    column1,
    column2 AS col2,
    column3 col3,
    ...
FROM
    table1
    INNER JOIN table2 ON expr1
    LEFT JOIN table3 ON expr2
    ...
WHERE
    search_condition
GROUP BY
    column1,
    column2,
    ...
HAVING filter_condition
ORDER BY
    column1 ASC NULLS FIRST,
    column2 DESC NULLS LAST,
    ...
LIMIT row_count OFFSET offset_num
;
```

### DISTINCT

去除返回结果中的重复行，必须紧跟 `SELECT` 之后。

如果返回结果有多列，则两行中必须每一列的值都对应相同才被认定为重复行。

多行 `NULL` 值被认为是重复的。

### JOIN

多表链接也是个大话题，单独一章：[JOIN](#join-1)

### WHERE

`WHERE` 太大了，值得单独一章：[WHERE](#where-1)

### GROUP BY

一般 `GROUP BY` 都是跟聚合函数 `sum` 、 `count` 、 `max` 等等之类的一起使用，如果不使用聚合函数的话，只会显示重复行的第一行的列数据。

多列打组的话，比如 A 列有 1 ， B 列有 2 ，那如果以 A 和 B 打组，那实际就是以 (1, 2) 为组整理，并非是先以 A 打组再以 B 打组。

### ORDER BY

#### 多列排序

按照多列排序的意思是，先按照第一列排序，比如 `1, 2, 2, 3, 3, 3, 4, 5, 5, 6` 这一列是排完序的，但是有些行在这一列的值是相同的，比如 `2, 3, 5` ，对于这些行，它们的顺序会按照第二列排序。如果有些行的第一列值和第二列值都相同，那如果有第三列排序，它们会再次被第三列排序。以此类推。

#### 使用列序号

此处的 `column1` 和 `column2` 除了可以是列的标签外，也可以是列的序号。比如

```sql
SELECT name, milliseconds, albumid FROM tracks ORDER BY 3, 2 DESC;
```

会先按照 `albumid` 升序排列，然后按照 `milliseconds` 降序排列。

列序号从 1 开始。

#### `NULL` 值

在排序当中，有些值可能是 `NULL` ， SQLite 会认为 `NULL` 值比所有其他值都小，也就是说，如果按照升序排列， `NULL` 值会出现在最顶端。

在 SQLite 3.30.0 之后新增加了 `NULLS FIRST` 和 `NULLS LAST` 的选项，可以显式指定 `NULL` 值是出现在开头还是结尾。

### LIMIT

对于偏移值，除了使用 `LIMIT row_count OFFSET offset_num` 的写法外，还可以使用 `LIMIT offset_num, row_count` 的写法，也就是说以下两句等同。

```sql
SELECT trackid, name FROM tracks LIMIT 10, 20;
SELECT trackid, name FROM tracks LIMIT 20 OFFSET 10;
```

## WHERE

`WHERE` 可以在 `SELECT` 、 `UPDATE` 和 `DELETE` 中使用。

#### 比较运算符

| 运算符 | 含义 |
| --- | --- |
| =  | 等于 |
| <> 或者 !=  | 不等于 |
| <  | 小于 |
| >  | 大于 |
| <=  | 小于等于 |
| >=  | 大于等于 |

在比较时，尽可能统一比较双方的类型，如果类型不统一， SQLite 会进行隐式转换。

#### 逻辑运算符

| 运算符 | 含义 |
| --- | --- |
| ALL  | 如果所有表达式都为 1 就返回 1 |
| AND  | 如果两个表达式都为 1 就返回 1 否则返回 0 |
| ANY  | 如果任意一个表达式为 1 就返回 1 |
| BETWEEN  | 如果值在范围内就返回 1 |
| EXISTS  | 如果子语句的返回结果至少存在一行就返回 1 |
| IN  | 如果值在列表内就返回 1 |
| LIKE 或者 GLOB  | 如果值符合指定的模式就返回 1 |
| NOT  | 反转其他运算符的结果，比如 NOT EXISTS ， NOT IN ， NOT BETWEEN 等 |
| OR  | 如果两个表达式任意一个为 1 就返回 1 |

因为 SQLite 并没有布尔类型，所以 1 表示真， 0 表示假。

### BETWEEN

`BETWEEN` 的范围包含边界。如果想要不包含边界的范围，只能手动用大于号和小于号。

### IN

`IN` 的语法为 `expression [NOT] IN (value_list|subquery);` 。

也就是说，括号里可以是一串值的列表，也可以是一个查询子句的返回结果，这个结果只能是**单列**。

```sql
SELECT trackid, name, albumid 
FROM tracks 
WHERE albumid IN (SELECT albumid FROM albums WHERE artistid=12);
```

### LIKE

`LIKE` 的语法为 `column_1 LIKE pattern;` 。

支持两个通配符，其中 `%` 匹配 0 个或多个字符， `_` 匹配 1 个字符。

在 ASCII 范围内的字符， `LIKE` 不区分大小写。要想区分，可以使用

```sql
PRAGMA case_sensitive_like = true;
```

另外 `pattern` 需要用引号引起来。

但如果想匹配的字符串中有 `%` 或者 `_` 怎么办？

使用 `column_1 LIKE pattern ESCAPE expression;` ，其中 `expression` 的计算结果是单个字符， SQLite 会把紧跟在这个字符后面的 `%` 或者 `_` 当成纯字符而不是通配符。

比如 `c LIKE '%10\%%' ESCAPE '\';` 可以匹配包含 `10%` 的字符串。

### GLOB

`GLOB` 也用于匹配字符，不过跟 `LIKE` 不同的是， `GLOB` 区分大小写，而且不支持逃逸字符。

`GLOB` 使用 `*` 匹配 0 个或多个字符，使用 `?` 匹配 1 个字符。

还使用 `[]` 匹配括号内的任意 1 个字符，或者形如 `[a-z]` 可以指定匹配的范围， `[^0-9]` 可以反转匹配的范围。

### IS NULL

因为 `NULL` 甚至不等于它自己，所以形如 `WHERE expr = NULL` 的条件永远返回假。那这时候要判断一个值是不是空，就用 `WHERE expr IS NULL` 或者 `WHERE expr IS NOT NULL` 。

## JOIN

假设有几个表：

表 A ：

| m | f |
| --- | --- |
| a1 | 1 |
| a2 | 2 |
| a3 | 3 |

表 B ：

| n | f |
| --- | --- |
| b1 | 1 |
| b2 | 3 |
| b3 | 5 |

表 C ：

| c | n |
| --- | --- |
| 1 | b2 |
| 2 | b3 |
| 3 | b4 |

### 内联（ INNER JOIN ）

多表联结其实干了这么一件事，先以内联为例，以下语句干了啥呢？

```sql
SELECT 
    m,
    A.f AS Af,
    B.f AS Bf,
    n
FROM
    A
    INNER JOIN B ON A.f=B.f;
```

输出结果

| m | Af | Bf | n |
| --- | --- | --- | --- |
| a1 | 1 | 1 | b1 |
| a3 | 3 | 3 | b2 |

首先，程序会拿 A 表中的**每一行**跟 B 表中的**每一行**进行比较，比较的条件就是 `A.f=B.f` 是否成立，符合条件的行会被联结成联表的新行。

比如，程序先拿 A 表的第一行，

| m | f |
| --- | --- |
| a1 | 1 |

| n | f |
| --- | --- |
| b1 | 1 |
| b2 | 3 |
| b3 | 5 |

跟 B 表的第一行对上了，所以 A 表的第一行跟 B 表的第一行联结成为联表的第一行。

然后拿 A 表的第二行，

| m | f |
| --- | --- |
| a2 | 2 |

| n | f |
| --- | --- |
| b1 | 1 |
| b2 | 3 |
| b3 | 5 |

一行也没对上，所以第二行略过。

然后第三行，

| m | f |
| --- | --- |
| a3 | 3 |

| n | f |
| --- | --- |
| b1 | 1 |
| b2 | 3 |
| b3 | 5 |

跟 B 表的第二行对上了，所以 A 表的第三行和 B 表的第二行联结成为联表的第二行。

于是我们可以认为这个联表长这个样子：

| A.m | A.f | B.n | B.f |
| --- | --- | --- | --- |
| a1 | 1 | b1 | 1 |
| a3 | 3 | b2 | 3 |

在对这个联表 `SELECT` 的时候，如果列标签没有冲突，是唯一的，则表名是可选的，比如 `SELECT m` 等同于 `SELECT A.m` ，但是如果有冲突，比如 `A.f` 和 `B.f` ，去掉表名就区分不出是哪个表的列了，就必须显式指定表名。

### 联结条件

也许你会觉得反正 `A.f` 和 `B.f` 是一样的，留一个不就行了吗？其实不然，它们在这里一样只是因为我们的联结条件是 `A.f=B.f` ，所以它们当然一样。但是如果是其它条件呢？比如：

```sql
SELECT 
    A.f AS Af,
    B.f AS Bf,
    m,
    n
FROM
    A
    INNER JOIN B ON A.f+B.f=4;
```

输出结果：

| Af | Bf | m | n |
| --- | --- | --- | --- |
| 1 | 3 | a1 | b2 |
| 3 | 1 | a3 | b1 |

这不就不一样了。但其实流程跟上边还是一样的，只是判断联结与否的条件变了，但依然，程序是拿 A 表的**每一行**与 B 表的**每一行**判断联结条件是否成立，成立的行都会联结起来成为联表的新行。如果 A 表的某一行跟 B 表的多行都符合联结条件，那这多行都会成为联表的新行， A 表的这一行就简单的重复多次。

### 三表内联

```sql
SELECT 
    A.f Af,
    A.m,
    B.f Bf,
    B.n Bn,
    C.c,
    C.n Cn
FROM 
    A 
    INNER JOIN B ON A.f = B.f
    INNER JOIN C ON C.n = B.n;
```

其实一个道理， A 和 B 内联的新表再跟 C 表内联，就出结果了。

| Af | m | Bf | Bn | c | Cn |
| --- | --- | --- | --- | --- | --- |
| 3 | a3 | 3 | b2 | 1 | b2 |

### 左联（ LEFT JOIN ）

理解了内联，其实左联就很简单了。

```sql
SELECT
    A.f Af,
    m,
    B.f Bf,
    n
FROM
    A
    LEFT JOIN B ON A.f=B.f;
```

输出结果：

| Af | m | Bf | n |
| --- | --- | --- | --- |
| 1 | a1 | 1 | b1 |
| 2 | a2 |  |  |
| 3 | a3 | 3 | b2 |

左联的意思就是把 A 表的行全部输出，然后按照联结条件填充 B 表的列值，不符合联结条件的就留空。

```sql
SELECT
    A.f Af,
    A.m,
    B.f Bf,
    B.n Bn,
    C.c,
    C.n Cn
FROM
    A
    LEFT JOIN B ON A.f=B.f
    LEFT JOIN C ON B.n=C.n;
```

输出结果：

| Af | m | Bf | Bn | c | Cn |
| --- | --- | --- | --- | --- | --- |
| 1 | a1 | 1 | b1 |  |  |
| 2 | a2 |  |  |  |  |
| 3 | a3 | 3 | b2 | 1 | b2 |

如果 A 表的一行在 B 表中有多行符合联结条件，那依然这多行都可以成为联表的新行， A 表的这行重复多次。

### `USING`

因为类似于 `A.f=B.f` 的用法太常见了，所以有一个简单的语法：

```sql
SELECT m, n FROM A INNER JOIN B USING(f);
SELECT m, n FROM A INNER JOIN B ON A.f=B.f;
```

以上两句等同。

### 交叉联结（ CROSS JOIN ）

交叉联结没有 `ON` 或 `USING` 选项，以下写法等同，都是交叉联结：

```sql
SELECT * FROM A JOIN B;
SELECT * FROM A INNER JOIN B;
SELECT * FROM A CROSS JOIN B;
SELECT * FROM A, B;
```

交叉联结表示 A 表的每一行都与 B 表的每一行构成联表的新行。这样也就是说，如果 A 表有 m 行， B 表有 n 行，那联表就有 m*n 行。

可能乍一看这种联表没啥用，但是也许在某些情景下还是有点方便的，比如要查某些人员在某些日期内每一天的出勤情况，那可以用这个：

```sql
SELECT
    name,
    date
FROM
    members
    CROSS JOIN dates;
```

### 其他联结

SQLite 不支持右联（RIGHT JOIN ）或者全联（FULL OUTER JOIN ），但是右联其实就是把左联的两个表位置换一下，全联就是左联加右联，可以使用 `UNION ALL` 实现。

SQLite 还可以实现自联（ SELF JOIN ），具体看[链接](https://www.sqlitetutorial.net/sqlite-self-join/)。

## SET

### UNION

`UNION` 的语法是这样的：

```sql
query1
UNION [ALL]
query2
UNION [ALL]
query3
...;
```

`UNION` 用来将多次查询的结果合并在一起。注意， `JOIN` 联结的是不同表的列，而 `UNION` 合并的是不同表的行。

所以 `UNION` 有以下规则：

1. 所有查询结果的列数必须相同。
2. 所有查询结果中对应列的数据类型必须一致。
3. 查询结果的列标签以第一条查询语句的结果为准。
4. `GROUP BY` 会应用在每一条子语句中。
5. `ORDER BY` 会应用在合并后的结果中。

`UNION` 和 `UNION ALL` 的区别是，前者会剔除重复行，后者不会，所以后者要快。

#### 全联

前面说过，可以用左联和合并实现全联，其实就是把左联和右联的结果合并在一起。

两个表，表 dogs ：

| type | color |
| --- | --- |
| Hunting | Black |
| Guard | Brown |

表 cats ：

| type | color |
| --- | --- |
| Indoor | White |
| Outdoor | Black |

```sql
SELECT d.type Dtype, d.color Dcolor, c.type Ctype, c.color Ccolor
FROM dogs d
LEFT JOIN cats c USING(color)
UNION ALL
SELECT d.type Dtype, d.color Dcolor, c.type Ctype, c.color Ccolor
FROM cats c
LEFT JOIN dogs d USING(color);
```

输出结果

| Dtype | Dcolor | Ctype | Ccolor |
| --- | --- | --- | --- |
| Hunting | Black | Outdoor | Black |
| Guard | Brown |  |  |
|  |  | Indoor | White |
| Hunting | Black | Outdoor | Black |

可以看到有重复行（第一行和第四行），只要使用 `UNION` 合并（而非 `UNION ALL` ）就不会有重复行了。

### EXCEPT

`EXCEPT` 的语法为

```sql
query1
EXCEPT
query2;
```

规则如下：

1. 两次查询结果的列数相同。
2. 对应列的值和类型可比较。

`EXCEPT` 会从 query1 中剔除掉 query2 中的结果。

### INTERSECT

`INTERSECT` 语法为

```sql
query1
INTERSECT
query2;
```

规则如下：

1. 两次查询的列数和顺序必须一致。
2. 数据类型必须可比较。

`INTERSECT` 会返回两次查询中共有的结果。

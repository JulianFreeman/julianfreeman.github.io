---
title: MySQL 学习笔记
date: 2023-12-14 19:46:00 -0500
categories: [学习笔记, SQL]
tags: [mysql]
description: 本文记录一些有关 SQLite3 的内容
media_subpath: /assets/img/mysql-notes/
---

## 开始之前

### 安装 MySQL Server

#### Windows & MacOS

网址：https://dev.mysql.com/downloads/mysql/

Windows 安装后需要把 `C:\Program Files\MySQL\MySQL Server 8.2\bin`{: .filepath} 添加到环境变量。

MacOS 安装后需要在 `~/.zshrc`{: .filepath} 文件中添加 `export PATH=${PATH}:/usr/local/mysql/bin`。

#### Ubuntu

```sh
apt install mysql-server
mysql_secure_installation
```
{: .nolineno}

参考：[How To Install MySQL on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04)

### 安装数据库管理工具

推荐 [DBeaver Community](https://dbeaver.io/download/)，三个平台都支持。

第一次连接 MySQL 服务器时可能需要下载该软件的 MySQL 驱动，下载完后重启软件，再尝试连接。

### 样例数据库

[MySQL Sample Database](https://www.mysqltutorial.org/getting-started-with-mysql/mysql-sample-database/)，内附来源。

### 加载样例数据库

在终端使用 `mysql -u root -p` 登录数据库后，使用 `source` 加数据库文件路径加载数据库。

```console
source /path/to/mysqlsampledatabase.sql
```

## 获取数据

### `SELECT`

想到一种可以理解 `SELECT` 的方法：`SELECT` 就跟 `print` 差不多。

比如 `SELECT 1 + 1;` 会输出

```console
1 + 1|
-----+
    2|
```

MySQL 也支持很多处理函数，比如返回当前时间的函数 `NOW()`，可以使用 `SELECT now(); `

```console
now()              |
-------------------+
2023-12-14 09:11:20|
```

还有字符串连接函数 `CONCAT()`，比如 `SELECT concat('John', ' ', 'Doe');`

```console
concat('John', ' ', 'Doe')|
--------------------------+
John Doe                  |
```

> MySQL 中字符串用 **单引号**。
{: .prompt-tip}

#### `AS`

输出的表头默认就是命令本身，不太好看，我们可以用 `AS` 重命名表头，比如 `SELECT concat('John', ' ', 'Doe') AS Name;`

```console
Name    |
--------+
John Doe|
```

如果表头有空格，需要用引号括起来：`SELECT concat('John', ' ', 'Doe') AS 'Full Name';`

```console
Full Name|
---------+
John Doe |
```

#### `FROM`

`FROM` 会在 `SELECT` 之前先执行，`FROM table SELECT col1, col2` 更好理解（当然不合语法），可以想象成“从数据表的所有列中选择某几列”。

![MySQL-SELECT-FROM](MySQL-SELECT-FROM.svg)

### `ORDER BY`

`ORDER BY` 会在 `SELECT` 之后执行。

![MySQL-ORDER-BY](MySQL-ORDER-BY.svg)

但是用来排序的列不一定非得是选择的这几列

```sql
SELECT 
    contactLastName, 
    contactFirstName 
FROM 
    customers 
ORDER BY 
    postalCode;
```

```console
contactLastName|contactFirstName|
---------------+----------------+
MacKinlay      |Wales           |
Gao            |Mike            |
Snowden        |Tony            |
Ricotti        |Franco          |
...
```

排序时也可以用别名指定列

```sql
SELECT 
    orderNumber, 
    orderLineNumber, 
    quantityOrdered * priceEach AS subtotal 
FROM 
    orderdetails 
ORDER BY 
    subtotal DESC;
```

```console
orderNumber|orderLineNumber|subtotal|
-----------+---------------+--------+
      10403|              9|11503.14|
      10405|              5|11170.52|
      10407|              2|10723.60|
      10404|              3|10460.16|
...
```

因为 `ORDER BY` 是在 `SELECT` 之后执行的，所以它知道 `SELECT` 里定义了哪些别名。

#### 使用自定义列表排序

`field()` 函数可以返回第一个参数在后面几个参数中的位置，比如 `SELECT field('A', 'A', 'B', 'C');` 会输出

```console
field('A', 'A', 'B', 'C')|
-------------------------+
                        1|
```

如果第一个参数是 `'B'`，结果就是 `2`，如果找不到参数，会返回 `0`。

下面观察如下语句

```sql
SELECT 
    orderNumber, 
    status 
FROM 
    orders 
ORDER BY 
    field(status, 
        'In Process', 
        'On Hold', 
        'Cancelled', 
        'Resolved', 
        'Disputed', 
        'Shipped');
```

对于每一行的 `status`，`field()` 函数都会将其匹配到一个数字，然后根据这些数字排序。

```console
orderNumber|status    |
-----------+----------+
      10420|In Process|
      10421|In Process|
      10422|In Process|
      10423|In Process|
      10424|In Process|
      10425|In Process|
      10334|On Hold   |
      10401|On Hold   |
...
```

> 暂时就先理解为就这么个语法吧。

#### `NULL` 值排序

`NULL` 值依然被认为是最小。

### `WHERE`

`WHERE` 会在 `FROM` 之后、`SELECT` 之前执行。也就是说，语句先过滤出符合条件的行，然后再选择指定的列。

![MySQL-WHERE](MySQL-WHERE.svg)

```sql
SELECT 
    lastName, 
    firstName, 
    jobTitle 
FROM 
    employees 
WHERE 
    jobTitle = 'Sales Rep';
```

```console
lastName |firstName|jobTitle |
---------+---------+---------+
Jennings |Leslie   |Sales Rep|
Thompson |Leslie   |Sales Rep|
Firrelli |Julie    |Sales Rep|
Patterson|Steve    |Sales Rep|
...
```

> 所以在 SQL 语句中相等判断用单等号就行。
{: .prompt-tip}

`WHERE` 的条件中可以用 `AND`、`OR`、`NOT` 做多重判断。

> 更多内容见 [SQLite3 学习笔记]({{site.url}}/posts/sqlite3-learning-notes#where-1) 。
{: .prompt-info}

在 MySQL 中也没有布尔类型，所以一个布尔表达式，明面上会说解析为三类值：`TRUE`、`FALSE`、`UNKNOWN`，但实际上是 `1`、`0`、`NULL`。

### `DISTINCT`

`DISTINCT` 的执行位置如下

![MySQL-DISTINCT](MySQL-DISTINCT.svg)

`NULL` 值也会被认为是相同的，因此如果输出列有多个 `NULL` 值，则只会输出一个。

### `AND`

前面说到布尔表达式的返回值除了真假之外，还有个空值，这个是这么回事：

对于 `AND` 操作符，

- 如果两边都不是 `0` 且不是 `NULL`，则返回 `1`
- 如果有一边是 `0`，则返回 `0`
- 否则，返回 `NULL`

看示例吧

```sql
SELECT 
    1 AND 1, 
    1 AND 0, 
    0 AND 1, 
    0 AND 0, 
    0 AND NULL;
```

```console
1 AND 1|1 AND 0|0 AND 1|0 AND 0|0 AND NULL|
-------+-------+-------+-------+----------+
      1|      0|      0|      0|         0|
```

只有下面的情况会返回 `NULL`

```sql
SELECT 
    1 AND NULL, 
    NULL AND NULL;
```

```console
1 AND NULL|NULL AND NULL|
----------+-------------+
          |             |
```

下面是真值表

| | TRUE | FALSE | NULL |
|--|--|--|--|
| **TRUE** | TRUE | FALSE | NULL |
| **FALSE** | FALSE | FALSE | FALSE |
| **NULL** | NULL | FALSE | NULL |


### `OR`

直接上真值表

|  | TRUE | FALSE | NULL |
|--|--|--|--|
| **TRUE** | TRUE | TRUE | TRUE |
| **FALSE** | TRUE | FALSE | NULL |
| **NULL** | TRUE | NULL | NULL |

其实对于常规的真假判断，都是一样的，无非是多了一个空值的情况，所以只需要记住有 `NULL` 掺和的情况是什么样子就行了

```sql
SELECT 
    1 OR NULL, 
    0 OR NULL, 
    NULL OR NULL;
```

```console
1 OR NULL|0 OR NULL|NULL OR NULL|
---------+---------+------------+
        1|         |            |
```

`AND` 的优先级比 `OR` 高。这一点可以通过加括号改变。

### `IN`

`value IN (value1, value2, ...)` 等同于 `value = value1 OR value = value2 OR ...`

一般来说会返回 `1` 或者 `0`，但是在下面的情况中会返回 `NULL`

- `value` 是 `NULL`
- `value` 匹配不到列表中的任何值，且列表中有 `NULL` 值

比如 `SELECT NULL IN (1, 2, 3);`

```console
NULL IN (1, 2, 3)|
-----------------+
                 |
```

比如 `SELECT 0 IN (1, 2, 3, NULL);`

```console
0 IN (1, 2, 3, NULL)|
--------------------+
                    |
```

注意，`SELECT NULL IN (1, 2, 3, NULL);` 也是 `NULL`

```console
NULL IN (1, 2, 3, NULL)|
-----------------------+
                       |
```

因为 `NULL` 也不等于它自己。

### `NOT IN`

正常情况就不说了，就记录一下涉及到 `NULL` 的吧

```sql
SELECT 
    1 NOT IN (1, 2, NULL), 
    0 NOT IN (1, 2, NULL), 
    NULL NOT IN (1, 2), 
    NULL NOT IN (1, 2, NULL) ;
```

```console
1 NOT IN (1, 2, NULL)|0 NOT IN (1, 2, NULL)|NULL NOT IN (1, 2)|NULL NOT IN (1, 2, NULL)|
---------------------+---------------------+------------------+------------------------+
                    0|                     |                  |                        |
```

这个结果，简单来看呢，其实就是说，`NOT IN` 就是先把 `IN` 的结果算出，然后取反，如果说 `IN` 的结果是 `NULL`，取反还是 `NULL`。

### `BETWEEN`

`value BETWEEN low AND high` 等同于 `value >= low AND value <= high`

如果 `value`、`low`、`high` 的任意一方是 `NULL`，返回值就是 `NULL`。

其取反

`value NOT BETWEEN low AND high` 等同于 `value < low OR value > high`

#### 日期比较

MySQL 中有日期类型，因此要比较日期时，需要显式地将值转换为日期

```sql
SELECT 
    orderNumber, 
    requiredDate, 
    status 
FROM 
    orders 
WHERE 
    requiredDate BETWEEN 
        cast('2003-01-01' AS date) AND 
        cast('2003-01-19' AS date);
```

```console
orderNumber|requiredDate|status |
-----------+------------+-------+
      10100|  2003-01-13|Shipped|
      10101|  2003-01-18|Shipped|
      10102|  2003-01-18|Shipped|
```

### `LIKE`

`%` 匹配 0 或多个字符，`_` 匹配一个字符。**不区分大小写**，默认的转义字符是 `\`

```sql
SELECT 
    productCode, 
    productName 
FROM 
    products 
WHERE 
    productCode LIKE '%\_20%';
```

```console
productCode|productName                              |
-----------+-----------------------------------------+
S10_2016   |1996 Moto Guzzi 1100i                    |
S24_2000   |1960 BSA Gold Star DBD34                 |
S24_2011   |18th century schooner                    |
S24_2022   |1938 Cadillac V-16 Presidential Limousine|
S700_2047  |HMS Bounty                               |
```

也可以手动指定转义字符

```sql
SELECT 
    productCode, 
    productName 
FROM 
    products 
WHERE 
    productCode LIKE '%$_20%' ESCAPE '$';
```

结果一样。

### `LIMIT`

语法 `LIMIT [offset,] row_count`，可以理解为，先跳过 `offset` 行，再限定输出 `row_count` 行。

所以，`LIMIT 0, row_count` 和 `LIMIT row_count` 是一样的。

一般 `LIMIT` 都是跟 `ORDER BY` 一起用的。

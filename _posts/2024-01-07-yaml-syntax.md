---
title: YAML 语法笔记
date: 2024-01-07 22:34:00 -0500
categories: [学习笔记, YAML]
tags: [yaml]
description: 本文记录一些基础的 YAML 语法
---

YAML 中的列表元素用 `- ` 声明，比如下面是一个列表

```yaml
- Mark McGwire
- Sammy Sosa
- Ken Griffey
```

YAML 中的键值对用 `: ` 分割，注释用 `#` 标记，比如下面是带注释的几个键值对

```yaml
hr:  65    # Home runs
avg: 0.278 # Batting average
rbi: 147   # Runs Batted In
```

下面是两个键值对，每个值都是一个列表，YAML 像 Python 一样使用缩进

```yaml
american:
    - Boston Red Sox
    - Detroit Tigers
    - New York Yankees
national:
    - New York Mets
    - Chicago Cubs
    - Atlanta Braves
```

下面是一个列表，每个元素都是一组键值对

```yaml
-
    name: Mark McGwire
    hr:   65
    avg:  0.278
-
    name: Sammy Sosa
    hr:   63
    avg:  0.288
```

如果不用缩进，也可以用类似 Python 的列表和字典语法。

下面是列表的列表

```yaml
- [name        , hr, avg  ]
- [Mark McGwire, 65, 0.278]
- [Sammy Sosa  , 63, 0.288]
```

下面是字典的字典

```yaml
Mark McGwire: {hr: 65, avg: 0.278}
Sammy Sosa: {
    hr: 63,
    avg: 0.288,
}
```

YAML 中一个文档以 `---` 行开始，以 `...` 行结束。同一文件中可以有多个文档。

> 其他的用到了再去看看 https://spacelift.io/blog/yaml 吧。
{: .prompt-info}

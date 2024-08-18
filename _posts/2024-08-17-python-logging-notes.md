---
title: Python logging 模块学习笔记
date: 2024-08-17 23:33:00 -0400
categories: [学习笔记, Python]
tags: [logging]
description: 本文根据 ChatGPT 的答案整理
---

## 介绍

`logging` 库是 Python 标准库中的一个模块，用于记录应用程序运行时的日志信息。与简单的 `print()` 输出相比，`logging` 提供了更强大的功能，包括日志的不同级别、日志格式、输出到不同目标（如文件、控制台等），以及灵活的日志配置。

## 基本概念和常见用法

### 日志级别

`logging` 模块定义了几个日志级别，每个级别代表不同的严重性。常用的日志级别如下：

- `DEBUG`: 最低级别，用于输出调试信息，通常只在开发或调试阶段使用。
- `INFO`: 一般信息，表示程序运行的正常信息，通常用来记录程序的正常操作。
- `WARNING`: 警告信息，表示程序中出现了某些不太正常的情况，但程序仍然可以继续运行。
- `ERROR`: 错误信息，表示程序中出现了错误，这些错误通常会影响程序的运行。
- `CRITICAL`: 最严重的级别，表示出现了严重的错误，程序可能无法继续运行。

### 基本配置 basicConfig()

`basicConfig()` 是配置日志记录的最简单方法。它允许你设置日志级别、日志格式、输出目标等。

```python
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

logging.debug("这是调试信息")   # 不会显示，因为日志级别设置为 INFO
logging.info("这是普通信息")    # 会显示
logging.warning("这是警告信息")  # 会显示
logging.error("这是错误信息")    # 会显示
logging.critical("这是严重错误信息") # 会显示
```

### Logger, Handler, Formatter

`logging` 模块采用模块化的设计，包含三个核心组件：

1. `Logger`: 负责生成日志消息的入口。可以通过 `logging.getLogger(name)` 获取。
2. `Handler`: 决定日志消息输出到哪里。常见的 `Handler` 包括 `StreamHandler`（输出到控制台）和 `FileHandler`（输出到文件）。
3. `Formatter`: 决定日志消息的格式，通常包括时间戳、日志级别、消息内容等。

示例见 [配置日志记录到多个目标](#配置日志记录到多个目标)

### 捕获异常并记录日志

`logging` 模块可以很好地与异常处理结合使用，你可以在捕获异常时记录错误信息。

```python
import logging

logging.basicConfig(level=logging.ERROR, format='%(asctime)s - %(levelname)s - %(message)s')

try:
    1 / 0
except ZeroDivisionError:
    logging.error("捕获到异常", exc_info=True)  # exc_info=True 会记录堆栈信息
```

### 日志的轮转（Rotating Logs）

当你需要将日志文件按大小或时间分割时，可以使用 `RotatingFileHandler` 或 `TimedRotatingFileHandler`。

- `RotatingFileHandler`: 当日志文件达到指定大小时，自动创建新文件并进行轮转。
- `TimedRotatingFileHandler`: 根据时间间隔（如每天、每小时等）进行日志文件的轮转。

```python
import logging
from logging.handlers import RotatingFileHandler

# 创建 RotatingFileHandler，每个日志文件大小为 1MB，最多保留 3 个备份
handler = RotatingFileHandler("app.log", maxBytes=1*1024*1024, backupCount=3)
logger = logging.getLogger("my_logger")
logger.addHandler(handler)

# 生成大量日志测试轮转
for i in range(10000):
    logger.info(f"这是第 {i} 个日志")
```

### 配置日志记录到多个目标

你可以通过添加多个 `Handler` 将日志输出到不同的目标（如控制台和文件）。

```python
import logging

# 创建 logger
logger = logging.getLogger("multi_handler_logger")
logger.setLevel(logging.INFO)

# 创建控制台 handler
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)

# 创建文件 handler
file_handler = logging.FileHandler("app.log")
file_handler.setLevel(logging.DEBUG)

# 添加 formatter
formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
console_handler.setFormatter(formatter)
file_handler.setFormatter(formatter)

# 将 handler 添加到 logger
logger.addHandler(console_handler)
logger.addHandler(file_handler)

# 生成日志
logger.info("这条日志会显示在控制台并记录在文件中")
logger.debug("这条日志只会记录在文件中")
```

## 总结

`logging` 模块提供了灵活且功能强大的日志记录功能，主要常见用法包括：

- 设置日志级别：控制输出的日志信息的详细程度。
- `basicConfig()`：快速配置日志输出目标、格式等。
- 使用 Logger、Handler 和 Formatter：实现复杂的日志记录需求。
- 捕获异常并记录详细的异常信息。
- 日志轮转：根据文件大小或时间自动切换日志文件。

通过灵活配置 `logging`，你可以轻松地将程序中的重要信息记录下来，便于调试和维护。

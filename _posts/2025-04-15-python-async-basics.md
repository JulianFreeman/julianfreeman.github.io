---
title: Python async 笔记
date: 2025-04-15 19:20:00 -0400
categories: [学习笔记, Python]
tags: [async]
description: 本文记录自己对 async 的理解
---

## 举个例子

为啥要举例子，主要是为了理解一下 **并发** 和 **并行**。

假如说有一个汉堡店，收到了 3 个汉堡的订单。为了更好理解，添加一些限定：汉堡店里的一台机器同时只能制作一个汉堡，一名店员同时只能操作一台机器。一台机器有三个功能：烙肉饼需要 1 分钟，烙面包需要 2 分钟，店员手动切蔬菜需要 3 分钟。

要制作这 3 个汉堡，有几种方法。

首先，只用一个店员，一台机器。这个店员是个 “同步” 店员，于是，当机器开始烙肉饼时，他就在那儿等着，等 1 分钟肉饼烙完了，再去烙面包，又等了 2 分钟，再去切菜。最后制作一个汉堡花了 6 分钟，3 个就需要 18 分钟。

店长一看这不行，又加购了 2 台机器，多雇了 2 个店员。但每个店员都是 “同步” 店员。

这时，3 个店员可以同时操作 3 台机器，但是每个汉堡依然需要 6 分钟制作完，但因为 3 个店员同时制作，总共就花了 6 分钟。

这就是 **并行**。

虽然买机器雇店员可以提升效率，但是成本太高，于是店长又卖掉了两台机器，解雇了 3 个 “同步” 店员，雇佣了 1 名 “异步” 店员。

这时，这名 “异步” 店员开始制作汉堡，当机器开始烙肉饼时，他不会傻傻地在那儿等着，而是着手开始烙上了面包，面包烙上之后，他依然没有停，转身去切菜了。这样一个汉堡制作完成总共就花了 3 分钟。3 个汉堡也就 9 分钟。

这就是 **并发**。

## 要点

并发并不是并行，不能提升程序运行的速度，它只是让线程在需要等待其他地方返回结果的时候，找点别的事情做，把这个等待的时间有效利用起来了。如果一段程序里压根没有需要等待的时候，线程一直都很忙，那并发也就没什么意义了。

比如上述的例子中，如果店员不是做汉堡，而是要炒一盘青椒炒肉，那他要先切菜，再切肉，再去炒，期间没有让他闲着的机会，那不管是 “同步” 店员还是 “异步” 店员，花的时间都一样了。

协程是由程序控制的一种轻量级的线程。程序可以进入异步模式，这个异步模式就是一个事件循环，在事件循环中会有很多个协程，事件循环控制着程序在多个协程之间反复横跳，执行着多个协程的代码。但其实同一时间还是只能执行一个协程，但是因为反复横跳的速度太快，看起来好像是在执行多个协程一样。如果每个协程都在忙，都没闲着，那其实这样反复横跳执行完所有协程的时间跟顺序执行完所有协程的时间是一样的。但如果个别协程是闲着的，比如在等待网络上的某个返回结果，暂时没事情做，那事件循环就可以找其他协程做事，这样就能把这部分时间节省下来，比顺序执行要快一些。因为顺序执行的时候如果一个任务在等待网络结果那整个程序都陪它一起等着，做不了别的。

## 上代码

> 以下为个人理解，可能有不准确的地方。只是为了方便理解，所以不进行深究。

使用协程并发的一种比较新的方式（在 Python 3.7 以上支持）是使用 `async` 语法。在函数定义的 `def` 之前加上 `async` 就变成了 `async def`，用 `async def` 定义的函数都会变成一个协程函数（coroutine function）。

```python
async def main():
    ...
```

与正常的函数不同的是，当用 `main()` 这样的方式调用协程函数的时候，不会返回函数执行的返回值，而是会返回一个协程对象（coroutine object）。**函数里面的代码并不会执行**。这一点非常重要，需要记住。

那怎么执行协程对象里面的代码呢，我们需要进入异步模式，开启事件循环，这基本上只用一个入口函数，就是 `asyncio.run`。该函数接受一个协程对象作为参数。

```python
import asyncio

async def main():
    print("hello")
    await asyncio.sleep(1)
    print("world")

asyncio.run(main())
```

开启事件循环之后，程序就开始依次执行传进来的协程对象（也就是那个参数）里的每一行代码。这就是一个协程了。在执行这个协程的过程中，它也可以去尝试切换到其他的协程，但是此时事件循环中只有它一个协程，所以没得切换。

当打印完 `hello` 之后，它遇到了 `await asyncio.sleep(1)`，这里的 `asyncio.sleep(1)` 也会返回一个协程对象，当我们 `await` 这个对象的时候，发生了这么几件事：

1. 这个对象会在事件循环中被注册为一个协程。
2. 这个协程对象里面的代码会被开始执行。
3. 等这里面的代码都执行完毕之后，返回值被拿到并返回（当然上述代码中没有用到返回值）。

在执行 `asyncio.sleep(1)` 这个协程的过程中，如果事件循环中还有其他协程，那么事件循环还是会尝试切换到其它协程的，但是有一个特殊情况是 `main()` 这个协程就不会继续执行了，它必须等 `asyncio.sleep(1)` 这个协程执行完了才能继续。这就是 “等” 的含义。

> 这里的 `asyncio.sleep` 不能换成 `time.sleep`，因为前者会被事件循环视为协程空闲，所以能去执行其他的协程，但后者会阻塞整个线程，不会被认为是空闲，所以并发没效果。

当然，上述代码中，其实除了 `asyncio.sleep(1)` 和 `main()` 就没其他协程了，所以事件循环到了 `asyncio.sleep(1)` 这一步也没其他事情可做。

那怎么增加协程呢？也许你会想到下面这种做法：

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)  # p3
    print(what)

async def main():
    t = time.time()
    await say_after(1, "hello")  # p1
    await say_after(2, "world")  # p2
    print(f"time: {time.time() - t:.2f}")

asyncio.run(main())  # p0
```

函数复杂了一些，但本质上其实还是没区别，调用了两次 `asyncio.sleep`。这里运行就能发现，时间还是过去了 3 秒。为什么呢，我们来逐步拆解一下。

p0 之后，开启异步模式，开启事件循环，此时只有 `main()` 一个协程。然后来到 p1，开启第二个协程 `say_after(1, "hello")`，`main()` 必须等它运行完代码才能继续，所以卡在这里了。然后 `say_after(1, "hello")` 运行到 p3，又开了一个协程 `asyncio.sleep(delay)`，但是协程 `say_after(1, "hello")` 必须等它完成才行，所以它也卡在这了。

现在分析一下现状，当前事件循环中一共有 3 个协程，`main()`，`say_after(1, "hello")` 和 `asyncio.sleep(delay)`。其中第一个要等第二个，第二个要等第三个，第三个虽然不用等，但是也没其他鞋协程切换了，所以大家一起等好了。什么，你说 p2 呢？还没运行到那儿呢，`say_after(2, "world")` 这个协程都还不存在呢。

所以，等 1 秒之后，`asyncio.sleep(delay)` 完事了，`say_after(1, "hello")` 解放了，可以往下执行了，期间如果有其他协程还是能切换的，只不过现在没有，没得切换。所以运行 `print(what)` 之后，`say_after(1, "hello")` 也完事了，`main()` 解放了，可以继续往下执行了，但此时事件循环中只有它一个协程了，也没什么能切换的。

来到 p2，`say_after(2, "world")` 这个协程现在才被创建，但是它的经历与 p1 也没什么两样，`main()` 要等 `say_after(2, "world")`，`say_after(2, "world")` 要等 `asyncio.sleep(delay)`，期间也没什么别的协程切换，于是大家又一起等了 2 秒。最后 `main()` 也结束了，程序结束，花了 3 秒。

到这里你会说，这跟不用异步有啥区别？的确从上述代码看没什么区别。但其实从上述分析中也能看到，每次在等待 `asyncio.sleep(delay)` 的时候，如果能有其他协程的话，是可以执行的，这部分时间能利用起来，只不过没有，那怎么办呢？怎么能有呢？怎么能让 p2 在 p1 开始之前就存在呢？

这就要说到 `asyncio.create_task` 函数了。该函数也接受一个协程对象，做了这么几件事：

1. 将这个协程对象注册为事件循环中的一个协程。
2. `asyncio.create_task` 函数 **所在的协程不会因此卡住**，而是会继续向下执行。
3. 该函数会返回一个 `Task` 对象，供之后使用。

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    t = time.time()
    task1 = asyncio.create_task(say_after(1, "hello"))
    task2 = asyncio.create_task(say_after(2, "world"))

    await task1
    await task2
    
    print(f"time: {time.time() - t:.2f}")

asyncio.run(main())
```

其实这里的 `await task1` 和 `await task2` 有些误导了，`task1` 和 `task2` 并不是到了 `await` 那一步才开始执行的，而是在创建之后有机会就已经开始执行了。哪怕不去 `await` 也会执行。但是如果没有 `await task1` 和 `await task2`，`main()` 这个协程就不会等待 `task1` 和 `task2` 执行完毕，它就一路走到底，直接运行完了整个程序，结束了。我们就看不到 `task1` 和 `task2` 的执行结果了。

所以必须得有个协程让 `main()` 等一会，等我们看到 `task1` 和 `task2` 的结果再结束程序，这个让 `main()` 等一会的，可以是 `await task1` 或 `await task2`，但也可以是其他的。比如

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    t = time.time()
    task1 = asyncio.create_task(say_after(1, "hello"))
    task2 = asyncio.create_task(say_after(2, "world"))

    await asyncio.sleep(3)

    print(f"time: {time.time() - t:.2f}")

asyncio.run(main())
```

这里我们没用上 `task1` 和 `task2`，但是运行程序我们依然是在 1 秒后看到了 `hello`，再 1 秒后看到了 `world`，再 1 秒后程序结束了。

但是注意，这里我们让 `main()` 等了 3 秒，所以我们才能看到 `task1` 和 `task2` 的结果，但如果只等了 0.5 秒，那依然啥也看不到，因为它俩还没执行完的，`main()` 又被解放了，又直接结束了。

因此如果我们希望 `main()` 能等两个 task 执行完再结束，我们可以 `await` 所需时间最长的那个 task。但是谁能知道哪个 task 需要时间最长呢？所以只好都 `await` 了。

但还有一个更好的办法，就是

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    t = time.time()
    task1 = asyncio.create_task(say_after(1, "hello"))
    task2 = asyncio.create_task(say_after(2, "world"))

    await asyncio.gather(task1, task2)

    print(f"time: {time.time() - t:.2f}")

asyncio.run(main())
```

这样，如果有 10 个 task 需要 `await`，我们也不用写十遍了，直接都丢给 `asyncio.gather`，然后就等它一个就好了。

这里还有一个简单的写法

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    t = time.time()
    await asyncio.gather(
        say_after(1, "hello"),
        say_after(2, "world")
    )
    print(f"time: {time.time() - t:.2f}")

asyncio.run(main())
```

不用 `create_task` 了，直接把协程对象丢给 `gather` 就好了。

所以现在想想，`say_after` 里为什么要 `await asyncio.sleep(delay)` 呢？不 `await` 的话，就直接 `print(what)` 然后 `say_after` 就结束了，我们就看不到 `asyncio.sleep` 的结果了。虽然这个函数没结果，但如果是其他函数呢，比如从网络请求数据的函数，就不能不要结果了。只是在等待这个结果的过程中，如果事件循环中还有其他协程，就可以先让它们做事。节省这部分等待的时间。


---
layout: post
title: VoiceMeeter Banana 教程
date: 2023-04-18 11:47:00 -0400
tags: voicemeeter
description: 关于理解 VoiceMeeter Banana 的使用和设置的一点介绍 # Add post description (optional)
img: voicemeeter-tutorial.jpg  # Add image post (optional)
---
## 前言

本文主要根据**个人理解**介绍 VoiceMeeter Banana（以下简称 VoiceMeeter），并使用 [VoiceMeeterSim][voicemeeter-sim] （以下简称模拟器）进行演示。

## VoiceMeeter 端口介绍

![voicemeeter-1]({{site.baseurl}}/assets/img/voicemeeter-tutorial/voicemeeter-1.png)

VoiceMeeter 有

- 三个硬件输入端口（HARDWARE INPUT 1/2/3）
- 两个虚拟输入端口（VoiceMeeter Input 和 VoiceMeeter Aux Input）
- 三个硬件输出端口（HARDWARE OUTPUT 1/2/3）
- 两个虚拟输出端口（VoiceMeeter Output 和 VoiceMeeter Aux Output）

接下来我们分别把

- 硬件输入端口称为 H1/H2/H3
- 虚拟输入端口称为 V1/V2
- 硬件输出端口称为 A1/A2/A3
- 虚拟输出端口称为 B1/B2

这就对应了模拟器的中间部分。

![sim-vm]({{site.baseurl}}/assets/img/voicemeeter-tutorial/sim-vm.png)

## 情景思考

一般情况下，系统输出的声音是通过扬声器传出来的，而系统输入的声音是通过麦克风传进去的。一些通讯软件（比如 Skype）或网站（比如 Messenger）需要获取电脑的扬声器或麦克风才能工作。

接下来我们以 Skype 为例。

我们可以把 Skype 的输入和输出设置想象成两个节点，一般情况下，Skype 的输入直接连接麦克风，而输出直接连接扬声器，所以我们说话的声音就能传进 Skype，而 Skype 的声音也能传出来。

![sim-skype-1]({{site.baseurl}}/assets/img/voicemeeter-tutorial/sim-skype-1.png)

> 注：模拟器中黄色表示音频输出，蓝色表示音频输入，声音是从左向右传输的。

如果此时我们用 Youtube 播放视频，浏览器默认使用系统输出设备来输出声音，而系统输出设备连接的也是扬声器，所以节点图就会如下：

![sim-skype-2]({{site.baseurl}}/assets/img/voicemeeter-tutorial/sim-skype-2.png)

但是如果我们想让 Skype 上通话的人能听到我们 Youtube 上播放的视频的声音，该怎么做呢？

单靠现有的内容是满足不了了，所以我们需要 VoiceMeeter。

## VoiceMeeter 的使用

简单来说，VoiceMeeter 提供了一些端口，允许我们把声音从一些端口中传进去，经过一些复杂或者不复杂的传递，再从另一些端口把声音传出来。

比如说，我们可以把 Youtube 的视频声音（系统输出）传入到一个虚拟输入端口，连接到另一个虚拟输出端口后把声音传到 Skype 的输入中。这样 Skype 通话中的人就能听到我们播放的 Youtube 的声音了。

![sim-skype-3]({{site.baseurl}}/assets/img/voicemeeter-tutorial/sim-skype-3.png)

此时只需要把系统和 Skype 进行一些设置：

系统输出设置：

![sim-sys-out]({{site.baseurl}}/assets/img/voicemeeter-tutorial/sim-sys-out.png)

Skype 输入设置：

![sim-skype-in]({{site.baseurl}}/assets/img/voicemeeter-tutorial/sim-skype-in.png)

> 其中 VoiceMeeter Input 用 V1 表示，VoiceMeeter Output 用 B1 表示。详见 [端口介绍](#voicemeeter-端口介绍)。

然后再把 VoiceMeeter 设置如下：

![voicemeeter-2]({{site.baseurl}}/assets/img/voicemeeter-tutorial/voicemeeter-2.png)

这是与模拟器节点图所对应的实际设置。除了涉及到的端口，其余的没有设置，比如麦克风输入以及系统声音输出等。所以现在的情况是 Skype 上的朋友可以听到我们电脑的 Youtube 声音，但我们自己听不到。

如果我们希望

- 能听到自己电脑的 Youtube 的声音
- Skype 上的朋友除了能听到 Youtube 的声音之外，同时也能听到我们的说话声
- 我们也能听到 Skype 上的朋友的说话声

那么只需要

- A1 选择扬声器，V1 连接 A1
- H1 选择麦克风，H1 连接 B1
- Skype 的输出设备选择 V1（VoiceMeeter Input）

节点图如下：

![sim-skype-4]({{site.baseurl}}/assets/img/voicemeeter-tutorial/sim-skype-4.png)

实际的 VoiceMeeter 设置如下：

![voicemeeter-3]({{site.baseurl}}/assets/img/voicemeeter-tutorial/voicemeeter-3.png)

## 重音的烦恼

虽然从节点图上看，Skype 的输出通过一定的连接传到了 Skype 的输入里，但实际测试这里并不会出现重音或者回音，可能是软件本身阻断了这种声音传输吧（猜的），所以这样的连接是安全的。我们可以认为这样的连接虽然看上去连着，但实际上是不通的。

不过这种阻断应该只限制在同一个程序内。就是说，如果我开了两个 Skype，我把两个 Skype 的输入输出都设置为上述设置，那么 Skype1 的输出不会自传到 Skype1 的输入（同一程序内被阻断了），但是会传递到 Skype2 的输入中（两个程序之间正常传递），反之亦然。所以这样的话，两个 Skype 端的朋友们其实是可以互相通话，而不会有重音的。

> 暂时这样猜想，以后有机会验证。

## Cable 虚拟线缆

VoiceMeeter 的官方网站同样还提供一个 Virtual Audio Cable 软件的下载。该软件安装后电脑会多出两个虚拟的输入输出端口

- CABLE Input（虚拟输入）
- CABLE Output（虚拟输出）

简单来说，**所有从 CABLE 输入端进去的声音都会从 CABLE 输出端出来**。

但是要使用这个东西的话，单纯靠脑子去想这个声音的流向，反正我是理不清楚，所以我设计了[模拟器][voicemeeter-sim]。呵呵。

比如，如果我们想让 Skype 上的朋友听到我们 Youtube 视频的声音，那就可以

- 系统输出连接 CABLE Input
- Skype 输入连接 CABLE Output

节点图及其简单：

![sim-cable-1]({{site.baseurl}}/assets/img/voicemeeter-tutorial/sim-cable-1.png)

> 实际操作中没有 Cable 输入连接 Cable 输出这一步，因为这是软件内部默认实现的。

但是单是上面这样的连接，我们自己听不到 Youtube 的声音，怎么办呢？系统输出已经选择了 CABLE Input，就不能再选择其他端口了，那怎么才能把系统输出的声音输出到其他地方呢？

我们可以把 CABLE Output 连接到 H2 上！（当然 H1/H2/H3 都行，没啥区别，哪个位置空着就用哪个）

然后再把 H2 连接到 A1 上，A1 选择的是扬声器，这样我们就能听到 Youtube 的声音了。

![sim-cable-2]({{site.baseurl}}/assets/img/voicemeeter-tutorial/sim-cable-2.png)

VoiceMeeter 实际设置如下：

![voicemeeter-4]({{site.baseurl}}/assets/img/voicemeeter-tutorial/voicemeeter-4.png)

同理，CABLE Input 也可以接到 A1/A2/A3 上，这个就不在这里演示了。

其实这里发现啥呢，原本系统输出不能连接到硬件输入的，只能连接到虚拟输入，比如 V1 或者 V2，但是有了 Cable 之后，我们就可以通过形如系统输出连接到 Cable 输入、Cable 输出连接到硬件输入的方式，间接地把系统输出连接到硬件输入了，相当于扩展了端口！

应该会有些用处吧。呵呵。

## 结语

关于 VoiceMeeter 的介绍先说这么多。当然这个软件的功能远不止这么多，但是介绍这些应该也就能应对平常的使用了吧。

主要还是希望 [VoiceMeeterSim][voicemeeter-sim] 这个模拟器能帮一些忙吧。

---

封面图：Photo by [Thomas Le][cover-img-author] on [Unsplash][cover-img-src]

[voicemeeter-sim]: https://julianfreeman.github.io/VoiceMeeterSim/

[cover-img-author]: https://unsplash.com/@thomasble?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
[cover-img-src]: https://unsplash.com/photos/bPD4U-mqC44?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText

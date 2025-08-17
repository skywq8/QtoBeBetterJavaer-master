---
title: 大白话+手绘图带你认识 JVM，JVM到底是什么？
shortTitle: 大白话带你认识JVM
category:
  - Java核心
tag:
  - Java虚拟机
description: JVM是Java程序执行的环境，它隐藏了底层操作系统和硬件的复杂性，提供了一个统一、稳定和安全的运行平台。
head:
  - - meta
    - name: keywords
      content: Java,JavaSE,教程,二哥的Java进阶之路,jvm,Java虚拟机
---


“二哥，之前你讲 Java 的[第一行代码 hello world](https://javabetter.cn/overview/hello-world.html) 的时候就提到了 JVM，那时候我就想知道 JVM 到底是什么，但你说这是一块非常大的内容，会放到后面专门来讲，那学完了 [Java 基础知识](https://javabetter.cn/overview/)，又学完了[并发编程](https://javabetter.cn/thread/)，今天我们就来学习 JVM 吧？”三妹咪了一口麦香可可奶茶后对我说。

“好的，三妹，这篇内容就来带你认识一下什么是 JVM， JVM 是 Java 体系中非常重要，又有一些难度的知识，但每个想要更加优秀的程序员都应该掌握它。尤其是想去大厂或者中厂的球友，更应该掌握它，因为 JVM 在大中厂面试的时候，比重很大，我随便从《[Java 面试指南](https://javabetter.cn/zhishixingqiu/mianshi.html)》中截张图大家感受一下。”我回答。

![](https://cdn.tobebetterjavaer.com/stutymore/what-is-jvm-20231223101555.png)

“JVM 在校招面试中的比重还是非常大的；同时，对于工作党来说，如果项目遇到内存泄露、CPU飙升的问题，也需要通过 JVM 的性能监控进行定位和解决。好，那就让我们开始吧！”我继续补充道。

三妹，你看过《[Java 发展简史](https://javabetter.cn/overview/what-is-java.html)》应该知道，Sun 在 1991 年成立了一个由詹姆斯·高斯林（James Gosling）领导的，名为“Green”的项目组，目的是开发一种能够在各种消费性电子产品上运行的程序架构。

一开始，项目组打算使用 C++，但 C++ 无法达到跨平台的要求，比如在 Windows 系统下编译的 Hello.exe 无法直接拿到 Linux 环境下执行。

![](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/overview/seven-01.png)

在当时，C++ 已经非常流行了，但无法跨平台，只能忍痛割爱了。

怎么办呢？

三妹不知道有没有听过直译器（解释器）这玩意？（估计你没听过）就是每跑一行代码就生成机器码，然后执行，比如说 Python 和 Ruby 用的就是直译器。

“那在每个操作系统上装一个直译器就好了，跨平台的目的就达到了啊。”三妹插话道。

![](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/overview/seven-02.png)

“但直译器有个缺点，就是没法像编译器那样对一些热点代码进行优化，从而让程序在机器上跑得更快一些。”我回答说。

怎么办呢？

来个结合体呗，编译器和直译器一块上！

![](https://cdn.tobebetterjavaer.com/stutymore/what-is-jvm-20231019153456.png)

编译器负责把 Java 源代码编译成[字节码](https://javabetter.cn/jvm/bytecode.html)，Java 虚拟机负责把字节码转换成机器码。转换的时候，可以做一些压缩或者优化，通过 [JIT](https://javabetter.cn/jvm/jit.html) 来完成，这样的程序跑起来就快多了。

![](https://cdn.tobebetterjavaer.com/stutymore/what-is-jvm-20231223102608.png)

>- Java 虚拟机：Java Virtual Machine，简称 JVM，也就是我们接下来要学习的重点。
>- 字节码：[Bytecode](https://javabetter.cn/jvm/bytecode.html)，接下来会细讲。
>- JIT：Just-In-Time，[即时编译器](https://javabetter.cn/jvm/jit.html)，后面会细讲。

这样的话，不仅跨平台的目的达到了，而且性能得到了优化，两全其美！

“为什么 Java 虚拟机会叫 Java 虚拟机呢？”三妹问了一个很古怪的问题。

虚拟机，顾名思义，就是虚拟的机器（多苍白的解释），反正就是看不见摸不着的机器，一个相对物理机的叫法，你把它想象成一个会执行字节码的怪兽吧。

记得上大学那会，由于没有 Linux 环境，但又需要在上面玩一些命令，于是我就会在 Windows 上装一个 Linux 的虚拟机，这个 JVM 就类似这种东西。

说白了，就是我们编写 Java 代码，编译 Java 代码，目的不是让它在 Linux、Windows 或者 MacOS 上跑，而是在 JVM 上跑。

## JVM 家族

我刚讲完这些，三妹就问：“都有哪些 Java 虚拟机呢？”

来看下面这张思维导图，一目了然。

![](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/overview/seven-04.png)

除了我们经常看到、听到的 Hotspot VM，还有很多，下面我来简单介绍一下。

- Sun Classic：世界上第一款商用 Java 虚拟机，但执行效率低下，导致 Java 程序的性能和 C/C++ 存在很大差距，因此给后来者留下了“Java 语言很慢”的刻板印象。
- Exact VM：为了提升 Classic 的效率，Sun 的虚拟机团队曾在 Solaris（Sun 研发的一款类似 Unix 的操作系统）上发布过这款虚拟机，它的执行系统里包含有热点探测、即时编译等，但不是很成熟。Sun Classic 在 JDK 1.4 的时候被彻底抛弃，而 Exact VM 被抛弃得更早，取代它的正是 HotSpot VM——时也命也。
- Mobile VM：Java 在移动手机端（被 Android 和 IOS 二分天下）的发展并没有那么成功，因此 Mobile VM 的声望值比较低。
- Embedded VM：嵌入式设备上的虚拟机。
- BEA JRockit：曾经号称是“世界上最快的 Java 虚拟机”，后来被 Oracle 收购后就没有声音了。
- IBM J9 VM：提起 IBM，基本上所有程序员都知道了，也是个巨头，所以他家的虚拟机也很强，在职责分离和模块化上做得比 HotSpot 更好。目前已经开源给 Eclipse 基金会。
- BEA Liquid VM：是 BEA 公司开发的可以直接运行在自家系统上的虚拟机，可以越过操作系统直接和硬件打交道，因此可以更大程度上的发挥硬件的能力。不过核心用的还是 JRockit，所以伴随着 JRockit 的消失，Liquid VM 也退出历史舞台了。
- Azul VM：是 Azul 公司在 HotSpot 基础上进行大量改进后的，可以运行在 Azul 公司专有硬件上的虚拟机。2010 年起，Azul 公司的重心从硬件转移到软件上，并发布了 Zing 虚拟机，性能方面很强大。
- Apache Harmony 和 Google Android Dalvik VM 并不是 严格意义上的 Java 虚拟机，但对 Java 虚拟机的发展起到了很大的刺激作用。但它们终究没有熬过时间。
- Microsoft JVM：在早期的 Java Applets 年代，微软为了在 IE 中支持 Applets 开发了自己的 Java 虚拟机。你敢相信？Microsoft JVM 只有 Windows 版本，它与 JVM 实现的“一次编译，到处运行”的理念完全沾不上边。关键是，1997 年 10 月，Sun 公司因为这事把微软告了，最后微软赔给了 Sun 公司 2000 万美金，并且终止了在 Java 虚拟机方面的发展。如果，我是说如果，如果微软保持着对 Java 的热情，后面还有 .Net 什么事？

![](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/overview/seven-05.png)

最后再来讲一下：HotSpot VM，它是OracleJDK（商用）和 OpenJDK（开源）的默认虚拟机，也是目前使用最广泛的 Java 虚拟机，也就是传说中的太子，Java 虚拟机中的嫡长子。

HotSpot 的技术优势就在于热点代码探测技术（名字就从这来的）和准确式内存管理技术，但其实这两个技术在 Exact VM 中都有体现，因此你看起个好的名字多重要（开玩笑了，这就是命）。

热点代码探测，指的是，通过执行计数器找出最具有编译价值的代码，然后通知即时编译器以方法为单位进行编译，解释器就可以不再逐行的将字节码翻译成机器码，而是将一整个方法的所有字节码翻译成机器码再执行。

这样的话，效率就提高了很多，对吧？也就是我们后面会讲到的 [JIT 技术](https://javabetter.cn/jvm/jit.html)。

## JVM 的组织架构

“JVM 的组织架构是什么样子的呢？它由哪些单位组成的呢？”三妹继续追问到。

JVM 大致可以划分为三个部门，分别是类加载器（Class Loader）、运行时数据区（Runtime Data Areas）和执行引擎（Excution Engine），见下图。

![](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/overview/seven-06.png)

这三个部门具体又是干什么的，可以通过下面这幅图来了解。

![](https://cdn.tobebetterjavaer.com/stutymore/what-is-jvm-20231030185742.png)

### 类加载器

类加载器是 JVM 最有权威的一个部门，相当于明朝张居正时期的内阁，大全独揽，朝廷想干什么，都得经过我这关。

好，类加载器用来加载[类文件](https://javabetter.cn/jvm/what-is-jvm.html)，也就是 .class 文件。如果类文件加载失败，也就没有运行时数据区和执行引擎什么事了，它们什么也干不了。

类加载器负责将字节码文件加载到内存中，主要会经历加载->连接->实例化这三个阶段，我们会放在[后面的章节单独来讲](https://javabetter.cn/jvm/class-load.html)。

### 运行时数据区

运行时数据区就相当于明朝时期的国库，国库里有钱，那接下来的执行引擎就能够继续执行字节码，国库里没钱就会抛出 OutOfMemoryError 异常。

JVM 定义了 Java 程序运行期间需要使用到的内存区域，简单来说，这块内存区域存放了字节码信息以及程序执行过程的数据，[垃圾收集器](https://javabetter.cn/jvm/gc-collector.html)也会针对运行时数据区进行对象回收的工作。看下面这张图就能理解（JVM 规范）：

![](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/overview/seven-07.png)

运行时数据区通常包括：方法区、堆、虚拟机栈、本地方法栈以及程序计数器五个部分。不过，运行时数据区的划分也随着JDK的发展不断变迁，JDK 1.6、JDK 1.7、JDK 1.8 的内存划分都会有所不同。

![](https://cdn.tobebetterjavaer.com/stutymore/neicun-jiegou-20240110195211.png)

我们会在[后面的章节单独来讲](https://javabetter.cn/jvm/neicun-jiegou.html)。

### 执行引擎

执行引擎（Execution Engine）就好像明朝时期的六部，主要用来干具体的事，“虚拟机”是一个相对于“物理机”的概念，这两种机器都有代码执行能力，其区别是物理机的执行引擎是直接建立在处理器、缓存、指令集和操作系统层面上的，而**虚拟机的执行引擎则是由软件自行实现的**，因此可以不受物理条件制约地定制指令集与执行引擎的结构关系，能够执行那些不被硬件直接支持的指令集格式。

执行引擎的任务就是将[字节码指令](https://javabetter.cn/jvm/zijiema-zhiling.html)解释/编译为对应平台上的本地机器指令才可以。简单来说，JVM 中的执行引擎充当了将高级语言翻译为机器语言的译者。

![](https://cdn.tobebetterjavaer.com/stutymore/what-is-jvm-20231223155202.png)

- 解释器：读取字节码，然后执行指令。因为它是一行一行地解释和执行指令，所以它可以很快地解释字节码，但是执行起来会比较慢（毕竟要一行执行完再执行下一行）。
- [即时编译器](https://javabetter.cn/jvm/jit.html)：执行引擎首先按照解释执行的方式来执行，随着时间推移，即时编译器会选择性的把一些热点代码编译成本地代码。执行本地代码比一条一条进行解释执行的速度快很多，因为本地代码是保存在缓存里的。
- [垃圾回收器](https://javabetter.cn/jvm/garbage-collector.html)，用来回收堆内存中的垃圾对象。

这些内容我们都会在后面的章节里单独来细讲。

## 小结

“三妹，关于 JVM 是什么的话题我们就先讲到这，后面再展开细讲其中的每一个细节，怎么样？”转动了一下僵硬的脖子后，我对三妹说，“JVM 是一块很大很深的内容，如果一上来学太多的话，我怕难倒你（😁）。”

“好的，二哥，我也觉得今天的知识量够了，我要好好消化几天。我会加油的！”三妹似乎对未来充满了希望，这正是我想看到的。

总的来说，JVM 是 Java 程序执行的环境，它隐藏了底层操作系统和硬件的复杂性，提供了一个统一、稳定和安全的运行平台。

![](https://cdn.tobebetterjavaer.com/stutymore/what-is-jvm-20231223154127.png)

我们把 Java 源代码编译后的字节码文件扔给它，它就可以在 JVM 中执行，不管是在 Windows、Linux 还是 MacOS 环境下编译的，它都可以跑，屏蔽了底层操作系统的差异。

----

GitHub 上标星 10000+ 的开源知识库《[二哥的 Java 进阶之路](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，500+张手绘图，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 10000+ 的 Java 教程](https://javabetter.cn/overview/)


微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)

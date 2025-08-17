---
title: Java 14 开箱，新特性Record、instanceof、switch香香香香
shortTitle: Java 14 新特性
category:
  - Java核心
tag:
  - Java新特性
description: 本文详细介绍了Java 14的新特性，包括Record、instanceof和switch语句的改进。通过实际的代码示例，展示了如何使用这些新特性来简化代码，提高编程效率。学习本文，让您快速了解Java 14的新亮点，领略新特性带来的编程魅力。
head:
  - - meta
    - name: keywords
      content: Java,Java SE,Java基础,Java教程,二哥的Java进阶之路,Java进阶之路,Java入门,教程,java8,java Record,java instanceof,java switch
---


Java 14 的时候，新增了记录 Record、模式匹配 instanceof 等新特性，转正了 Java 12 时的 switch 表达式，我们一起来过一遍。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMTc5Mzg5LTA2N2E2NGZiZWY3OTU0MTAucG5n?x-oss-process=image/format,png)

### 01、下载 JDK 14

> 截止到2023年03月30日，Java 20 已经发布了，不过不是 LTS（长期支持）版本，Java 17、11、8 是目前提供支持的 LTS 版本。

要想开箱，得先下载 JDK 14（如果你有更高版本，当然也可以），不然拿什么开箱呢，对吧？有 2 处地方可供下载，Oracle 上可以下载商用版， [jdk.java.net](https://jdk.java.net/14/)  上可以下载开源版。我们就选择后者吧。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMTc5Mzg5LTJkM2Q3MGQ4ODQ0NDk1MDEucG5n?x-oss-process=image/format,png)

我目前用的是 Windows 操作系统，所以就选择 Windows 版的 zip 包进行下载，完成后记得解压。

### 02、升级 IntelliJ IDEA

>截止到 2023年03月30日，Intellij IDEA 的最新版本是 2023.1。 

需要把 IDEA 升级到抢先体验版 2020.1 EAP（如果你当前使用的版本大于此，当然也可以），否则无法支持 Java 14 的新特性。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMTc5Mzg5LThhYjFjYmRiNDA3MjBiM2UucG5n?x-oss-process=image/format,png)

社区版的下载地址如下所示：

```
[https://www.jetbrains.com/idea/nextversion/#section=windows](https://www.jetbrains.com/idea/nextversion/#section=windows)
```

安装的时候可以把之前的版本卸载，也可以选择保留。完成后，我们来新建一个 Java 14 的项目。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMTc5Mzg5LTBhYWIwYWNiY2RiMzllZGIucG5n?x-oss-process=image/format,png)

### 01、instanceof

按照新特性的顺序，我们就先从 instanceof 说起吧。旧式的 instanceof 的用法如下所示：

```java
public class OldInstanceOf {
    public static void main(String[] args) {
        Object str = "Java 14，真香";
        if (str instanceof String) {
            String s = (String)str;
            System.out.println(s.length());
        }
    }
}
```

需要先使用 instanceof 在 if 条件中判断 str 的类型是否为 String（第一步），再在  if 语句中将 str 强转为字符串类型（第二步），并且要重新声明一个变量用于强转后的赋值（第三步）。

三个步骤也不算多，但总觉得应该还有更好的语法，这不，Java 14 就想到了这一层。

```java
public class NewInstanceOf {
    public static void main(String[] args) {
        Object str = "Java 14，真香";
        if (str instanceof String s) {
            System.out.println(s.length());
        }
    }
}
```

可以直接在 if 条件判断类型的时候添加一个变量，就不需要再强转和声明新的变量了。是不是特别简洁？但模式匹配的 instanceof 在 Java 14 中是预览版的，默认是不启用的，所以这段代码会有一个奇怪的编译错误（Java 14 中不支持模式匹配的 instanceof）。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMTc5Mzg5LTg1MTQ4YWI3MjI2ZmM4ZTUucG5n?x-oss-process=image/format,png)

那怎么解决这个问题呢？需要在项目配置中手动设置一下语言的版本。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMTc5Mzg5LTdhMzE3MDYwNjk5NTJmMDgucG5n?x-oss-process=image/format,png)

设置完成后，编译错误就随风飘走了。程序输出的结果如下所示：

```
10
```

不错不错，真香。想知道 Java 编译器在背后帮我们做了什么吗？看一下反编译后的字节码就明白了。

```java
public class NewInstanceOf {
    public NewInstanceOf() {
    }

    public static void main(String[] args) {
        Object str = "Java 14，真香";
        String s;
        if (str instanceof String && (s = (String)str) == (String)str) {
            System.out.println(s.length());
        }

    }
}
```

在 if 条件判断前，先声明了变量 s，然后在 if  条件中进行了强转 `s = (String)str)`，并且判断了 s 和 str 是否相等。确实是一个解放开放者生产力的好特性，强烈希望这个特性在下个版本中转正。

### 02、Records

在之前的一篇文章中，我谈到了[类的不可变性](https://javabetter.cn/basic-extra-meal/immutable.html)，它是这样定义的：

```java
public final class Writer {
    private final String name;
    private final int age;

    public Writer(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public int getAge() {
        return age;
    }

    public String getName() {
        return name;
    }
}
```

那么，对于 Records 来说，一条 Record 就代表一个不变的状态。尽管它会提供诸如 `equals()`、`hashCode()`、`toString()`、构造方法，以及字段的 getter，但它无意替代可变对象的类（没有 setter），以及 Lombok 提供的功能。

来用 Records 替代一下上面这个 Writer 类：

```java
public record Writer(String name, int age) { }
```

你看，一行代码就搞定。关键是比之前的代码功能更丰富，来看一下反编译后的字节码：

```java
public final class Writer extends java.lang.Record {
    private final java.lang.String name;
    private final int age;

    public Writer(java.lang.String name, int age) { /* compiled code */ }

    public java.lang.String toString() { /* compiled code */ }

    public final int hashCode() { /* compiled code */ }

    public final boolean equals(java.lang.Object o) { /* compiled code */ }

    public java.lang.String name() { /* compiled code */ }

    public int age() { /* compiled code */ }
}
```

类是 final 的，字段是 private final 的，构造方法有两个参数，`toString()`、`hashCode()`、`equals()` 方法也有了，getter 方法也有了，只不过没有 get 前缀。但是没有 setter 方法，也就是说 Records 确实针对的是不可变对象——鉴定完毕。那怎么使用 Records 呢？

```java
public class WriterDemo {
    public static void main(String[] args) {
        Writer writer = new Writer("沉默王二",18);
        System.out.println("toString：" + writer);
        System.out.println("hashCode：" + writer.hashCode());
        System.out.println("name：" + writer.name());
        System.out.println("age：" + writer.age());

        Writer writer1 = new Writer("沉默王二", 18);
        System.out.println("equals：" + (writer.equals(writer1)));
    }
}
```

程序输出的结果如下所示：

```
toString：Writer[name=沉默王二, age=18]
hashCode：1130697218
name：沉默王二
age：18
equals：true
```

不错不错，真香，以后定义不可变类时就简单了，强烈希望这个特性在下个版本中转正。

### 03、switch 表达式

关于 switch 表达式，我在之前的一篇文章中已经详细说明了，点击[传送门](https://mp.weixin.qq.com/s/1BDDLDSKDGwQAfIFMyySdg)可以跳转过去看看。两周时间过去了，switch 表达式终于“媳妇熬成婆”，转正了，恭喜恭喜。

记得这篇文章发表到掘金的时候，被喷子各种无脑 diss，说：“还以为你有什么技巧，没想到用的是 Java 13，可我们还停留在 Java 8 啊！”这显然是一种固步自封的心态，非常不可取，程序员不应该这样。一个最简单的道理就是，Java 6 当年也很经典，不是被 Java 8 取代了吗？随着时间的推移，Java 8 早晚会被更划时代的新版本取代——总要进步嘛。

关于 switch 表达式，这里就简单地搬个例子给你瞧瞧：

```java
public class SwitchDemo {
    enum PlayerTypes {
        TENNIS,
        FOOTBALL,
        BASKETBALL,
        PINGPANG,
        UNKNOWN
    }

    public static void main(String[] args) {
        System.out.println(createPlayer(PlayerTypes.BASKETBALL));
    }

    private static String createPlayer(PlayerTypes playerType) {
        return switch (playerType) {
            case TENNIS -> "网球运动员费德勒";
            case FOOTBALL -> "足球运动员C罗";
            case BASKETBALL -> "篮球运动员詹姆斯";
            case PINGPANG -> "乒乓球运动员马龙";
            case UNKNOWN -> throw new IllegalArgumentException("未知");
        };
    }
}
```

除了可以使用 `->` 的新式语法，还可以作为 return 结果，真香。

### 04、Text Blocks

在文本块（Text Blocks）出现之前，如果我们需要拼接多行的字符串，就需要很多英文双引号和加号，看起来就好像老太婆的裹脚布，非常不雅。如果恰好要拼接一些 HTML 格式的文本（原生 SQL 也是如此）的话，还要通过空格进行排版，通过换行转义符 `\n` 进行换行，这些繁琐的工作对于一名开发人员来说，简直就是灾难。

```java
public class OldTextBlock {
    public static void main(String[] args) {
        String html = "<html>\n" +
                "    <body>\n" +
                "        <p>Hello, world</p>\n" +
                "    </body>\n" +
                "</html>\n";
        System.out.println(html);
    }
}
```

Java 14 就完全不同了：

```java
public class NewTextBlock {
    public static void main(String[] args) {
        String html = """
              <html>
                  <body>
                      <p>Hello, world</p>
                  </body>
              </html>
              """;
        System.out.println(html);
    }
}
```

多余的英文双引号、加号、换行转义符，统统不见了。仅仅是通过前后三个英文双引号就实现了。我只能说，香，它真的香！

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xMTc5Mzg5LTU3Njg1YmQzZDdmNzRkMDcuZ2lm)

----

GitHub 上标星 10000+ 的开源知识库《[二哥的 Java 进阶之路](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括Java基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM等等，共计 32 万余字，500+张手绘图，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 10000+ 的 Java 教程](https://javabetter.cn/overview/)


微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。

![](https://cdn.tobebetterjavaer.com/tobebetterjavaer/images/gongzhonghao.png)




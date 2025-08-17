---
title: 工作六年，我学会了用 Arthas 来辅助我的日常工作 
shortTitle: 工作六年，我学会了用 Arthas 来辅助我的日常工作 
description: 如何通过 arthas 来解决日常工作中的疑难问题，如何通过 arthas 处理工作以前需要 debug，需要打印日志才能找的 bug。 集合案例来谈谈如何使用 arthas 这些命令。
tag:
  - 优质文章
author: uzong
category:
  - 掘金社区
head:
  - - meta
    - name: keywords
      content: Java
---

## 工作六年，我学会了用 Arthas 来辅助我的日常工作

*很久就想写一篇介绍 Arthas 的文章，虽然 Arthas 已有大量文章介绍；但我依然想结合我的实际工作，来聊聊这款我喜爱的 Java 监控诊断产品。*

> 🔊一位 Java 开发者的使用总结，只谈使用经验，不聊原理。

## 📆 那些辛酸的过往

***历历在目的场景*🥹**(❁´◡`❁)(❁´◡`❁)

*   客户线上问题，应该如何复现，让客户再点一下吗？
*   异常被吃掉，手足无措，看是哪个家伙写的，竟然是自己！
*   排查别人线上的 bug，不仅代码还没看懂，还没一行日志，捏了一把汗！
*   预发 debug，稍微时间长点，群里就怨声载道！
*   加日志重新部署，半个小时就没了，问题还没有找到，头顶的灯却早已照亮了整层楼......
*   线上机器不能 debug，也不能开 debug 端口，重新部署会不会破坏现场呢?
*   怀疑入参有问题，怀疑合并代码有问题，怀疑没有部署成功，全是问号......
*   一个问题排查一天，被 Diss 排查问题慢......

> *直到我遇到了 Arthas，那些曾经束手无策的问题，都变得轻而易举。于是想把这些遇到的场景和用法做个总结。*

## 📕一、通过命令执行方法--vmtool

**vmtool 命令是我最喜欢用的，也是用最多的一个命令。通过这个命令执行方法，检查各种不符合预期的分支逻辑，入参出参，以及各种外部依赖接口，甚至还能修改数据等。**

#### 1.1 场景

解决过的场景|具体描述|
---|---|
发布导致线上的缓存 key 错误，需要清理，但过期时间还长，没有删除 key 的远程接口|通过执行 service 方法，删除缓存 key；另外读取 redis 中的 key 也极其方便|
缺少日志，不知道上游是否返回数据合理|通过执行方法，确定依赖返回数据不正确|
发布应用同时修改分布式配置，导致推送配置到该节点失败|通过执行方法，查询配置信息不是最新|
常量值不符合预期，配置在 properties 中的免登 url 失效|通过执行方法，查询当前常量值，判断读取不合理|
新增配置信息、删除脏数据|通过接口执行方法，添加了配置、删除了脏数据|
集群环境，想要请求打在指定机器上查看日志|需要反复请求多次才能命中特定机器查看日志，通过vmtool 执行方法，快速实现日志查看|
出参入参不符合预期|在调用链路上执行所有可疑方法|
以前需要写 controller 调用触发的测试方法|直接用这个命令，减少代码，还能测试上下游的各种二方接口，十分方便|

*案例还有很多很多，因为真的可以拿着入参尽情的 invoke*

提升了排查问题、解决问题的效率，也帮助其他人解决他们的问题。不再依赖打印大量日志反复部署服务，也不再申请 debug 端口进行远程 debug ，因为确实方便。

#### 1.2 使用

工欲善其事必先利其器，我在 IDEA 装上一个 Arthas 插件，用它来快速复制命令，想执行哪个方法拷贝即可。 

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a19a329d1834bccab44454f811f01ec~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1563&h=515&s=88254&e=png&b=3c3f41)

> 上图是使用 Arthas 插件生成执行命令。光标放在执行方法上右击选择 vmtool 即可得到可运行命令。

**情景一： 执行的方法是对象**：需要对参数对象赋值，以下图中的方法为例：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da59573084794985be61f1da3bb246f7~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2038&h=430&s=106438&e=png&b=3b3b3b)

`queryIBCcContactConfig` 方法参数是对象， 首先通过 Arthas 工具查找到参数 `IbCcContactConfigParam` 的**classLoaderHash**， 如下命令：（sc -d 路径）

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35d04f03ee5d44cfbdea15fec56b281e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1507&h=1089&s=629859&e=png&b=101010)

对参数对象进行字段赋值,方式参考下面加粗部分（ognl方式）：

> vmtool  -x 3 -c 76707e36 --action getInstances --className com.xxx.impl.IbCcContactConfigServiceImpl --express 'instances\[0\].queryIbCcContactConfig(**(#demo=new com.xxx.param.IbCcContactConfigParam(), #demo.setContactBizCode('12345L'),#demo)**)'

情景二、基础类型，比如String、Long、Int。直接填充数值即可

举例|语句|描述|
---|---|---|
基础类型，比如String、Int、Long类型等|vmtool -x 3 --action getInstances --className com.xxl.mq.admin.service.IXxlMqMessageService --express 'instances\[0\].delete(0)'|执行 IXxlMqMessageService#delete 方法，参数为0|
|||

参数|解释|
---|---|
\-x 3|返回参数展开形式的，默认1，如果返回体是对象，建议设置3，方便观察返回结果|
\-c xxx|指定classLoaderHash，如果不指定，new 方法报错|

#### 1.3 限制

其一、尽量避开 ThreadLocal 。执行线程没有 ThreadLocal 的上下文；

其二、只能有一个端口，只支持一个arthas-server，用完及时关掉。

#### 1.4 扩展

使用 getstatic 命令查看静态变量

场景描述|语句执行|解释|
---|---|---|
查看静态变量的实际值|getstatic com.xxx.xxx.interceptor.env.EnvIsolationInterceptor FILL\_FIELD\_NAME -x 3|查看 EnvIsolationInterceptor # FILL\_FIELD\_NAME 的静态变量值|
配置application.properties的免登 uri，发现没有生效|getstatic com.xxx.sso.filter.InitParamContext excludeList -x 3|查看自己的免登 uri 是否在集合里面，从而快速定位问题|
修改静态变量值|ognl -x 3 '#field=@[com.xxl.mq.admin.conf.XxlMqBrokerImpl@class.getDeclaredField](https://link.juejin.cn?target=mailto%3Acom.xxl.mq.admin.conf.XxlMqBrokerImpl%40class.getDeclaredField "mailto:com.xxl.mq.admin.conf.XxlMqBrokerImpl@class.getDeclaredField")("ENV"),#field.setAccessible(true),#field.set(null," ")'|![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d60784f5a53d4326917c49af16a3425d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1737&h=327&s=72781&e=png&b=3c3f41)
|

## 🖥️二、热部署 # redefine && retransform

😭**拍桌子拍大腿感叹发布的的代码少写或者漏写；拍脑门惋惜为啥不多打一行日志；口吐芬芳为什么把判断条件写死......，那些只能发布才能调试、部署一次要半小时的应用，真的会让生命变得廉价。**

#### 2.1 场景

解决过的场景|描述|
---|---|
加日志语句，入参出参观察|联调查看参数|
将判断条件恒定成了 false，目标分支无法执行，阻塞进度|修改判断逻辑|
漏写一行赋值代码|对象自己赋值给自己，字段值为NULL|
研发、联调阶段，代码验证|需要反复修改代码验证|
测试同学提Bug及时修复验证|快速修复问题，不影响测试进度|

**热部署的优势用过的都说好**👍。

#### 2.2 使用

IDEA 集成 ArthasHotSwap 插件，方便快捷：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6dd3e1fc0dc049e0af782fcf608dd915~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1295&h=561&s=128313&e=png&b=343638)

> 很多公司通过 API 方式已经集成了工具，定制化更好用😄。

#### 2.3 限制

*   不允许新增加 field/method
*   正在跑的函数，没有退出不能生效
*   redefine 和 watch/trace/jad/tt 等命令冲突

热部署能力，是一个很强大的能力，线上谨慎使用，属于高危操作。

## 📑三、OGNL && 条件过滤

**顶流功能，可以使用 OGNL 解决很多复杂场景，其条件过滤属于绝佳。 适用于 watch、trace、stack、monitor等，有大量请求、 for 场景等。**

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/55686acfd0fc4ce18990387012c30be4~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1019&h=915&s=412228&e=png&b=f9f9f9)

#### 3.1 场景

条件过滤的适用场景实在太多，简单举例

解决过的场景|描述|
---|---|
想拦截特定参数值的方法入参出参，过滤其他参数请求|只有特定参数才会被拦截，否则跳过，不影响其他人使用|
拦截特定参数，这个参数方法调用耗时长|排查到了参数异常情况，特定账号数据量太大|
指定账号登录异常|通过监控指定 userId 的调用栈，排查问题|

#### 3.2 使用

条件判断形式：形如 params\[0\] == "orgIdxxx726" （OGNL）

场景|案例|描述|
---|---|---|
**watch 命令**：只监控特定组织的数据信息|watch com.xxx.controller.OrgServiceController getOrgInfo '{params,returnObj,throwExp}' 'params\[0\] == "orgIdxxx726"' -n 5 -x 3|通过“ '**params\[0\] == "orgIdxxx726**"'”命令，判断只有当参数为“orgIdxxx726” watch 才生效|
**trace 命令**：只有特定组织的数据比较消耗时间|trace com.xxx.controller.OrgServiceController getOrgInfo 'params\[0\] == "orgIdxxx726"' -n 5 --skipJDKMethod false|查询只要特定组织耗时长，忽略其他参数，精准定位|
**stack 命令**：查看调用栈，非常适合代理调用、AOP、Filter等|stack com.xxx.controller.OrgServiceController getOrgInfo 'params\[0\] == "orgIdxxx726"' -n 5|排查调用链路|

更多使用条件判断的案例如下

```java
-- 判空
trace com.xxx.controller.OrgServiceController getOrgInfo'params[0] != null'

-- 等于
trace *com.xxx.controller.OrgServiceController getOrgInfo 'params[0] == 1L'

-- 字符串不等于
trace com.xxx.controller.OrgServiceController getOrgInfo 'params[1] != "AA"'
```
 

OGNL 可以组合各种形式的条件判断。非常适合 watch、trace、stack 等场景。

#### 3.3 扩展

[更多灵活的用法](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Falibaba%2Farthas%2Fissues%2F11 "https://github.com/alibaba/arthas/issues/11")

[ognl | arthas](https://link.juejin.cn?target=https%3A%2F%2Farthas.aliyun.com%2Fdoc%2Fognl.html "https://arthas.aliyun.com/doc/ognl.html")

## 🔖 四、其他命令汇总

#### 4.1 logger

在写代码的时候，也可以刻意加 log.debug 级别的日志。日志级别一般为 info ,当需要排查问题的时候，可以修改日志级别为 debug。

解决过的场景|描述|
---|---|
自定义日志失效，排查日志的实现类由哪个包引入或者提供|排查间接引入的日志依赖包|
改变当前类的日志级别，查看日志|将 info 级别修改成 debug 级别|

```css
logger -n com.xxx.controller.OrgServiceControlle

通过sc 查看这个类的claasLoaderHash；

logger --name ROOT --level debug -c 4839ebd
```
 

#### 4.2 monitor

监控某个方法的调用次数。包括调用次数，平均RT、成功率等信息。在性能调优使用：

```arduino
monitor com.XXXX.handler.HandlerManager process  -n 10  --cycle 10
```
 

#### 4.3 thread

排查线程死锁，以及线程状态：

语句|详细|
---|---|
thread -b|排查死锁情况，注意，当 Arthas 不能加载的时候，还是继续使用原来的 top 命令那一套排查问题|
thread -n 3|查询当前最忙的 N 个线程|

#### 4.4 jad 命令、反编译

场景|描述|
---|---|
排查 jar 中的 class 文件加载是否符合预期。比如：突然发现某一台机器上的执行结果和其他的机器的结果不一致。|怀疑机器部署异常|
依赖包冲突，加载类不符合预期或者支合并的时候出错|检查运行的代码|

案例：springBoot 应用遇到 NoSuchMethodError 等问题，可以使用 Jad 反编译确认，看一下加载的类是否有问题。

#### 4.5 watch

watch 用来查看入参出参，配合 OGNL 条件过滤非常实用：

```arduino
watch com.xxl.mq.admin.service.IXxlMqMessageService pageList '{params,returnObj,throwExp}'  -n 5  -x 4
```
 

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9dc1a2af991c4f4282f3624ae0efddf9~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1447&h=443&s=57064&e=png&b=ffffff)

条件判断 #cost>200(单位ms) 表示只有当耗时大于200ms才输出：

```arduino
watch demo... primeFactors '{params, returnObj}' '#cost>200' -x 2
```
 

如果 x 设置超过 4 也就只展示4。使用 x=4 的情况比较常见，因为展开的信息最多。但需要注意线上数据量太大情况。

#### 4.6 tt （Time Tunnel）

tt 可以实现重做，实现方法调用；个人比较喜欢 vmtool，不过多介绍， tt 功能也十分强大。

#### 4.7 用得不多的命令

下面几个命令个人用得少，但很重要：

命令|描述|
---|---|
memory|查看内存信息|
options|全局开关，jvm 比较高级少用的命令|
sysprop / sysenv|当前 JVM 的系统属性，环境属性|
profiler|[火焰图](https://link.juejin.cn?target=https%3A%2F%2Farthas.aliyun.com%2Fdoc%2Fprofiler.html "https://arthas.aliyun.com/doc/profiler.html")|
dashboard|[实时数据面板](https://link.juejin.cn?target=https%3A%2F%2Farthas.aliyun.com%2Fdoc%2Fdashboard.html "https://arthas.aliyun.com/doc/dashboard.html")|

其他命令就不再赘述了📎。

#### 4.8 Arthas 插件功能

Arthas 插件生成的命令如下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1eeb9c5772914b17b466ac427bb3e9a2~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1745&h=1511&s=502786&e=png&b=fcfcfc)

> 注：图片来源于 [arthas 插件作者](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FWangJi92%2Farthas-idea-plugin%2Fblob%2Fmaster%2FREADME.md "https://github.com/WangJi92/arthas-idea-plugin/blob/master/README.md")，插件和文章都很好🌺

针对插件的热部署配置详见： [(Hot Swap) Redefine 热更新支持](https://link.juejin.cn?target=https%3A%2F%2Fwww.yuque.com%2Farthas-idea-plugin%2Fhelp%2Fpwxhb4 "https://www.yuque.com/arthas-idea-plugin/help/pwxhb4"))； 个人更推荐热部署用 ArthasHotSwap 插件。

## 📌五、一些限制 && 注意事项

*   执行trace / tt 等命令时，本质上是在 method 的前后插入代码，会影响原来 JVM 里面 JIT 编译生成的代码。可能执行并发高的函数有抖动；
*   只能有一个端口，只支持一个 arthas-server；
*   热部署有限制且不一定能成功，线上属于高危操作；
*   如果服务不能响应，可能 Arthas 不能启动使用，需要使用 Linux 相关命令排查问题。

## 📇六、好文推荐

1.  [官网文档](https://link.juejin.cn?target=https%3A%2F%2Farthas.aliyun.com%2F "https://arthas.aliyun.com/")
2.  [个人最推荐的学习资料： arthas idea plugin手册](https://link.juejin.cn?target=https%3A%2F%2Fwww.yuque.com%2Farthas-idea-plugin%2Fhelp%2Fpe6i45 "https://www.yuque.com/arthas-idea-plugin/help/pe6i45")

## 🧣七、最后的话

🖲要成为 Arthas 使用的好手，一定多多练习：**纸上得来终觉浅，绝知此事要躬行**。


>参考链接：[https://juejin.cn/post/7291931708920512527#heading-2](https://juejin.cn/post/7291931708920512527#heading-2)，整理：沉默王二

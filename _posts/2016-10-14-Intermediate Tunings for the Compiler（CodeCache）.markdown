---
layout:     post
title:      "Intermediate Tunings for the Compiler（CodeCache）"
subtitle:   "Intermediate Tunings for the Compiler（CodeCache）"
date:       2016-10-14
author:     "BilboDai"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
    - Java
    - Jvm
---

概述
---
通常情况下对编辑器编译器进行调优实际上就是在选择最合适的JVM。在安装的时候可以简单的通过(-client, -server or -XX:+TieredCompilation) 来切换编译器。但无论是对长时间运行的应用还是很短时间执行完的应用来说`Tiered compilation`总是最好的选择。

调整Code Cache
---
Code Cache是是一块固定大小的空间，它存放着是`assembly-language instructions`的集合。一旦这个空间被塞满了，JVM就不会再编译任何代码。也就是说即使某些热点（Hot spots）方法，循环也不会得到编译优化后执行，而是使用解释执行这种低效率的方式执行。当Code Cache空间满的时候，JVM通常会打印出警告信息：

```
Java HotSpot(TM) 64-Bit Server VM warning: CodeCache is full.
             Compiler has been disabled.
    Java HotSpot(TM) 64-Bit Server VM warning: Try increasing the
             code cache size using -XX:ReservedCodeCacheSize=

```

这种警告信息不容易发现，并且当开启`Tiered compilation`之后有些版本的java7不能正确的打印。

| JVM Type        | 默认CodeCache的大小   |
| --------   | -----:  |
| 32bit client,java8     | 32M |
| 32bit server with tired compilation,java8        |   240M   |
| 64bit server with tired compilation,java8        |    240M    |
| 32bit client,java7        |    32M    |
| 32bit server,java7        |    32M    |
| 64bit server,java7        |    48M    |
| 64bit server with tired compilation,java7        |    96M    |

Java7如果是用`Tiered compilation`那么默认的Code Cache空间往往是不够的。使用`-XX:ReservedCodeCacheSize=N `来声明Code Cache可以使用的最大空间，使用`-XX:InitialCodeCacheSize=N`来声明初始空间大小。
CodeCache的空间大小可以通过jconsole来监测到。

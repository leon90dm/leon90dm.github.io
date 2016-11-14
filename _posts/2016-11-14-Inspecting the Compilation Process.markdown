---
layout:     post
title:      "Inspecting the Compilation Process"
subtitle:   "Inspecting the Compilation Process"
date:       2016-11-14
author:     "BilboDai"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - Java
    - Jvm
---

通过打开`-XX:+PrintCompilation`（默认是false）开关，每一次方法或者循环被编译，JVM都会打印一条编译日志，日志的格式随着不同的java版本而不同，直到java7后，日志格式是：

```
timestamp compilation_id attributes (tiered_level) method_name size deopt
```

- `timestamp`是编译完成时相对于JVM启动的时间（0）偏移
- `compilation_id`是一个内部的编译任务ID，一般情况下是依次递增的，但当编译线程多了之后可能会乱序了，因为有的先启动编译的时间会比后开始编译的较长
- `attributes`揭示被编译的代码的状态，

```
%: The compilation is OSR.
s: The method is synchronized.
!: The method has an exception handler.
b: Compilation occurred in blocking mode.
n: Compilation occurred for a wrapper to a native method.
```

如果编译状态由多个`attribute`，会以**空格**分割。`b`阻塞属性在现有的java版本中不会出现，它表示编译不是在后台异步进行的。

- `tiered_level` 如果程序不是以tiered compilation运行的，那么这个字段将是空的。否则它会是一个`Tiered Compilation Levels`。
- `method_name` 被编译的方法名，一般是`ClassName::method`格式。
- `size` 单位是（bytes），表示被编译代码的大小，这个大小是指被编译的bytecode的大小而不是编译之后的。
- `deopt` 表示发生了**反优化**行为。主要是**made not entrant** or **made zombie** 

编译日志也有可能类似：

```
timestamp compile_id COMPILE SKIPPED: reason
```

这个日志标识编译没能成功，原因可能是：

- Code cache filled 需要使用ReservedCodeCache提高codecache大小
- Concurrent classloading class在编译期间被修改了。JVM会等会儿再做编译。

下面是一个编译日志的样例：

```
 28015  850
 28179  905  s
 28226   25 %
 28244  935
 29929  939
106805 1568   !
net.sdo.StockPrice::getClosingPrice (5 bytes)
net.sdo.StockPriceHistoryImpl::process (248 bytes)
net.sdo.StockPriceHistoryImpl::<init> @ 48 (156 bytes)
net.sdo.MockStockPriceEntityManagerFactory$\
    MockStockPriceEntityManager::find (507 bytes)
net.sdo.StockPriceHistoryImpl::<init> (156 bytes)
net.sdo.StockServlet::processRequest (197 bytes)
```

Inspecting Compilation with jstat
---
如果没有打开`-XX:+PrintCompilation`开关，还可以通过`jstat`来提供编译的信息。`jstat`提供两个选项：

- -compiler 

```
% jstat -compiler 5003
Compiled Failed Invalid Time FailedType FailedMethod
206 0 0 1.97 0
```

- -printcompilation

```
% jstat -printcompilation 5003 1000 Compiled Size Type Method
           207     64    1 java/lang/CharacterDataLatin1 toUpperCase
           208      5    1 java/math/BigDecimal$StringBuilderHelper getCharArray
```

但jstat得到的信息不是特别的全面。

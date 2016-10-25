---
layout:     post
title:      "JIT-Compilation-Threshold"
subtitle:   "Compiler Threshold Turning"
date:       2016-10-25
author:     "BilboDai"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - Java
    - Jvm
---

Compilation Thresholds
---
编译在JVM中受两个计数器的影响

- 方法调用的次数
- 在一个循环中，循环的次数，通俗理解也就是调用continue或者运行到了循环结束位置的次数

JVM会检查这个两个计数器是否达到可以编译的资格。如果已经达到了，那么就会把当前的函数放入编译队列等待编译线程来执行编译。
通常情况下如果编译好了，在再次执行该方法的时候，JVM会将编译好的性能更高的方法替换原来的。但是如果一个方法包含一个永真循环或者一个要循环很久的循环该怎么处理呢？JVM使用**on-stack replacement（OSR）**技术，JVM会把编译好的循环的代码替换正在执行栈里的代码，等到循环下一次迭代的时候就会执行新的编译好的代码。

标准编译阈值可以通过**-XX:CompileThreshold=N**来设置，C1编译器默认值是**1500**，C2编译器默认值是**10,000**，通过调整这个值的大小可以控制编译是否提前或者推后。前面讲到有两个计数器（方法调用次数和循环迭代次数）而这里只有一个设置的阈值，其实实际运行时是取的**方法调用次数和循环迭代次数**的和和这个阈值进行比较。

Changing OSR Compilation
---
绝大多数情况下不需要调整OSR的阈值，OSR通常发生在**microbenchmarks**中，而普通应用程序中是很少会触发到OSR的。OSR的触发条件公式：

```
OSR trigger = (CompileThreshold *
              ((OnStackReplacePercentage - InterpreterProfilePercentage)/100))
```

所有的编译器（C1,C2) `-XX:InterpreterProfilePercentage=N`默认的值都是33，C1中`-XX:OnStackReplacePercentage=N`的默认值是933，所以触发OSR的阈值是13500。而C2中是140，触发OSR的阈值是10700。

实际使用中是推荐修改适当降低CompileThreshold阈值的，比如循环迭代8,000次之后。但是也有风险，风险就是编译出来的代码的没有比JVM�通过观察更多次之后收集到更多信息编译出来的代码性能更高。降低阈值有两个原因：

-  可以节约程序启动的时间
-  可以编译某些可能永远无法编译的方法（每当JVM到达一个safepoint，计数器值就会减少，也就是说计数器的值只是方法或者循环最近是否是热点的反应）

这也是为什么`tiered compilation`会比C2稍稍的快些的原因。
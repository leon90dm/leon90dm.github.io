---
layout:     post
title:      "Something need to know about CMS"
subtitle:   "CMS那点东东"
date:       2016-11-16
author:     "BilboDai"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - Java
---

所有的GC算法除了`serial collector`都会使用多线程，线程的数量可以通过`-XX:ParallelGCThreads=N`来显示控制。默认为：

> ParallelGCThreads = 8 + ((N - 8) * 5 / 8) N为CPU的核数

对于关注吞吐量的GC，可以通过`-XX:MaxGCPauseMillis=N` 和 `-XX:GCTimeRatio=N`来控制程序和GC的执行时间占比。其中

> ThroughputGoal = 1 - 1/(1 + GCTimeRatio)

比如，要想达到ThroughputGoal=95%，那么代入即可得到`GCTimeRatio=19`.

CMS Old GC分以下好几个阶段

CMS-initial-mark->CMS-concurrent-mark->CMS-concurrent-preclean->**CMS-concurrent-abortable-preclean**->CMS-remark->CMS-concurrent-sweep->CMS-concurrent-reset

其中有个比较有趣的阶段`CMS-concurrent-abortable-preclean`，看看它的日志输出

```
    90.892: [CMS-concurrent-abortable-preclean-start]
    92.392: [GC 92.393: [ParNew: 629120K->69888K(629120K), 0.1289040 secs]
                    1331374K->803967K(2027264K), 0.1290200 secs]
                    [Times: user=0.44 sys=0.01, real=0.12 secs]
    94.473: [CMS-concurrent-abortable-preclean: 3.451/3.581 secs]
                    [Times: user=5.03 sys=0.03, real=3.58 secs]
    94.474: [GC[YG occupancy: 466937 K (629120 K)]
            94.474: [Rescan (parallel) , 0.1850000 secs]
            94.659: [weak refs processing, 0.0000370 secs]
            94.659: [scrub string table, 0.0011530 secs]
                    [1 CMS-remark: 734079K(1398144K)]
                    1201017K(2027264K), 0.1863430 secs]
            [Times: user=0.60 sys=0.01, real=0.18 secs]
```

由于CMS在remark前的阶段和应用程序是并发执行的因此随时可能会有Young GC，而remark阶段和Young GC是会造成STW的。所以很可能会造成remark阶段STW紧接着又开始Young GC的STW造成`back-to-back pause`所以这是一个巧妙的阶段，这个阶段它会等至少一个Young GC然后通过计算GC的时间间隔估算到新生代快占满50%的时候进行remark阶段。

什么是`concurrent mode failure` CMS在进行老年代GC的时候也会失败，主要有两种情况

- (concurrent mode failure) ：当老年代GC跟不上时就会出现老年代没有足够的内存空间给新生代晋升的对象存放。
- (promotion failed) 老年代有足够的空间给晋升的对象，但是由于太过碎片化，无法给出一个大的空间装载这个大的对象。

这种时候，就会发生Full GC。新生代包括Survive区都被清空，老年代被清理并压缩。
此外还有一种情况会发生Full GC：当permgen被填满的时候，java8以后没有了permgen，但是每当metaspace需要调整大小的时候都需要一个Full GC。

```
279.803: [Full GC 279.803:
                    [CMS: 88569K->68870K(1398144K), 0.6714090 secs]
                    558070K->68870K(2027264K),
                    [CMS Perm : 81919K->77654K(81920K)],
                    0.6716570 secs]
```

因此尽量避免`concurrent mode failures`是十分关键的，具体策略如下：

- 如果可以，适当增大堆空间大小
- 调整`CMSInitiatingOccupancyFraction`参数使得并发后台清理线程能尽快启动
- 如果可以，增加后台清理线程的数量

Java7中CMS默认是只会等到permgen满了之后CMS执行一个Full GC来清理。但通过打开`-XX:+CMSPermGenSweepingEnabled`这个开关可以让CMS像清理老年代一样清理permgen。同样也可以设置-XX:CMSInitiatingPermOccupancyFraction=N，默认是80%让CMS在permgen占用达到多少时进行清理。最后特别需要注意，还需要打开`-XX:+CMSClassUnloadingEnabled`开关，不然CMS是不会清理class metadata的（java8中这个开关默认是打开的）
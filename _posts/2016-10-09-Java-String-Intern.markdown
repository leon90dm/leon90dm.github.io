---
layout:     post
title:      "String.intern 和 string pooling"
subtitle:   "介绍Java中字符串常量池相关的内容"
date:       2016-10-09
author:     "BilboDai"
header-img: "img/post-bg-alitrip.jpg"
catalog: true
tags:
    - Java
---

String pooling
---
曾经Java6之前是不允许使用`String.intern()`，因为很可能使用不当就会造成`OutOfMemoryException`.然后Java7之后这个问题得到了官方的解决。

Java6之前，所有`interned`的字符串都被存储在`PermGen`（永久区），永久带必须设置一个固定的大小，并且一旦虚拟机启动，则大小不可更改。一般情况下，永久区大小设置`-XX:MaxPermSize=N `在32M到96M之间。

Java7之后，这个情况变了，`interned`的字符串有被重新分配堆上。准确的说是一个叫做`String Pool`的地方。Java6及以前这个`Pool`在永久区，而Java7之后被安放在了堆当中。

`String Pool`是用`HashMap`来实现的,每个槽是一个包含了相同`Hash Code`的字符串列表，默认槽的初始大小为1009，在java6及以前这个大小是不可以调整的，java7之后这个参数可以通过`-XX:StringTableSize=N`来调整，注意`N`最好是一个**素数**，以便获得更好的性能。

**提示：**如果你要使用`String.intern()`方法的话，你应该设置一个更高的`-XX:StringTableSize`值（相比默认的1009），否则性能会急剧下降。

那么我们应该构建一个手工的String Pool么？

```java
private static final WeakHashMap<String, WeakReference<String>> s_manualCache =
    new WeakHashMap<String, WeakReference<String>>( 100000 );
 
private static String manualIntern( final String str )
{
    final WeakReference<String> cached = s_manualCache.get( str );
    if ( cached != null )
    {
        final String value = cached.get();
        if ( value != null )
            return value;
    }
    s_manualCache.put( str, new WeakReference<String>( str ) );
    return str;
}
```

答案是没必要的，因为从测试的性能来看，相差不大。

从Java7u40之后，`String Pool`大小改为了60013，这个值能够在出现冲突前包含大概30000个不同的字符串。

总结
---
- Java6及以前最好不要使用`String.intern()`
- Java7以后的实现是将`String Pool`放在了堆上，这样你可以通过`-XX:StringTableSize`适当调整它的大小。
- 使用`-XX:+PrintStringTableStatistics`来打印`String.intern()`相关的信息
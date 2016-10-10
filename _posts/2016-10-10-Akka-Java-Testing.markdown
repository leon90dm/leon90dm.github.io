---
layout:     post
title:      "Akka-Java-Testing"
subtitle:   "使用akka-testkit进行单元测试"
date:       2016-10-10
author:     "BilboDai"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
    - Java
    - Akka
---

Akka包含了一个用于支持不同程度测试的包`akka-testkit`.

依赖
---

```xml
<dependencies>
        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-actor_2.11</artifactId>
            <version>2.4.7</version>
        </dependency>
        <dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-testkit_2.11</artifactId>
            <version>2.4.7</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

使用`TestActorRef`进行同步的单元测试
---
测试一个Actor业务逻辑可以划分为两个部分
- 原子操作必须在隔离情况下正常工作
- 必须按照接收消息的顺序正确处理

通常情况下`ActorRef`隐藏了`Actor`实例内部的数据，唯一能和它打交道的就是通过`mailbox`。但这个是对测试不友好的。而`TestActorRef`就能通过获取底层的`actor`实例或者直接调用`receive`方法。看下面这个代码：

```java
static class MyActor extends UntypedActor {
  public void onReceive(Object o) throws Exception {
    if (o.equals("say42")) {
      getSender().tell(42, getSelf());
    } else if (o instanceof Exception) {
      throw (Exception) o;
    }
  }
  public boolean testMe() { return true; }
}
 
@Test
public void demonstrateTestActorRef() {
  final Props props = Props.create(MyActor.class);
  final TestActorRef<MyActor> ref = TestActorRef.create(system, props, "testA");
  final MyActor actor = ref.underlyingActor();
  assertTrue(actor.testMe());
}
```

我们是没办法通过`ActorRef`直接去调用`MyActor`的`testMe()`方法的。但通过`TestActorRef.underlyingActor()`可以直接获得其实例。
当然也可以直接调用`receive()`方法，这样`MyActor`的`onReceive()`会被自动回调。

```java
final Props props = Props.create(MyActor.class);
final TestActorRef<MyActor> ref = TestActorRef.create(system, props, "myActor");
try {
  //receive会直接抛出onReceive抛出的异常
  ref.receive(new Exception("expected"));
  fail("expected an exception to be thrown");
} catch (Exception e) {
  assertEquals("expected", e.getMessage());
}
```

使用JavaTestKit进行异步集成测试
---
当很确定每个Actor内部的业务逻辑是正确的时候，就需要在模拟它应该运行的环境中进行测试。

```java
public static class SomeActor extends UntypedActor {
        ActorRef target = null;

        public void onReceive(Object msg) {

            if (msg.equals("hello")) {
                getSender().tell("world", getSelf());
                if (target != null)
                    target.forward(msg, getContext());

            } else if (msg instanceof ActorRef) {
                target = (ActorRef) msg;
                getSender().tell("done", getSelf());
            }
        }
    }
    
@Test
    public void testMyIt() throws Exception {
        new JavaTestKit(system){
            {
                final Props props = Props.create(SomeActor.class);
                final ActorRef subject = system.actorOf(props);
                subject.tell(getRef(), getRef());
                expectMsgEquals(duration("1 second"), "done");
            }
        };
    }
```

`JavaTestKit`内部有一个探针`TestProbe p`，这个探针内部有一个`testActor`可以通过的`JavaTestKit.getRef()`获得它的引用.
再来看看那个测试代码，有一个很怪的模式：
`new JavaTestKit(system){ {...} }`所有的测试代码都是放在`{...}`中的。这是因为所有类似`new IgnoreMsg() {};`内部会通过`JavaTestKit.this`来获取其他方法来完成测试验证。

Akka Built-In Assertions
---

```java
//在给定的时间必须收到"hello",不然抛出AssertionFailure
final String hello = expectMsgEquals(duration,"hello");
//在给定的时间里收到"hello"或者"world"即可
final Object   any = expectMsgAnyOf(duration,"hello", "world");
//在给定的时间里必须都收到"hello"和"world"
final Object[] all = expectMsgAllOf(duration,"hello", "world");
//在给定的时间收到整型的消息
final int i        = expectMsgClass(duration,Integer.class);
//在给定的时间里收到整型或者长整型
final Number j     = expectMsgAnyClassOf(duration,Integer.class, Long.class);
//在给定时间里收不到消息，如果收到则抛出异常
expectNoMsg(duration);
//在给定的时间里，收到2次消息
final Object[] two = receiveN(duration,2);
```

### ExpectMsg<T>
ExpectMsg需要重写match方法，它的作用很单一，就是收到match之后的消息之后通过get就会返回。

```
new JavaTestKit(system) {
{
  getRef().tell(42, ActorRef.noSender());
  final String out = new ExpectMsg<String>("match hint") {
      protected String match(Object in) {
        if (in instanceof Integer) {
          return "match";
        } else {
          throw noMatch();
        }
      }
    }.get(); // this extracts the received message
  assertEquals("match", out);
}
};
```

### ReceiveWhile<T>
这个也需要重写match方法，只是它会连续接收match通过的消息，知道出现不符合的消息才终止，最后get返回的是一个接收到消息的数组。

```java
new JavaTestKit(system) {
{
  getRef().tell(42, ActorRef.noSender());
  getRef().tell(43, ActorRef.noSender());
  getRef().tell("hello", ActorRef.noSender());
  final String[] out =
    new ReceiveWhile<String>(String.class, duration("1 second")) {
      // do not put code outside this method, will run afterwards
      protected String match(Object in) {
        if (in instanceof Integer) {
          return in.toString();
        } else {
          throw noMatch();
        }
      }
    }.get(); // this extracts the received messages
  assertArrayEquals(new String[] {"42", "43"}, out);
  expectMsgEquals("hello");
}
};
```

### AwaitCond
这个是一个条件等，需要重写cond方法，等待固定的间隔去调用cond()如果返回true，则跳过等待，否则继续等待直到超过总等待时间。

```java
new JavaTestKit(system) {
{
  getRef().tell(42, ActorRef.noSender());
  new AwaitCond(
        duration("1 second"),  // maximum wait time
        duration("100 millis") // interval at which to check the condition
        ) {
    // do not put code outside this method, will run afterwards
    protected boolean cond() {
      // typically used to wait for something to start up
      return msgAvailable();
    }
  };
}
};
```

### AwaitAssert
这个是一个条件检查，需要重写check方法，等待固定的间隔去调用check()直到超过总时间。

```java
new JavaTestKit(system) {
{
  getRef().tell(42, ActorRef.noSender());
  new AwaitAssert(
        duration("1 second"),  // maximum wait time
        duration("100 millis") // interval at which to check the condition
        ) {
    // do not put code outside this method, will run afterwards
    protected void check() {
      assertEquals(msgAvailable(), true);
    }
  };
}
};
```

### IgnoreMsg
这个是一个过滤器，需要重写ignore方法，它会过滤所有ignore返回true的消息。

```java
new JavaTestKit(system) {
{
  // 忽略所有的字符串类型的消息
  new IgnoreMsg() {
    protected boolean ignore(Object msg) {
      return msg instanceof String;
    }
  };
  getRef().tell("hello", ActorRef.noSender());
  getRef().tell(42, ActorRef.noSender());
  expectMsgEquals(42);
  // remove message filter
  ignoreNoMsg();
  getRef().tell("hello", ActorRef.noSender());
  expectMsgEquals("hello");
}
};
```

### Expecting Log Messages
将akka的loggers配置加上`akka.testkit.TestEventListener`，即可利用`EventFilter`来进行日志的断言。
首先在Classpath下的配置文件 `application.conf` 中加上`akka.loggers = [akka.testkit.TestEventListener]`

```java
new JavaTestKit(system) {
{
  assertEquals("TestKitDocTest", system.name());
  final ActorRef victim = system.actorOf(Props.empty(), "victim");
 
  final int result = new EventFilter<Integer>(ActorKilledException.class) {
    protected Integer run() {
      victim.tell(Kill.getInstance(), ActorRef.noSender());
      return 42;
    }
  }.from("akka://TestKitDocTest/user/victim").occurrences(1).exec();
 
  assertEquals(42, result);
}
};
```

### Timing Assertions
Within非常的方便，构造方法传(min,max)意思要求在min-max这段时间内，重写的run方法必须执行完，否则就抛出异常。

```java
new JavaTestKit(system) {
{
  getRef().tell(42, ActorRef.noSender());
  new Within(Duration.Zero(), Duration.create(1, "second")) {
    // do not put code outside this method, will run afterwards
    public void run() {
      assertEquals((Integer) 42, expectMsgClass(Integer.class));
    }
  };
}
};
```

### 时间放大
JavaTestKit的dilated方法可以把一个Duration进行放大，放的原理是：获取`application.conf`中`akka.test.timefactor = 2`的值乘以原本的Duration。

```java
public Duration dilated(Duration d) {
    return d.mul(TestKitExtension.get(getSystem()).TestTimeFactor());
  }
```

### Watching Other Actors from Probes
使用JavaTestKit可以watch一个Actor然后即可收到Terminated消息

```java
new JavaTestKit(system) {
{
  final JavaTestKit probe = new JavaTestKit(system);
  probe.watch(target);
  target.tell(PoisonPill.getInstance(), ActorRef.noSender());
  final Terminated msg = probe.expectMsgClass(Terminated.class);
  assertEquals(msg.getActor(), target);
}
};
```

### Replying to Messages Received by Probes
JavaTestKit提供reply方法可以获取到最后一次发送给probe的actor，并且给它发消息。

```java
new JavaTestKit(system) {
{
  final JavaTestKit probe = new JavaTestKit(system);
  probe.getRef().tell("hello", getRef());
  probe.expectMsgEquals("hello");
  probe.reply("world");
  expectMsgEquals("world");
  assertEquals(probe.getRef(), getLastSender());
}
};
```

### Auto-Pilot
JavaTestKit可以设置一个AutoPilot，当probe收到消息之后AutoPilot的run方法就会被回调，完成之后需要返回下一次接收消息处理的AutoPilot，如果返回`noAutoPilot()`，则表示probe不在接收消息。

```java
new JavaTestKit(system) {
{
  final JavaTestKit probe = new JavaTestKit(system);
  // install auto-pilot
  probe.setAutoPilot(new TestActor.AutoPilot() {
    public AutoPilot run(ActorRef sender, Object msg) {
      sender.tell(msg, ActorRef.noSender());
      return noAutoPilot();
    }
  });
  // first one is replied to directly ...
  probe.getRef().tell("hello", getRef());
  expectMsgEquals("hello");
  // ... but then the auto-pilot switched itself off
  probe.getRef().tell("world", getRef());
  expectNoMsg();
}
};
```

Reference
---
[http://doc.akka.io/docs/akka/2.4.11/java/testing.html](http://doc.akka.io/docs/akka/2.4.11/java/testing.html)
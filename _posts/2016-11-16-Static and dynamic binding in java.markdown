---
layout:     post
title:      "Static and dynamic binding in java"
subtitle:   "Java中的静态和动态绑定"
date:       2016-11-16
author:     "BilboDai"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - Java
---

什么是实例，什么是引用
---

```
class Human{
....
}
class Boy extends Human{
   public static void main( String args[]) {
       /*这个语句创建了一个Boy实例，并把Boy实例的引用给了obj1*/  
       Boy obj1 = new Boy();

       /* 因为Boy继承自Human，因此父类类型的引用同样可以指向子类实例*/
       Human obj2 = new Boy();
   }
}
```

Java拥有两种不同类型的类型绑定：**Static and Dynamic Binding**

Static Binding or Early Binding
---
静态绑定是在编译期间就被解析的。所有`static`，`private`和`final`的方法总是在编译期间被绑定。为什么？因为编译器知道这些方法不会被重写，方法总是被本地类的实例访问，因此编译器很容易辨别这种类型的类，所以这些方法总是静态绑定的。例如：

```
class Human{
....
}
class Boy extends Human{
   public void walk(){
      System.out.println("Boy walks");
   }
   public static void main( String args[]) {
      Boy obj1 = new Boy();
      obj1.walk();
   }
}
```

Dynamic Binding or Late Binding
---
当子类重写了父类的方法后，编译器就会比较迷糊了，搞不清楚究竟应该是调用父类的还是子类的方法，毕竟方法签名都是一样的。

```
class Human{
   public void walk()
   {
       System.out.println("Human walks");
   }
}
class Boy extends Human{
   public void walk(){
       System.out.println("Boy walks");
   }
   public static void main( String args[]) {
       //Reference is of parent class
       Human myobj = new Boy();
       myobj.walk();
   }
}
```

由于Human和Boy类都有`void walk()`方法，由于我们使用的是父类Human类型的引用指向Boy实例，然后调用的`void walk()`，因此编译器搞不清楚那个walk方法应该被调用。这种使用就需要使用动态绑定，需要在程序运行时确定`myobj`的类型。

Method Overloading in Java 
---
方法的重载主要依赖以下几个因素：

1. 参数的个数[f(int) =/= f(int,int)]
2. 参数的类型[f(int) =/= f(float)]
3. 参数的顺序[f(int,float) =/= f(float,int)]

```
public class RuntimeBindingTest {

    public static class Foo{}

    public static class Bar extends Foo{}

    public void f(Object f1) {
        System.out.println("f1");
    }

    public void f(Foo f2) {
        System.out.println("f2");
    }

    public void f(Bar f3) {
        System.out.println("f3");
    }

    public static void main(String[] args) {
        Bar bar = new Bar();
        Foo foo = new Bar();
        Object obj = new Bar();

        RuntimeBindingTest test = new RuntimeBindingTest();
        test.f(bar);
        test.f(foo);
        test.f(obj);
        test.f((Bar)obj);
    }
}

Output:
f3
f2
f1
f3
```

通过`javap -v`查看字节码可以看出方法的调用是编译期间绑定的。
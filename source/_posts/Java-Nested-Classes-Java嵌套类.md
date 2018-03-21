---
title: Java Nested Classes - Java嵌套类
date: 2018-03-21 16:07:03
tags:
    - Java
    - Nested Classes
categories:
---

## Nested Classes - 嵌套类
Java 编程语言允许在一个类中定义另一个类，这样的类被称为嵌套类。嵌套类分为两种：static（静态） 和 non-static (非静态)。声明为 <code>static</code> 的嵌套类称为静态嵌套类（static nested classes），非静态嵌套类称为内部类（inner classes）。

```java
static OuterClass {
    ...
    static class StaticNestedClass {
        ...
    }
    class InnerClass {
        ...
    }
}
```

<!-- more -->

## 静态嵌套类和内部类的区别
内部类可以访问外部类的成员变量和方法，即使这些变量和方法被声明为<code>private</code>，静态嵌套类不可以访问外部类的其它成员。作为外部类的一员，嵌套类可以声明为<code>private</code>、 <code>public</code>、<code>protected</code> 或者 <code>package private</code>，回想一下外部类只能声明为 <code>public</code> 或者 <code>package private</code>。

### Static Nested Classes - 静态嵌套类
一个静态嵌套类与它的外部类的实例成员(和其他类)就像任何其他顶级类，它的行为跟一个顶级类一样，只是为了包装方便而嵌套在另一个顶级类中。换句话理解就是：静态嵌套类只是为了方便而将类隐藏在另一个顶级类中，和外部类之间不存在本质上的“内外”关系。

静态嵌套类的调用方式如下：
```java
// Access method
OuterClass.StaticNestedClass

// Sample: create a new static nested class object
OuterClass.StaticNestedClass nestedObject = new OuterClass.StaticNestedClass();
```

### Inner Classes - 内部类
与实例方法和变量一样，一个内部类与包含它的外部类实例相关联，可以直接访问外部类实例的方法和字段。此外一个内部类因为与一个实例相关联，所以内部类本身不能定义任何静态成员。

一个内部类的实例只能存在于一个外部类的实例中，并且可以直接访问外部类实例的方法和字段。想要实例化一个内部类，必须先实例化外部类，然后再通过下面的方式创建内部类的实例：
```java
OuterClass outerClass = new OuterClass();
OuterClass.InnerClass innerClass = outerClass.new InnerClass();
```

有两个特殊类型的内部类：[本地类 local classes](https://docs.oracle.com/javase/tutorial/java/javaOO/localclasses.html) 和 [匿名类 anonymous classes](https://docs.oracle.com/javase/tutorial/java/javaOO/localclasses.html)。

#### Local Classes - 本地类
目前在实际开发中还没遇到这种情况，有需要的请自行阅读[官方文档](https://docs.oracle.com/javase/tutorial/java/javaOO/localclasses.html)。

#### Anonymous Classes - 匿名类
匿名类可以让代码更简洁，匿名类允许在同一时间声明和实例化一个类。除了没有一个名字之外，匿名类就像本地类，如果你只需要使用本地类一次，那么推荐你使用匿名类。

本地类是类声明，而匿名类是一个表达式，这意味着你在表达式里面定义了一个类。下面的 HelloWorldAnonymousClasses 示例在初始化本地变量frenchGreeting spanishGreeting 使用了匿名类，而在初始化变量 englishGreeting 使用了本地类：

```java
public class HelloWorldAnonymousClasses {

    interface HelloWorld {
        public void greet();
        public void greetSomeone(String someone);
    }

    public void sayHello() {

        class EnglishGreeting implements HelloWorld {
            String name = "world";
            public void greet() {
                greetSomeone("world");
            }
            public void greetSomeone(String someone) {
                name = someone;
                System.out.println("Hello " + name);
            }
        }

        HelloWorld englishGreeting = new EnglishGreeting();

        HelloWorld frenchGreeting = new HelloWorld() {
            String name = "tout le monde";
            public void greet() {
                greetSomeone("tout le monde");
            }
            public void greetSomeone(String someone) {
                name = someone;
                System.out.println("Salut " + name);
            }
        };

        HelloWorld spanishGreeting = new HelloWorld() {
            String name = "mundo";
            public void greet() {
                greetSomeone("mundo");
            }
            public void greetSomeone(String someone) {
                name = someone;
                System.out.println("Hola, " + name);
            }
        };
        englishGreeting.greet();
        frenchGreeting.greetSomeone("Fred");
        spanishGreeting.greet();
    }

    public static void main(String... args) {
        HelloWorldAnonymousClasses myApp =
            new HelloWorldAnonymousClasses();
        myApp.sayHello();
    }            
}
```

如前所述，一个匿名类是一个表达式。一个匿名类表达式的语法就像一个构造函数的调用，除了在代码中有一块类定义的区域。因为一个匿名类定义是一个表达式,所以它必须是声明的一部分。在这个例子中,匿名类表达式声明实例化frenchGreeting对象的一部分。(这就解释了为什么有分号后关闭括号)

匿名类表达式包含以下:
-

。

## 参考文章
[Nested Classes - Oracle](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html)

---
title: Java Nested Classes - Java嵌套类
date: 2018-03-21 16:07:03
tags:
    - Java
    - Nested Classes
categories:
---

本文节选了 [Nested Classes](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html) 官方文档的部分内容进行翻译，通过翻译过程系统地学习 Java 嵌套类。

<!-- more -->

## Nested Classes - 嵌套类
Java 编程语言允许在一个类中定义另一个类，这样的类被称为嵌套类：
```java
class OuterClass {
    ...
    class NestedClass {
        ...
    }
}
```

嵌套类分为两种：static（静态） 和 non-static (非静态)。声明为 <code>static</code> 的嵌套类称为静态嵌套类（static nested classes），非静态嵌套类称为内部类（inner classes）。
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

嵌套类是封闭类的一个成员。非静态嵌套类可以访问封闭类的其它成员，即使它们被声明为 <code>private</code>。静态嵌套类无法访问封闭类的其它成员。作为外部类的一个成员，嵌套类可以声明为<code>private</code>、 <code>public</code>、<code>protected</code> 或 <code>package private</code>，回想一下外部类只能声明为 <code>public</code> 或者 <code>package private</code>。

### 为何使用嵌套类
有以下几条令人信服的理由使用嵌套类：
- 它是一种将只在一个地方使用的类进行逻辑分组的方式
- 增强了封装性
- 可以使代码具有更好的可读性和可维护性

### Static Nested Classes - 静态嵌套类
与类方法和变量一样，静态嵌套类与外部类相关联。与静态方法一样，静态嵌套类不能直接引用封闭类中定义的实例变量或方法，只能通过一个对象引用来使用它们。静态嵌套类与它外部类实例成员或方法(和其它类)之间的交互就像任何其它顶级类一样。实际上，为了方便打包，静态嵌套类在行为上是嵌套在另一个顶级类中的顶级类。通俗的理解就是：静态嵌套类只是为了打包方便而将一个类隐藏在另一个类中，和外部类之间不存在本质上的“内外”关系。

静态嵌套类的调用方式如下：
```java
// 访问方式
OuterClass.StaticNestedClass

// 实例：创建一个新的静态嵌套类对象
OuterClass.StaticNestedClass nestedObject = new OuterClass.StaticNestedClass();
```

### Inner Classes - 内部类
与实例方法和变量一样，一个内部类与包含它的外部类实例相关联，可以直接访问外部类实例的方法和字段。此外，因为内部类与实例相关联，所以内部类本身不能定义任何静态成员。

一个内部类的实例只能存在于一个外部类的实例中，并且可以直接访问外部类实例的方法和字段。想要实例化一个内部类，必须先实例化外部类，然后再通过外部类对象创建内部类的实例，语法如下：
```java
OuterClass outerClass = new OuterClass();
OuterClass.InnerClass innerClass = outerClass.new InnerClass();
```

有两种特殊的内部类：[局部类 local classes](https://docs.oracle.com/javase/tutorial/java/javaOO/localclasses.html) 和 [匿名类 anonymous classes](https://docs.oracle.com/javase/tutorial/java/javaOO/localclasses.html)。

### 遮盖
如果特定作用域（如内部类或方法定义）中的类型声明（例如成员变量或参数名称）与封闭作用域中的另一个声明具有相同的名称，则该声明会隐藏封闭范围中的声明。你无法单独通过名称来引用被隐藏的声明。下面的示例 ShadowTest 演示了这一点：
```java
public class ShadowTest {

    public int x = 0;

    class FirstLevel {

        public int x = 1;

        void methodInFirstLevel(int x) {
            System.out.println("x = " + x);
            System.out.println("this.x = " + this.x);
            System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);
        }
    }

    public static void main(String... args) {
        ShadowTest st = new ShadowTest();
        ShadowTest.FirstLevel fl = st.new FirstLevel();
        fl.methodInFirstLevel(23);
    }
}
```

示例输出如下：
```
x = 23
this.x = 1
ShadowTest.this.x = 0
```

这个例子定义了三个名为<code>x</code>的变量：ShadowTest 类的成员变量，内部类 FirstLevel 的成员变量，以及 methodInFirstLevel 方法的参数。methodInFirstLevel 方法定义的参数<code>x</code>遮盖了内部类 FirstLevel 的变量<code>x</code>。因此在方法 methodInFirstLevel 中使用变量<code>x</code>时，它指向的是方法参数。要引用内部类 FirstLevel 的成员变量<code>x</code>，请使用关键字 <code>this</code> 来表示封闭范围：
```java
System.out.println("this.x = " + this.x);
```

通过它们所属的类名引用包含较大范围的成员变量。例如下面的语句在 methodInFirstLevel 方法中访问了类 ShadowTest 的成员变量<code>x</code>：
```java
System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);
```

### 序列化
序列化内部类，包括局部类和匿名类是强烈不建议的。当 Java 编译器编译某些结构（如内部类）时，它会创建合成结构，这些是在源代码中没有相应构造的类，方法，字段和其他构造。合成结构使 Java 编译器能够在不改变 JVM 的情况下实现新的 Java 语言功能。但是合成结构在不同的 Java 编译器实现中可能会有所不同，这意味着<code>.class</code>文件在不同的实现中也会有所不同。因此如果序列化内部类，然后使用不同的 JRE 实现对其进行反序列化，则可能会遇到兼容性问题。

## Local Classes - 局部类
局部类是定义在一个块中的类，是一组大括号之间的零个或多个语句，通常你会发现局部类定义在一个方法的区块里。

### 定义局部类
你可以在任何区块中定义局部类。例如：你可以在方法体、<code>for</code>循环或者 <code>if</code>语句中定义局部类。

下面的 <code>LocalClassExample</code> 示例用来校验两个电话号码，它在 <code>validatePhoneNumber</code> 方法中定义了局部类 <code>PhoneNumber</code>：
```java
public class LocalClassExample {

    static String regularExpression = "[^0-9]";

    public static void validatePhoneNumber(String phoneNumber1, String phoneNumber2) {
        final int numberLength = 10;

        class PhoneNumber {
            String formattedPhoneNumber = null;

            PhoneNumber(String phoneNumber) {
                String currentNumber = phoneNumber.replaceAll(regularExpression, "");
                if (currentNumber.length() == numberLength) {
                    formattedPhoneNumber = currentNumber;
                } else {
                    formattedPhoneNumber = null;
                }
            }

            public String getPhoneNumber() {
                return formattedPhoneNumber;
            }

            // Valid in JDK 8 or later
            public void printOriginalNumbers() {
                System.out.println("Original numbers are " + phoneNumber1 + " and " + phoneNumber2);
            }
        }

        PhoneNumber myNumber1 = new PhoneNumber(phoneNumber1);
        PhoneNumber myNumber2 = new PhoneNumber(phoneNumber2);

        if (myNumber1.getPhoneNumber() == null) {
            System.out.println("First number is invalid");
        } else {
            System.out.println("First number is " + myNumber1.getPhoneNumber());
        }

        if (myNumber2.getPhoneNumber() == null) {
            System.out.println("Second number is invalid");
        } else {
            System.out.println("Second number is " + myNumber2.getPhoneNumber());
        }
    }

    public static void main(String... args) {
        validatePhoneNumber("123-456-7890", "456-7890");
    }
}
```

上面的示例运行结果如下：
```
First number is 1234567890
Second number is invalid
```

### 访问封闭类的成员
局部类可以访问外部类的成员。在上面的示例中， <code>PhoneNumber</code> 的构造函数访问了成员 <code>LocalClassExample.regularExpression</code>。

此外局部类可以访问局部变量，但是局部类只能访问声明为 <code>final</code> 的局部变量。当局部类访问封闭区块的本地变量或参数的时候，它捕获了那个变量或参数。例如，PhoneNumber 的构造函数可以访问本地变量 numberLength 是因为它被声明为 <code>final</code>，此时 numberLength 是一个捕获变量。

然而从<code>Java SE 8</code>开始，局部类可以访问封闭区块<code>final</code> 或者 <code>effectively final</code> 的局部变量或参数。***一个变量或者参数初始化后值不再改变称为 effectively final。*** 例如：假设变量 numberLength 没有被定义为 <code>final</code>，而是在 PhoneNumber 的构造函数中初始化 numberLength 的值为7：

```java
PhoneNumber(String phoneNumber) {
    numberLength = 7;
    String currentNumber = phoneNumber.replaceAll(regularExpression, "");
    ......
}
```

因为赋值语句，变量 numberLength 不再 <code>effectively final</code>。因此 Java 编译器会在 PhoneNumber 访问变量 numberLength 的地方生成错误信息 "local variables referenced from an inner class must be final or effectively final"：
```java
if (currentNumber.length() == numberLength)
```

从<code>Java SE 8</code>开始，如果在方法中定义局部类，局部类可以访问方法的参数:
```java
public void printOriginalNumbers() {
    System.out.println("Original numbers are " + phoneNumber1 +
        " and " + phoneNumber2);
}
```

### 局部类与内部类相似
局部类与内部类相似，因为它们无法定义或声明任何静态成员。静态方法中的局部类只能引用封闭类的静态成员，就像定义在静态方法 validatePhoneNumber 中的 PhoneNumber 类。如果你没有将成员变量  regularExpression 定义为 <code>static</code>，那么 Java 编译器就会生成错误信息 "non-static variable regularExpression cannot be referenced from a static context"。

局部类由于是非静态的，所以它们可以访问封闭区块的实例变量，因此它们不能包含绝大部分的静态声明。

你不能在一个区块中定义接口，因为接口在本质上是静态的。下面的代码无法编译通过，因为在 greetInEnglish 方法中定义了接口 HelloThere ：
```java
public void greetInEnglish() {
    interface HelloThere {
       public void greet();
    }
    class EnglishHelloThere implements HelloThere {
        public void greet() {
            System.out.println("Hello " + name);
        }
    }
    HelloThere myGreeting = new EnglishHelloThere();
    myGreeting.greet();
}
```

你不能在局部类中声明静态初始化器或成员接口。下面的代码无法编译通过，因为方法  EnglishGoodbye.sayGoodbye 被定义为 <code>static</code>。当编译器遇到这个方法时会生成错误信息 “modifier 'static' is only allowed in constant variable declaration”：
```java
public void sayGoodbyeInEnglish() {
    class EnglishGoodbye {
        public static void sayGoodbye() {
            System.out.println("Bye bye");
        }
    }
    EnglishGoodbye.sayGoodbye();
}
```

局部类可以包含常量静态成员。常量是一个定义为<code>final</code> 的基本数据类型或<code>String</code>类型的变量，通过编译时常量表达式进行初始化。编译时常量表达式通常是一个字符串或一个可以在编译时评估出值的算数表达式。下面的代码可以编译通过，因为静态成员 EnglishGoodbye.farewell 是一个常量：
```java
public void sayGoodbyeInEnglish() {
    class EnglishGoodbye {
        public static final String farewell = "Bye bye";
        public void sayGoodbye() {
            System.out.println(farewell);
        }
    }
    EnglishGoodbye myEnglishGoodbye = new EnglishGoodbye();
    myEnglishGoodbye.sayGoodbye();
}
```

## Anonymous Classes - 匿名类
匿名类可以让代码更简洁，匿名类允许在同一时间声明和实例化一个类。除了没有类名之外，匿名类跟局部类很像。如果你只需要使用局部类一次，那么推荐你使用匿名类。

### 定义匿名类
局部类是类声明，而匿名类是表达式，这意味着你在其它表达式里面定义了一个类。下面的 HelloWorldAnonymousClasses 示例在初始化本地变量frenchGreeting 和 spanishGreeting 使用了匿名类，而在初始化变量 englishGreeting 使用了本地类：

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

### 匿名类的语法
如前所述，匿名类是一个表达式。匿名类表达式的语法就像一个构造函数的调用，除了在代码中包含一块类定义的区域。

思考一下 frenchGreeting 对象的初始化：
```java
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
```

匿名类的语法由如下组成：
- 新的操作者
- 用于实现的接口或者继承的类的名称。在这个示例中，匿名类实现了接口 HelloWorld。
- 一个包含构造函数参数的括号，就像一个正常类创建实例的表达式。注意：当你实现接口的时候，接口没有构造函数，所以需要像这个例子一样使用一对大括号。
- 一块用来定义类的区域。更具体的说，在该区域内可以定义方法但是不能声明方法。(定义方法和声明方法的区别请阅读参考文章2)

因为匿名类定义是一个表达式，所以它必须是声明的一部分。在这个例子中，匿名类表达式是实例化 frenchGreeting 对象声明的一部分，这解释了为什么大括号后面有一个分号。

### 访问封闭范围的局部变量，声明和访问匿名类的成员
匿名类可以像局部类一样捕获变量，它们对封闭范围的局部变量有同样的访问权限：
- 匿名类可以访问封闭类的成员
- 匿名类不能访问封闭范围内未声明为 <code>final</code> 或 <code>effectively final</code> 的局部变量
- 像嵌套类一样，匿名类的类型声明（例如变量）会隐藏包含范围中具有相同名称的其它任何声明。

匿名类和局部类对于它们的成员有同样的限制：
- 你不能在一个匿名类中声明静态初始化器或成员接口
- 匿名类可以拥有静态常量成员

注意你可以在匿名类中声明下面的东西：
- 字段
- 额外的方法（即使它们没有实现超类的任何方法）
- 实例初始值设定项
- 局部类

但是你不能在匿名类中声明构造函数。

## 参考文章
[1. Nested Classes - Java Documentation](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html)
[2. What is the difference between declaration and definition in Java? - StackOverflow](https://stackoverflow.com/questions/11715485/what-is-the-difference-between-declaration-and-definition-in-java)

---
layout: post
title: 『笔记』Thinking in Java - 访问权限控制
category: Java
tags: []
---

类库重构时，避免破环客户端代码（即调用类库的代码），所以需要指明类库的哪些构件是可以被外部调用的，隐藏类库内部实现的细节，Java通过“访问权限标识符 access specifiers”解决这个问题（access control or implementation hiding）。

## 包：库单元 (package: the library unit)
两个类内部的方法即使重名也不会引起冲突，但类重名呢？所以需要对名称空间完全控制，为每个类创建惟一的标识符组合。使用package解决这个问题，把class放在package内。

``` Java
|--src
    |--(default package)
        |--MyClass.java
        |--QualifiedMyClass.java

public class MyClass {
}

public class QualifiedMyClass {
    public static void main(String[] args) {
        new MyClass();    // 因为2个类处于同一个package，所以可以直接调用。
    }
}
```

如果调用不同package的class呢？

``` Java
|--src
    |--(package)access
        |--QualifiedMyClass.java
    |--(package)access.mypackage
        |--MyClass.java

// package声明该类是access.mypackage类库的一部分。package名对应到文件目录是src/access/mypackage.
// 若想使用该类，必须指定包名和类名构成的完整的标识符。
package access.mypackage;
public class MyClass {
    public MyClass() {
        System.out.println("MyClass");
    }
}

package access;
// 可以由import导入，也可以在代码行中写全名（显得冗长）。
import access.mypackage.MyClass;
public class QualifiedMyClass {
    public static void main(String[] args) {
        new access.mypackage.MyClass(); // 如果没有import，必须写全名，否则cannot be resolved to a type
        new MyClass();  // 因为import导入了，所以可以直接写类名
    }
}
```

Java创建package名称的惯例，使用倒序的internet域名，以此保证package名称的惟一性。

## Java访问权限标识符 (Java access specifiers)
类通过access specifiers控制着成员（字段、方法）的访问权限。

- package access: 没有权限标识符的情况下，默认是包访问权限，可以被包内的其它类访问；
- public (interface access): 可以被任意访问；
- private (you can't touch that!): 只允许被类内部访问；
- protected (inheritance access): 允许基类的成员仅能被派生类访问（protected也提供包访问权限）。

## 接口和实现 (Interface and implementation)

## 类访问权限 (Class access)
类只有2个访问权限：public和包访问权限。不可以使用private, protected.
一个Java源码文件通常被称为编译单元，以.java后缀，只能有一个public class，类名必须与文件名相同（包括大小写）。如果还有non-public classes，则对包之外是隐藏的 ，它们的主要目的是为主pulic class提供支持。
拿掉public，也就是没有为类指定访问权限标识符，那么就是包访问权限。

``` Java
class Soup1 {
    // 如果不希望类被任何人访问，可以把所有的构造器指定为private，从而阻止任何人创建该类的对象。
    private Soup1() {
        System.out.println("private Soup1 constructor");
    }
    // 但通过该类的static method，仍可以new该类的对象并返回它的引用。这是第1种方法。
    // 若想在返回引用前在Soup1上做额外的处理，或想记录创建了几个Soup1对象，这种方法很有用处。
    // 为什么static method可以调用private构造器？
    // 7.9.1“继承与初始化”脚注里提到，构造器也是static方法，尽管static并未显示地写出。
    static Soup1 makeSoup() {
        return new Soup1();
    }
}
 
class Soup2 {
    private Soup2() {
        System.out.println("private Soup2 constructor");
    }
    // 这样的类往往提供一些static method生成该类的实例，
    // 只要这个类有public method/field，仍可以供外部调用，比如“单例(singleton)”, 这是第2种方法：
    // 这是一种特殊的设计模式，通过 private static成员创建该类的对象，因为static，所以只能创建一个对象，
    private static Soup2 ps1 = new Soup2();
    // 然后只提供一个static method访问。
    static Soup2 access() {
        return ps1;
    }
    // 这是供外部调用的public method.
    public void f() {
        System.out.println("Soup2.f()");
    }
}
 
public class Lunch {
    void testPrivate() {
        // Unresolved compilation problem: The constructor Soup1() is not visible.
        // Soup1 soup = new Soup1();
    }
    void testStatic() {
        Soup1.makeSoup();
    }
    void testSingleton() {
        Soup2.access().f();
    }
}
```
 

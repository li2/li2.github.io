---
layout: post
title: 『笔记』Thinking in Java - 对象
category: Java
tags: []
---

## 抽象过程
一种建模方式，要求程序员建立在机器模型（位于“**解空间**”内）和实际待解决问题模型（位于“**问题空间**”）之间的关联。
另一种，只针对待解问题建模。
面向对象的方法，通过向程序员提供工具，以表示问题空间中的元素，这些元素称为“对象”，使得程序员不受制于特定类型的问题。所以，设计良好的OOP程序，由两部分组成：用来表示问题空间概念的对象，以及发送给这些对象的用来表示此空间内行为的消息。（程序是对象的集合，它们通过发送消息来告知彼此所要做的。）

## 每个对象都有一个接口(An object has an interface)

![每一个对象都有一个接口](/assets/img/java/java-每个对象都有一个接口.png)

``` Java
Light lt = new Light();
lt.on();
```
类型/类(type/class)的名称是**Light**，特定**Light**对象(object)的名称是**lt**，可以向**Light**对象发出的请求(requests)是：打开它、关闭它、调亮它、调暗它。你以下列方式创建了一个**Light**对象：定义这个对象的“引用reference” **lt**，然后调用**new**方法请求该类型的新对象。为了向对象发送消息，需要声明对象的名称，并以圆点符号连接一个消息。这差不多就是用对象进行设计的全部。

## 每个对象都提供服务(An object provides services)
着手从事一个程序设计时要问一下自己：“如果我可以将问题从表象中抽取出来，那么什么样的对象可以马上解决我的问题？”
人们在设计对象时面临的问题是，将过多的功能都塞在一个对象中。在良好的面向对象设计中，每个对象都可以很好地完成一项任务，但它并不试图做更多的事情。

## 被隐藏的具体实现(The hidden implementation)
类创建者只向客户端程序员暴露必需的部分，而隐藏其它部分，一是避免对象内部被损坏以减少bug，二是允许库设计者改变类内部的工作方式而不用担心影响到客户端程序员。
Java用三个explicit keywords在类的内部设定边界：public, private, protected. （访问指示符access specifier）

## 复用具体实现(Reusing the implementation)
直接使用该类的对象。
将类的对象置于新类中，称为“创建一个成员对象”。新类可以由任意数量和类型的其它对象组成，称为组合(composition)，如果组合是动态发生的，称为聚合(aggregation).


## 继承(Inheritance)
添加新方法、覆盖(overriding).
如果继承只覆盖基类的方法，而不添加新的方法，意味着导出类和基类具有完全相同的接口，称导出类和基类是"is-a"（是一个）的关系。
如果导出类添加了新的接口，那么两者之间是"is-like-a"（像一个）的关系。
![继承](/assets/img/java/java-继承.png)

## 伴随多态的可互换对象(Interchangeable objects with polymorphism)
将导出类型的对象(derived-type objects)当作其泛化基类型对象(generic base types)来看待时，可以编写出不依赖于特定类型的代码。
但是如果某个方法要操作泛化基类型对象，编译器在编译时不可能知道应该执行哪一段代码，编译器只能确保被调用的方法存在，并检查参数和返回值类型。在OOP中使用后期绑定(late binding)解决这个问题，当向一个对象发送消息时，对象就知道对这条消息应该做什么。

``` Java
void doSomething(Shape shape) {
    shape.erase();
    shape.draw();
}
Circle circle = new Circle();
Square square = new Square();
doSomething(circle);        // 所以这两个doSomething()调用的是不同的draw.
doSomething(square);
```
doSomething()可以与任何Shape对话，因此它独立于具体的类型（比如Circle, Square, Triangle）。
Circle传给doSomething()时，会被看作Shape，也就是说doSomething()可以发送给Shape的消息，Circle都可以接收。（这里面有一个被称为向上转型upcasting的概念）。

## 容器(Containers)
当不知道需要多少空间来存储对象时，使用容器。这是一个新的对象，持有对其它对象的引用，而且在任何时候都可以扩充以容纳更多的对象。List, Map, Set, queue, tree, stack.
把一个object reference置入容器时，会被向上转型为Object，因此会丢失其身份。当把它取回时，就获取了一个对Object对象的引用，那么怎么才能向下转型为具体的对象类型呢？解决的办法是，记住容器中保存的对象类型。被称作参数化类型机制 Parameterized types (generics). 在java中称为泛型generics, 一对尖括号，中间包含类型信息。

``` Java
ArrayList<Shape> shapes = new ArrayList<Shape>();
```

## 对象的创建和生命周期(Object creation & lifetime)
在被称为heap的内存池中动态地创建。Java完全采用了动态内存分配方式，每当要创建新对象时，就要使用new operator来构建对象的动态实例(dynamic instance). 牺牲了效率（在heap中分配存储空间需要的时间，远大于在stack中），但带来了更大的灵活性。
在堆上创建对象，编译器就不知道它的生命周期，直到相关代码被执行的那一刻才能确定。Java提供垃圾回收机制，自动发现不再被使用的对象并销毁它。

## 用引用操作对象 & 必须由你创建所有对象
(You manipulate objects with references & You must create all the objects)
**Java中（几乎）一切都是对象，但操作的标识符实际上是对象的一个“引用 reference”.** 

``` Java
String s;    // 创建的只是引用，并不是对象。如果向s发送消息，会得到错误。
String s = "a string";    // 安全的做法是，创建引用时就初始化，建立与对象的关联。
String s = new String("a string");    // 采用通用的初始化方法，new表示“给我一个新对象”。
```
对于基本类型(Primitive type)，不需要new存储于heap中，采取和C相同的办法，存储于stack中，比如boolean, char, byte, short, int, long, float, double, void.
 “wrapper” classes 可以把上述基本类型包装成对象。

Java中的数组
**Java确保数组会被初始化**。若创建一个数组对象就创建了一个数组引用，每个引用都会被初始化为null，而试图操作一个还是null的引用，运行时会报错；若创建一个基本类型数组，数组所占的内存会被全部置零。

## 永远不需要销毁对象(You never need to destory an object)
因为对象存储于heap中，所以其生命周期和基本类型的生命周期不一样。

``` Java
{
    String s = new String("a string");
} // end of scope
```

对象的引用存储于stack中，所以s在作用域结束后就消失了，但s指向的对象仍位于内存空间中。因为惟一的引用已经消失，所以无法访问这个对象，即使它还存在。稍后将学习如何传递和复制对象。

## 创建新的数据类型：类 (Creating new data types: class)
面向对象的编程语言采用关键字**class**表示“我准备告诉你一种新类型的对象看起来像什么样子”。这样就引入了一个新的类型（**在Java中所做的全部工作就是定义类，产生类的对象，发送消息给这些对象**）。然后定义类的字段fields和方法methods.
**如果类的字段是基本数据类型，Java确保其得到初始化**。（但Java不会初始化局部的基本类型变量，若使用了未初始化的局部变量，编译时返回错误）。
调用一个方法，被称为**“向对象发送消息”**。
**当对象作为方法的参数传递时，实际传递的是对象的引用。**

Java采用Internet域名来保证类库的名字空间(namespaces)的惟一性。由于方法存在于类中，就可以避免与其它类同名函数的冲突。
当要引用其它类时，使用关键字import通知编译器。
**创建类时，只是对该类的描述。只有new创建类的实例时，才分配存储空间**（而创建几个对象，就会分配几个存储空间），方法才可供外部调用。
然而有两种需求：其一，只想为某个field分配单一的存储空间，和创建的对象个数无关；其二，在创建对象被创建之前，也可以调用某个方法。**通过static关键字完成**，这样就可以通过类名直接引用。（要注意的是，即使在同一个类中，static-method也不可以直接访问non-static-field/method，否则编译器提示错误 Cannot make a static reference to the non-static field）。

``` Java
package firstJavaProgram;
public class HelloWorld {
    private String name;
    public HelloWorld() {
        System.out.println("Hello World");
    }
    public HelloWorld(String name) {
        System.out.println("Hello World, I'am " + name);
    }
    public static void main(String[] args) {
        new HelloWorld();
        new HelloWorld("li2");
    }
}
```

编译后生成HelloWorld.class，

``` Java
>javap -private HelloWorld.class
Compiled from "HelloWorld.java"
public class firstJavaProgram.HelloWorld {
  private java.lang.String name;
  public firstJavaProgram.HelloWorld();
  public firstJavaProgram.HelloWorld(java.lang.String);
  public static void main(java.lang.String[]);
}
```

如果省略构造器，

``` Java
package firstJavaProgram;
public class HelloWorld {
}
```

编译后：

``` Java
>javap -private HelloWorld
Compiled from "HelloWorld.java"
public class firstJavaProgram.HelloWorld {
  public firstJavaProgram.HelloWorld();
}
```
 

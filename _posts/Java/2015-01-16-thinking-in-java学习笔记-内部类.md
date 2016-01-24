---
layout: post
title: 『笔记』Thinking in Java - 内部类
category: Java
tags: []
---
 
## 创建内部类、链接到外部类、使用.this和.new
创建内部类(inner class)的方法是把类的定义置于外围类(surrounding class)的里面，这不同于组合。
在外围类方法里使用内部类，与使用普通类没有什么不同。

``` java
public class Outer {
private String str = "I'm a Outer class private field";
class Inner {
// 通过Outer.this获取外部类对象的引用; this代表内部类自己。
public Outer outer() {
System.out.println(this.getClass());
System.out.println(Outer.this.getClass());
return Outer.this;
}
// 内部类(inner class)自动拥有外部类(enclosing class)成员的访问权限，
// 这是因为内部类可以私密地获取创建它的外部类对象的引用，通过这个引用访问外部类对象的成员。
// 所以内部类的对象只能被相关联的外围类对象所创建。
public void referToOuterField() {
System.out.println(str);
}
}
// 通过外部类的方法返回内部类的引用。
Inner inner() {
return new Inner();
}
// 而非静态方法，就可以直接创建内部类的对象。
void nonStaticMethod() {
Inner i = new Inner();
}
public static void main(String[] args) {
// 在静态方法内创建内部类的对象，必须具体指明对象的类型Outer.Inner, 而不能直接使用Inner,
// 否则会提示错误“没有可访问的外部类对象的引用” No enclosing instance of type Outer is accessible.
// Must qualify the allocation with an enclosing instance of type Outer
// (e.g. x.new A() where x is an instance of Outer).
//! Inner i = new Inner(); // 方法1是错误的

// 而且，必须使用外部类的对象，不能直接使用外部类名，解决了内部类名字作用域的问题（TODO 不理解）。
Outer outer = new Outer();
Outer.Inner i = outer.inner();  // 方法2，通过外部类的方法返回内部类的引用。
Outer.Inner i2 = outer.new Inner(); // 方法3，通过new
//! Outer.Inner i3 = Outer.new Inner(); // 错误

i.referToOuterField();
i.outer();
}/*
I'm a Outer class private field
class innerclass.Outer$Inner
class innerclass.Outer
*/
}
```

内部类编译后：

``` Java
package innerclass;
public class Outer {
Outer() {}
Outer(String who) {}
private int value;

class Inner {
Inner() {}
Inner(String inner) {}
}
}

javap -private Outer
Compiled from "Outer.java"
public class innerclass.Outer {
private int value;
innerclass.Outer();
innerclass.Outer(java.lang.String);
}

innerclass>javap -private Outer$Inner.class
Compiled from "Outer.java"
class innerclass.Outer$Inner {
final innerclass.Outer this$0;    // 包含外围类的引用
innerclass.Outer$Inner(innerclass.Outer);    // 注意第0个参数
innerclass.Outer$Inner(innerclass.Outer, java.lang.String);
}
```



## 内部类和向上转型

``` Java
// Selector接口使用了iterator设计模式，隐藏了遍历对象的内部结构。
interface Selector {
boolean end();
Object current();
void next();
}

// Sequence类只是一个固定大小的Object数组，以类的形式包装，
// 调用add()在数组末尾增加新Object, 调用Selector接口获取数组的每一个元素。
public class Sequence {
public Sequence(int size) { items = new Object[size]; }
public void add(Object x) {
if (next < items.length)
items[next++] = x;
}
public Selector selector() { return new SequenceSelector(); }

private Object[] items;
private int next = 0;
// inner class自动拥有enclosing class成员的访问权限，
// 这是因为内部类可以获取创建它的外部类对象的引用，通过这个引用访问外部类对象的成员。
private class SequenceSelector implements Selector {
private int i = 0;
@Override public boolean end() { return (i == items.length); }
@Override public Object current() { return items[i]; }
@Override public void next() { if (i < items.length) i++; }
}

public static void main(String[] args) {
Sequence sequence = new Sequence(4);
for (int i=0; i<4; i++) {
sequence.add(Integer.toString(i));
}
// 内部类向上转型成基类或接口时，只能得到基类/接口的引用，那么内部类的实现细节被隐藏。
Selector selector = sequence.selector();
while(!selector.end()) {
System.out.print(selector.current() + " ");
selector.next();
}
}
}
```

## 在方法和作用域的内部类(Inner classes in methods and scopes)
可以在方法内或者某个作用域内（比如if）定义内部类，称为**局部内部类**，两个应用场景：
(1) 实现了某类型的接口，可以创建并返回对它的引用；
(2) 要解决一个复杂的问题，需要创建一个类辅助解决方案，但不希望这个类是公共可用的。

## 匿名内部类

``` Java
package innerclass;
abstract class Basic {
Basic() {
}
Basic(String who, String Where) {
}
abstract String get();
}
public class Outer2 {
private int outerValue = 1;
public Basic AnonymousInner(int i, final String who, final String where) {
return new Basic() {  // 这是一个无参的匿名类，因为编译后也会生成一个构造器，所以这里的有参/无参是针对构造器而讲
//return new Basic(who, where) {    // 若基类的构造器有参数，那么也可以构建有参的匿名类
{
// 匿名内部类不能显示地声明构造器（因为没有名字），
// 可以通过instance initializer创建一个构造器效果，
// 但执行顺序和放置位置有关（一般创建对象时，先执行实例初始化，在执行构造器）
// 所以innerValue值是2015.
System.out.println("Inside instance initializer, Constructor-like.");
innerValue = 2011;
}
private int innerValue = getInt(2015);
public String get() {
// 如果匿名内部类使用了外部定义的对象，那么编译器要求其参数必须是final, 否则编译时出错：
// Cannot refer to a non-final variable para inside an inner class defined in a different method
return (who + " at " + where + "." + innerValue);
}
private int getInt(int i) {
System.out.println("getInt");
return i;
}
}; // 一个return语句，返回一个新的对象，同时定义这个类，而类没有名字，称为匿名内部类(anonymous inner class)
// 所以这里的分号只是一条语句的结束标志。
}
public static void main(String[] args) {
Outer2 ai = new Outer2();
Basic b = ai.AnonymousInner(1, "li2", "ShangHai");
System.out.println(b.get());
}/* Output
Inside instance initializer, Constructor-like.
getInt
li2 at ShangHai.2015
*/
}
```

``` Java
innerclass>javap -private Outer2
Compiled from "Outer2.java"
public class innerclass.Outer2 {
private int outerValue;
public innerclass.Outer2();
public innerclass.Basic AnonymousInner(java.lang.String, java.lang.String);
public static void main(java.lang.String[]);
}

innerclass>javap -private Outer2$1
Compiled from "Outer2.java"
class innerclass.Outer2$1 extends innerclass.Basic {
private int innerValue;
final innerclass.Outer2 this$0;   // 外围类的引用
private final java.lang.String val$who;   //传进来的参数成了private实例变量，也就是说在匿名内部类中有一份参数的拷贝，要求参数是final的理由：匿名内部类不能改变参数的值或者引用。
private final java.lang.String val$where;
innerclass.Outer2$1(innerclass.Outer2, java.lang.String, java.lang.String);   // 这是return new Basic()编译后生成的构造器，注意第0个参数
public java.lang.String get();
private int getInt(int);
}

innerclass.Outer2$1(innerclass.Outer2, java.lang.String, java.lang.String, jav
a.lang.String, java.lang.String);   // return new Basic(who, where)生成的构造器，Outer2$1.class文件的其余行是一样的。
```
## 嵌套类
静态内部类被称为嵌套类(Nested class)，和普通内部类的区别是，不存在外围类对象的引用，从编译后的文件也可以看出。

``` Java
package innerclass;
public class Outer {
static class Inner {
Inner() {}
Inner(String inner) {}
}
}

innerclass>javap -private Outer$Inner.class
Compiled from "Outer.java"
class innerclass.Outer$Inner {
innerclass.Outer$Inner();
innerclass.Outer$Inner(java.lang.String);
}
```

## 为什么需要内部类
### 闭包与回调
不同于外围类实现接口的方式，每个内部类都可以独立继承一个接口，不论外围类是否已经继承了某个接口，对内部类都没有影响。所以内部类解决了多重继承(multiple-inheritance)的问题。
闭包(closure)是一个可调用的对象，它记录了一些信息，这些信息来自于创建它的作用域。所以内部类是面向对象的闭包。

``` Java
package innerclass;
interface Incrementable {
void increment();
}
// 通过外围类实现一个接口。如果只需要实现一个接口，可以满足需要，就可以通过外围类实现。
class Callee1 implements Incrementable {
private int i = 0;
@Override public void increment() {
i++;
System.out.println(i);
}
}
class MyIncrement {
// 类MyIncrement和接口Incrementable不相关，所以虽然方法同名，但也是不相关的。
public void increment() {
System.out.println("Other operation");
}
static void f(MyIncrement mi) {
mi.increment();
}
}
class Callee2 extends MyIncrement {
private int i = 10;
// Callee2继承了类MyIncrement，所以覆写的是MyIncrement的increment方法，
// 如果要实现接口Incrementable的increment方法，只能通过内部类独立地实现。
@Override public void increment() {
super.increment();
i++;
System.out.println(i);
}
// 通过内部类实现一个接口。这是一个闭包。
private class Closure implements Incrementable {
@Override public void increment() {
Callee2.this.increment();
}
}
// 通过内部类提供的闭包功能，完成回调(callback)的功能，而不是通过指针实现回调。
// Closure是private，所以外部只能通过下面的方法得到接口的引用，也就限制了外部可以做的事情。
Incrementable getCallbackReference() {
return new Closure();
}
}
class Caller {
private Incrementable callbackReference;
// Caller使用接口Incrementable的引用作为参数，那么就可以根据需要调用方法的不同实现。
Caller(Incrementable cbh) {
callbackReference = cbh;
}
void go() {
callbackReference.increment();
}
}
public class Callbacks {
public static void main(String[] args) {
Callee1 c1 = new Callee1();
Callee2 c2 = new Callee2();
MyIncrement.f(c2);
Caller caller1 = new Caller(c1);
Caller caller2 = new Caller(c2.getCallbackReference());
caller1.go();
caller1.go();
caller2.go();
caller2.go();
}/*Output
Other operation
11
1
2
Other operation
12
Other operation
13   
*/
}
```

### 内部类和应用程序框架 (inner classes & control frameworks)
应用程序框架(application framework)是被设计用以解决某类特定问题的一个类或一组类。要使用某个应用程序框架，通常是继承一个或多个类，并覆盖某些方法。在覆盖后的方法中，编写代码以解决特定问题。
这是设计模式中的“模板方法”，模板方法是保持不变的事物，可覆盖的方法是变化的事物。
控制框架是特殊的应用程序框架，用以解决响应事件。




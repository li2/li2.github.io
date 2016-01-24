---
layout: post
title: 『笔记』Thinking in Java - 多态
category: Java
tags: []
---
 
多态(Polymorphism).
接口
内部类（闭包）
泛型

## 向上转型
如果方法仅接收基类作为参数，在调用该方法时传递的是导出类，这样就可以不用为每一个导出类编写对应的方法；
或者，把导出类显示地赋值给基类，目的是只与基类的方法打交道；
这两种情况下隐含的问题是，调用时发生了upcasting，而编译器（在编译阶段）不知道导出类的类型，但运行时却能正确调用导出类的覆写方法。这是因为“**后期绑定**”（Late binding，又称为动态绑定dynamic binding、运行时绑定runtime binding）。

**Java中除了static和final方法（private method已经被隐式地标识为final），其它方法都是动态绑定**。
这样看来，final有2个用处，防止导出类覆写该方法，禁止该方法的动态绑定。但禁止动态绑定，多数情况下不会明显改善程序性能，所以不要以提升性能为目的使用final.

只有method的调用是多态的。而**field的访问在编译时就能确定，不是多态的**。
如果导出类中有和基类名字相同的字段a，那么导出类将包含两个a，位于不同的存储空间，必须通过super.a显示地调用基类的a.

## 构造器和多态
构造器的调用顺序在“复用类, 初始化及类的加载”中提到。而**cleanup的顺序和初始化的顺序相反**。

如果成员对象为其它多个对象所共享，需要使用引用计数(reference counting)来跟踪访问共享对象的对象数量。（这不是语言本身的特性吗？需要程序员自己去设计实现？TODO）

在前述的“初始化及类的加载”中，并没有详述初始化在构造器内部的细节。首先要知道，如果构造器调用了一个动态绑定的方法，实际调用的是覆写的方法。由于基类构造器先执行，调用了导出类覆写的方法，而此时导出类的对象还处于未初始化状态（构造器没有执行；实例变量未初始化，但分配存储空间时全被初始化为二进制的0）。所以编译器不会报错，但程序执行逻辑是错误的，这就是问题所在。
因此，**编写构造器的准则是：用尽可能简单的方法使对象进入正常状态，避免调用其它方法**。而在构造器内能被安全调用的方法是基类中不可被覆写的方法。

## 协变返回类型 (Covariant return types)
导出类的覆写方法可以返回**基类方法返回类型的某种导出类型**（可以认为是更“**狭窄**”的类型）。
比如基类方法返回Object, 导出类覆写方法可以返回String.
协变（covariant），如果它保持了子类型序关系≦，该序关系是：子类型≦基类型。
[维基百科-协变与逆变](http://zh.wikipedia.org/wiki/协变与逆变)

## 用继承进行设计
使用现成的类建立新类时，如果首先考虑继承技术，会使事情不必要得复杂起来。最好首先选择“组合”，尤其是不确定应该使用哪一种方式。组合不会强制程序设计进入继承的层次结构中。组合更加灵活地选择类型（因此也就选择了行为）。
纯粹的继承（is-a）使得基类和导出类具有完全相同的接口，所以基类可以接收导出类收到的任何消息。但解决问题时总需要扩展接口（is-like-a），而upcasting会丢失类型的信息，导致基类就不能访问导出类扩展的接口。所以需要downcasting以查明确切的类型，那么就需要某种方法确保向下转型的正确性。

``` Java
class Useful {
public void f() {}
public void g() {}
}

class MoreUseful extends Useful {
public void u() {}
}

public class RTTI {
public static void main(String[] args) {
Useful[] x = {
new Useful(),
new MoreUseful(),
};
x[0].f();
x[1].g();
// 编译时就可以进行的错误检查，Compile time: The method is undefined for the type Useful
//！ x[1].u();
// 运行时类型鉴别 runtime type identification(RTTI).
((MoreUseful)x[1]).u(); // 类型正确，downcast成功。
((MoreUseful)x[0]).f(); // 类型错误，downcast失败，抛出ClassCastException.
}
}
```

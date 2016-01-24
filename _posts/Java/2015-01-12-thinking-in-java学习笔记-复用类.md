---
layout: post
title: 『笔记』Thinking in Java - 复用类
category: Java
tags: []
---


## 组合语法 (Composition syntax)
几个基本数据类型、几个String对象、另一个类的对象，把这些组合在一起，就是一个新类！

## 继承语法 (Inheritance syntax)
继承会自动得到基类的所有field和method. 通过**关键字super**调用继承到的方法。
可以改写继承到的方法，可以添加新的方法。
基类的构造器总会被调用，而且在导出类构造器之前被调用。
如果想调用带参数的基类构造器，就必须用super显示地调用基类的构造器。

## 代理 (Delegation)
??

## 结合使用组合和继承 (Combining composition and inheritance)

确保正确清理??

如果基类的某个方法已经被重载多次，导出类仍然可以继续重载该方法。

Method overloading deals with the notion of having two or more methods(functions) in the same class with the same name but different arguments.
**方法重载，一个类中有多个名称相同、参数列表不同的方法。**
Method overriding means having two methods with the same arguments, but different implementation. One of them would exist in the Parent class (Base Class) while another will be in the derived class(Child Class).
**方法覆写，在导出类中，重新实现基类的某个方法，而方法的参数列表和名称仍然要保持一致。**

所以为了避免混淆，可以给需要覆写的方法标注**注释(annotation) `@Override`**, 当这个方法被错误地重载时，编译器会报错：The method must override or implement a supertype method.

## 在组合和继承之间选择 (Choosing composition vs. inheritance)

## protected关键字

## 向上转型 (Upcasting)

## final关键字

``` Java
class Value {
    int i;  // package access
    public Value(int i) {
        this.i = i;
    }
}
public class FinalData {
    private static Random rand = new Random(47);
    private String id;
    public FinalData(String id) {
        // 既然定义处没有初始化，那么必须在构造器中对blank final初始化。
        // 这样类中的final字段就可以根据对象的不同而不同（显然需要通过有参构造器完成），同时保持其值（或者引用）恒定不变的特性。
        blankFinal = 0;
        this.id = id;
    }
    // [1] 关于 Blank final field.
    // blank final在使用前必须被初始化，否则编译器报错：
    // The blank final field blank may not have been initialized.
    private final int blankFinal;
 
    // [2] 编译时常量 (compile-time constants, 即有常量初始值的变量)
    private final int valueOne = 9;
    private static final int VALUE_TWO = 99;
    // 这是更为典型的常量定义方式，public说明可用于包外，static说明只有一份，final说明是常量。
    // final static，全用大写字母命名，字母间用下划线隔开。
    public static final int VALUE_THREE = 39;
 
    // [3] 关于static
    // 即使数据是final，编译时也不一定能知道其值，比如用random方式初始化。
    private final int i4 = rand.nextInt(20);
    // 这里使用random方式初始化，同时说明static的意义，static声明的变量，
    // 在第一次被访问时其值就已确定，不会因为创建新的对象而再次被初始化。
    private static final int INT_5 = rand.nextInt(20);
 
    // [4] 关于final，final使基本类型变量的值恒定，对于object
    // 因为标识符实际是对象的引用，所以使对象的引用恒定，也就是对象的成员是可以被改变的。
    private Value v1 = new Value(11);
    private final Value v2 = new Value(22);
    private static final Value VAL_3 = new Value(33);
    private final int[] a = {1,2,3,4,5,6};
 
    private void f(final int i) {
        // [5] final参数，可以读取，但不能修改。主要用于向匿名内部类传递数据。
        System.out.println(i);
        //! i++;
    }
 
    public String toString() {
        return id + ": " + "i4 = " + i4 + ", INT_5 = " + INT_5;
    }
    public static void main(String[] args) {
        FinalData fd1 = new FinalData("fd1");
        //! fd1.valueOne++; // The final field cannot be assigned.
 
        fd1.v1 = new Value(9);  // Object isn't constant, so can be modified.
        //! fd1.v2 = new Value(0);  // The final field cannot be assigned.
        fd1.v2.i++;
        //! fd1.VAL_3 = new Value(1);   // The final field cannot be assigned.
 
        //! fd1.a = new int[3]; // The final field cannot be assigned.
        for (int i=0; i<fd1.a.length; i++) {
            fd1.a[i]++;
        }
        System.out.println(fd1);
        System.out.println("Creating new FinalData");
        FinalData fd2 = new FinalData("fd2");
        System.out.println(fd1);
        System.out.println(fd2);
    } /* Output
    fd1: i4 = 15, INT_5 = 18
    Creating new FinalData
    fd1: i4 = 15, INT_5 = 18
    fd2: i4 = 13, INT_5 = 18
    */
}
}
```

## 初始化及类的加载 (Initialization and class loading)

``` Java
class Insect {
    private int i = printInit("Insect.i initialized", 11);
    protected int j = printInit("Insect.j initialized", 12);
    Insect(int id) {
        System.out.println("Insect" + id + ".constructor: i = " + i + ", j = " + j);
        j = id;
    }
    private static int x1 = printInit("static Insect.x1 initialized", 14);
    static int printInit(String s, int val) {
        System.out.println(s);
        return val;
    }
}
 
public class Beetle extends Insect{
    private int k = printInit("Beetle.k initialized", 15);
    public Beetle(int id) {
        super(id);
        System.out.println("Beetle" + id + ".constructor: k = " + k + ", j = " + j);
    }
    private static int x2 = printInit("static Beetle.x2 initialized", 16);
    public static void main(String[] args) {
        System.out.println("Beetle.main()");
        Beetle b1 = new Beetle(1);
        Beetle b2 = new Beetle(2);
    } /* Output:
    static Insect.x1 initialized            (1&2) 加载
    static Beetle.x2 initialized
 
    Beetle.main()
    Insect.i initialized                    (3) 基类的实例变量初始化
    Insect.j initialized
    Insect1.constructor: i = 11, j = 12     (4) 基类的构造器被调用
 
    Beetle.k initialized                    (5) 导出类的实例变量和构造器
    Beetle1.constructor: k = 15, j = 1
 
    Insect.i initialized                    (6) 创建第2个对象
    Insect.j initialized
    Insect2.constructor: i = 11, j = 12
    Beetle.k initialized
    Beetle2.constructor: k = 15, j = 2
    */
```
(1) Beetle开始运行时，首先试图访问Beetle.main(), 于是加载器开始启动并找出Beetle类的编译代码（在Beetle.class文件中），因为存在基类，那么会先加载基类，依次类推；
(2) 执行根基类的static初始化，然后是下一个导出类的static初始化，依次类推（**导出类的初始化可能依赖于基类是否正确初始化**）；
至此，必要的类加载完毕，然后就可以创建对象（如果不创建对象，上述过程也会执行）：
(3) 按顺序初始化基类的实例变量(Instance variable is the variable declared inside a class, but outside a method);
(4) 执行基类的构造器；
(5) 导出类同理，也是先初始化实例变量，再执行构造器；
(6) 加载类的动作只发生1次，也就是说第2次创建对象不会加载类。

## 总结
设计系统时，目标应该是找到或创建某些类，其中每个类都有具体的用途，既不会太大（包含太多功能而难以复用），也不会太小（不添加其它功能就无法使用）。如果你的设计变得过于复杂，将现有类拆分为几个小类，通常会有所帮助。

 

---
layout: post
title: 『笔记』Thinking in Java - 初始化与清理
category: Java
tags: []
---
 

## 用构造器确保初始化(Guaranteed initialization with the constructor)

``` Java
class RockLi2 {
    public RockLi2() {
        System.out.println("This is a default constructor.");
    }
}
public class SimpleConstructorLi2 {
    public static void main(String[] args) {
        // Java automatically calls constructor Rock() when an objcet is created.
        // 构造器的命名与类名相同。Java在用户有能力操作对象之前，自动调用构造器，从而确保初始化的执行。
        // 所以此处会看到打印信息："This is a default constructor."
        new RockLi2();
    }
}
```

## 方法重载(Method overloading)
几个方法可以有相同的名字，以参数列表来区分。方法重载的原因一，即使漏掉几个词，仍可以从语境中推断出含义，所以没必要向C语言中定义惟一的标识符；原因二构造器的名字已由类名决定。

## this关键字(The this keyword)

``` Java
class BananaLi2 {
    void peel(int i) {
        // this只能在方法内调用，表示“调用方法的那个对象的引用”，
        // method知道被哪个object调用，这是由编译器在幕后完成的，效果类似于 Banana.peel(a, 1);
        System.out.println(this.toString() + ".peel " + i);
    }
}
public class SimpleConstructorLi2 {
    public static void main(String[] args) {
        BananaLi2 b1 = new BananaLi2 ();
        BananaLi2 b2 = new BananaLi2 ();
        System.out.println("A string representation of object b1 is " + b1.toString());
        b1.peel(1);
        System.out.println("A string representation of object b2 is " + b2.toString());
        b2.peel(2);
    }
    /* Output:
    A string representation of object b1 is BananaLi2 @1e57e8f
    BananaLi2 @1e57e8f.peel 1
    A string representation of object b2 is BananaLi2 @1d7fbfb
    BananaLi2 @1d7fbfb.peel 2
    *///:~
}
```
可以在构造器中，使用this调用另一个构造器，注意最多只能调用一个。但不允许在方法中调用构建器。
如果在方法内调用同类的另一个方法，不必使用this。只有在可能引起混淆的地方，需要明确指出对象的引用时，才需要使用this。要记住不必处处使用this，要遵循一致直观的编程风格。

回头看static的含义，它是没有this的方法。

## 清理：终结处理和垃圾回收(Cleanup: finalization and garbage collection)

## 成员初始化(Member initialization)
若字段是基本类型，每个字段都会有一个默认初始值(false, 0, 0.0, ' ')。如果字段是对象引用，若不初始化，其值为null.
当然可以显示地指定初始化值，在定义字段的地方为其赋值。
也可以在构造器中显示地初始化。
要注意的是，即使字段定义散布于方法之间，**字段仍会在任何方法（包括构造器）被调用之前得到初始化，静态字段初始化先于非静态字段**。

``` Java
class BananaLi2 {
    public BananaLi2 (int i) {
        System.out.println("BananaLi2 (" + i + ")");
    }
    void peel(int i) {
        System.out.println("BananaLi2 .peel(" + i + ")");
    }
}
class Bananas {
    static BananaLi2 b1;
    static BananaLi2 b2;
    // 显示地static initialization, 被称作static block, 这段代码仅执行一次：
    static {
        b1 = new BananaLi2 (20150108);
        b2 = new BananaLi2 (20150109);
    }
    public Bananas() {
        System.out.println("Bananas()");
    }
}
 
public class SimpleConstructor {
    public static void main(String[] args) {
        System.out.println("main:");
        Bananas.b1.peel(1);    // (1) 当静态字段被首次访问时
        // static Bananas bananas = new Bananas();    // (2) 当首次创建该类的一个对象时
    }
    /*
    (1) Output:
    main:
    BananaLi2 (20150108)
    BananaLi2 (20150109)
    BananaLi2 .peel(1)
    
    (2) Output:
    BananaLi2 (20150108)
    BananaLi2 (20150109)
    BananaLi2 ()
    main:
    */
}
```

## 数组初始化(Array initialization)

``` Java
import java.util.Arrays;
public class SimpleArrayLi2 {
    public static void main(String[] args) {
        int[] array1 = {10, 11, 12, 13, 14,};   // 初始化列表最后一个逗号可写可不写
        //array1 = {10, 11,};  // Error: Array constants can only be used in initializers.
 
        int[] array2 = new int[]{20, 21, 22,};
        int[] array3 = new int[3];  // 基本类型的array，若未初始化，默认值为0
        int[] array4 = new int[3];
        array4[0] = 40;
        array4[1] = 41;
        array4[2] = 42;
        // array4[3] = 44; // Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 2 at line:11)
 
        Integer[] array5 = new Integer[3];  // object array，若未初始化，默认值为null
        Integer[] array6 = new Integer[] { new Integer(60), new Integer(61), 62,};
 
        System.out.println("array1.length = " + array1.length);
        // Arrays.toString()方法属于java.util标准类库，返回一个代表contens of array的字符串。
        System.out.println("array1 = " + Arrays.toString(array1));
        System.out.println("array2 = " + Arrays.toString(array2));
        System.out.println("array3 = " + Arrays.toString(array3));
        System.out.println("array4 = " + Arrays.toString(array4));
        System.out.println("array5 = " + Arrays.toString(array5));
        System.out.println("array6 = " + Arrays.toString(array6));
 
        array5 = array6;    // 数组名只是数组的引用，所以赋值的也只是引用。
        System.out.println("array5 = " + Arrays.toString(array5));
 
    } /* Output:
    array1.length = 5
    array1 = [10, 11, 12, 13, 14]
    array2 = [20, 21, 22]
    array3 = [0, 0, 0]
    array4 = [40, 41, 42]
    array5 = [null, null, null]
    array6 = [60, 61, 62]
    array5 = [60, 61, 62]
    */
}
```

可变参数列表 arivable argument lists, C语言称之为varargs. 适用于参数个数/类型未知的场合。

``` Java
public class NewVarArgs {
    static void printArray(Object...objects) {
        for (Object obj : objects) {
            System.out.print("[" + obj + "] ");
        }
        System.out.println();
    }
 
    public static void main(String[] args) {
        // 因为是可变参数，所以不必传递数组，编译器会将参数列表转为数组。
        printArray( new Integer(60), new Integer(61));
        printArray(47, 3.14f, 11.11);
        printArray("li2", "li22", "li222");
        // 当然仍可以传递数组，但编译器不会执行任何转换，所以要显示地转换，以移除编译器警告：
        // The argument of type Integer[] should explicitly be cast to Object[]
        // for the invocation of the varargs method printArray(Object...) from type NewVarArgs.
        // printArray(new Integer[]{1,2,3});
        printArray((Object[])new Integer[]{1,2,3});
        // 允许传递空的参数。
        printArray();
    }/* Output
    [60] [61]
    [47] [3.14] [11.11]
    [li2] [li22] [li222]
    [1] [2] [3]
    */
}
```


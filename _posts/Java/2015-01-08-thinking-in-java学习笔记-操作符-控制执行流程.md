---
layout: post
title: 『笔记』Thinking in Java - 操作符 控制执行流程
category: Java
tags: []
---


``` Java
import java.util.Date;
import java.util.Random;
 
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello world.");    // Hello world.
        System.out.println(new Date());    // Thu Jan 08 11:30:35 CST 2015
 
        Random rand = new Random(47);
        System.out.println(rand.nextInt(256));    // 186
        System.out.println(rand.nextInt(256));    // 102
 
        Integer n1 = new Integer(22);
        Integer n2 = new Integer(22);
        // 对象比较用equals, 基本类型比较用==,
        // 自定义类除非override equals(),否则不会得到预期结果.
        System.out.println(n1.equals(n2));    // true
 
        System.out.println("round 0.9 = " + Math.round(0.9f));     // round 0.9 = 1
        System.out.println("(int) 0.9 = " + (int)(0.9f));        // (int) 0.9 = 0
        System.out.println("round 1.9 = " + Math.round(1.9f));    // round 1.9 = 2
        System.out.println("(int) 1.9 = " + (int)(1.9f));        // (int) 1.9 = 1
    }
}
```

Java是怎么实现“所有数据类型在所有机器中的大小都是相同的”？ Java不需要sizeof().

 

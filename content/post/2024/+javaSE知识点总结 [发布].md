---
title: "JavaSE知识点总结+idea部分用法"
date: 2024-06-24T11:50:50+08:00
draft: true
tags: ["java"]
---

## 前言

​	从一些大佬的建议里了解了java目前的重要性，然后之前也是一直学了一点就放下了，所以现在决定完整的学完。该笔记基于[JavaSE核心知识点总结-CSDN博客](https://blog.csdn.net/ss810540895/article/details/124575074?spm=1001.2014.3001.5506)

## java学习路线

![img](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407182317170.png)

## java基础知识

### 一、标识符和关键字

1. 标识符
   1. 在java语言中，用来标志类名、对象名、变量名、方法名、类型名、数组名、包名的有效字符序列，称为“标识符”；
   2. 标识符由字母、数字、下划线、美元符号组成，且第一个字符不能是数字；
   3. java语言区分大小写；
   4. 标志符命名规则：类名首字母大写，变量名和方法名采用驼峰标志法，包名全小写，常量全大写，多个单词之间用“_”隔开；
2. 关键字
   1. 在java语言中，有一些专门的词汇已经被赋予了特殊的含义，不能再使用这些词汇来命名标识符，这些专有词汇，称为“关键字”；
   2. java有50个关键字和3个保留字，均不能用来命名标识符；
   3. true、false、null不是关键字，是保留字，但是仍不能用来命名标识符

### 二、数据类型

![img](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407182320261.png)

存储原理

1. 基本类型的存储原理：直接存储在内存中的内存栈上的，数据本身的值就是存储在栈空间里面，而Java语言里面八种数据类型是这种存储模型；
2. 引用类型的存储原理：按照存储对象的内存模型来进行数据存储的，“引用”是存储在有序的内存栈上的，而对象本身的值存储在内存堆上的；

### 三、运算符

![img](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407182325323.png)

### 四、流程控制语句

#### 1. 条件分支语句 （swtich...case...）

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        // 创建对象
        Scanner sc = new Scanner(System.in);
        System.out.println("输入选择：");
        // 多行输入
        int n = sc.nextInt();
        int[] arr = new int[n];
        switch (n) {
            case 1:
                System.out.println("ssr");
                break;
            case 2:
                System.out.println("sr");
                break;
            case 3:
                System.out.println("r");
                break;
        }
    }
}

// 输入选择：
// 3
// r
```

#### 2. 循环语句 （for / while）

```java
public class Main {
    public static void main(String[] args) {
        int sum = 0;
        int i = 0;
        while (i <= 100) {
            sum += i;
            i++;
        }
        System.out.println(sum);

        // for 循环
        for (int j = 0; j <= 100; j++){
            sum += j;
        }
        System.out.println(sum);

        //foreach 循环
        int [] array = {0,1,2,3,4,5};
        for (int j : array) {
            System.out.println(j);
        }
    }
}

// 5050
// 10100
// 0
// 1
// 2
// 3
// 4
// 5
```

## 数组

### 1. 概念

- 概念：相同数据类型的集合
- 作用：用于存储基本数据类型和引用数据类型的数据

### 2. 用法

```java
import jdk.internal.cmm.SystemResourcePressureImpl;

public class Main {
    public static void main(String[] args) {
        // 创建数组
        int[] array1 = new int [5];
        int[] array2 = {2, 5, 3, 1, 4};
        int[] array3 = new int[]{2, 5, 3, 1, 4};

        // 数组初始化
        array1[0] = 1;
        array1[1] = 7;
        array1[2] = 4;
        array1[3] = 9;
        array1[4] = 3;

        // 数组遍历
        // 1. for 循环打印数据
        for (int i = 0; i < array2.length; i++) {
            System.out.println(array2[i]);
        }

        // 2. foreach 打印数组
        for (int i : array3) {
            System.out.println(i);
        }

        // 数组排序
        // 1. 冒泡排序
        for (int i = 0; i < array1.length; i++) {
            for (int j = i+1; j < array1.length; j++) {
                if (array1[i] > array1[j]) {
                    int temp = array1[i];
                    array1[i] = array1[j];
                    array1[j] = temp;
                }
            }
        }
        for (int i : array1) {
            System.out.print(i);      // 打印的结果不带换行符
        }
        System.out.println();
        // 2. 选择排序
        for (int i = 0; i < array2.length; i++) {
            int min = i;
            for (int j = i; j < array2.length; j++) {
                if (array2[min] > array2[j]) {
                    min = j;
                }
            }
            int temp = array2[i];
            array2[i] = array2[min];
            array2[min] = temp;
        }

        for (int i : array2) {
            System.out.print(i);
        }
        System.out.println();
        // 3. 反转排序
        for (int i = 0; i < array2.length / 2; i++) {
             int temp = array2[i];
             array2[i] = array2[array2.length-1-i];
             array2[array2.length-1-i] = temp;
        }
        for (int i : array2) {
            System.out.print(i);
        }
        System.out.println();
        // 4. 插入排序
        for (int i = 0; i < array2.length; i++) {
            int j = i, temp = array1[i];
            for (; j > 0 && temp < array2[j-1]; j--) {
                array2[j] = array2[j-1];
            }
            array2[j] = temp;
        }
        for (int i : array2) {
            System.out.print(i);
        }
    }
}

//        2
//        5
//        3
//        1
//        4
//        2
//        5
//        3
//        1
//        4
//        13479
//        12345
//        54321
//        13479
```

### 3. arrays工具类



## idea补充用法

### 代码补全

#### 1. 初始化主函数

```
main+Enter
```

#### 2. 初始化for函数

```
变量.for+enter
变量.fori+enter
```


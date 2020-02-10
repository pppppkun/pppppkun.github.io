---
layout:     post
title:      Java注解
subtitle:   Java基础知识
date:       2020-02-10
author:     Pkun
header-img: img/Java-basic.jpg
catalog: true
tags:
    - Java
    - stream
---

## Stream

Java8中的新特性Stream能够将集合转化成一种叫做流的元素序列，通过声明的方法能够对集合中每个元素进行一系列并行或者串行的流水线操作。

这种函数式编程可以表达更多业务逻辑的意图，而不是实现机制，代码更简洁，可读性更高。

## 从一个简单的栗子开始

```java
List<Integer> list = new ArrayList<>();
for (int i = 0; i < 100; i++) 
{
    list.add(rd.nextInt(101));
}

System.out.println(list.stream().filter(t -> t >= 30 && t <= 70).count());
```

上面的栗子表明我们建立了一个100个元素的Arraylist，然后往里面填了100个随机数，最后输出了100个随机数中在$[30,70]$之间的数的个数。

如果用普通的循环语句，需要在循环体中不断修改count，大家自己写一写就会发现stream的高效和优美。

## API介绍

- stream()/parallelStream()
    - 最常用的方法，将集合转换成流，后者是并行流方法。
- filter(T -> boolean_exp)
    - 保留T中boolean_exp为true的元素
- collect(toList())
    - 可以将流再转会List
- distinct()
    - 去掉重复元素，记得提前重写类的equals方法
- sorted()/sorted((T,T)->int)
    - 如果流中的元素的类实现了 Comparable 接口，即有自己的排序规则，那么可以直接调用 sorted() 方法对元素进行排序，如 Stream<Integer>。反之, 需要调用 sorted((T, T) -> int) 实现 Comparator 接口
```java
list = list.stream()
           .sorted((p1, p2) -> p1.getAge() - p2.getAge())
           .collect(toList());
```
- limit(long n)/skip(long n)
    - 返回前n个元素和去掉前n个元素
- map(T -> R)
    - 将流中的每一个元素 T 映射为 R（类似类型转换）
- flatMap(T -> Stream<R>)
    - 将流中的每一个元素T映射成一个流，然后再把每一个流连接成为一个流
- anyMatch(T -> boolean_exp)/allMatch(T -> boolean_exp)/noneMatch(T -> boolean_exp)
    - 流中是否有一个元素/全部元素/没有元素匹配给定的boolean_exp条件
- findAny()
    - 找到其中一个元素 （使用 stream() 时找到的是第一个元素；使用 parallelStream() 并行时找到的是其中一个元素）
- findFirst()
    - 找到第一个元素

值得注意的是，这两个方法返回的是一个 Optional<T> 对象，它是一个容器类，能代表一个值存在或不存在，这个后面会讲到

- reduce((T,T)->T)/reduce(T,(T,T)->T)
    - 用于组合流中的元素，如求和，求积，求最大值等
```java
int sum = list.stream().map(Person::getAge).reduce(0, (a, b) -> a + b);
int sum = list.stream().map(Person::getAge).reduce(0, Integer::sum);
```
reduce 第一个参数 0 代表起始值为 0，lambda (a, b) -> a + b 即将两值相加产生一个新值
也可以
```java
Optional<Integer> sum = list.stream().map(Person::getAge).reduce(Integer::sum);
```

即不接受任何起始值，但因为没有初始值，需要考虑结果可能不存在的情况，因此返回的是 Optional 类型

- count()
    - 返回个数，long类型
- collect()
    - 收集方法，参数是一个收集器接口
- forEach()
```java
list.stream().forEach(System.out::println);
list.stream().forEach(PersonMapper::insertPerson);
```
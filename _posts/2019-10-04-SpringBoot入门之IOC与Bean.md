---
layout:     post
title:      Spring Boot入门之IOC与Bean
subtitle:   Spring Boot学习之旅
date:       2019-10-04
author:     Pkun
header-img: img/SpringBootITR.jpg
catalog: true
tags:
    - Java
    - Spring Boot
---

# Spring Boot 之 IOC 与 Bean


## IOC

IOC(`Inversion of Control`)，中文叫控制反转，它有两种实现，分别叫做DI(`Dependency Injection`)和DL(`Dependency Lookup`)，即依赖注入和依赖查找。

目前DL已经用的越来越少，因为他需要用户自己去使用API来操作对象，具有入侵性，DI是Spring框架使用的方法，容器负责组件的装配。

### 理解

控制反转可以分解成四个方面
- 谁控制谁？
- 反转是什么？
- 在哪里反转了？

让我们先看看IOC的定义
> 所谓 IOC ，就是由 Spring IOC 容器来负责对象的生命周期和对象之间的关系

让我们从一个简单的栗子入手，来回忆一下我们以前是怎么写代码的
```
public class Teacher
{

    private Homework homework;

    Teacher()
    {
        //检查作业
        homework = new Homework();
    }

    public void setHomework(Homework homework)
    {
        this.homework = homework;
    }

}
```

如果老师需要一个Homework来检查，一般采用直接创建的方式(`new Homework()`)，这个过程非常的繁琐，我们必须控制对象的每个环节，最后再销毁它，在这种情况下，我们不同的对象之间耦合度会比较高。

#### 思考
我们真的需要自己来创建一个新的对象吗？我们每次用到对象的时候，到底是需要这个***对象***，还是这个对象为我们提供的***服务***呢？

我们知道，其实我们所需要的是这个对象给我们提供的各种服务，只要在我们需要它的时候，它能够为我们提供我们想要的东西就行了。至于是我们自己创建还是别人拿给我们，这不重要。

这个给我们送对象的人，就是IOC，在上面的栗子中，相当于收作业的小组长，在你需要作业的时候，就把准备好的作业交到你的手上，我们只需要检查作业即可。

所以，简单的来说，IOC的理念就是***让别人为你服务***

在没有引入 IoC 的时候，**被注入的对象**直接依赖于**被依赖的对象**，有了 IoC 后，两者及其他们的关系都是通过 Ioc Service Provider 来统一管理维护的。被注入的对象需要什么，直接跟 IoC Service Provider 打声招呼，后者就会把相应的被依赖对象注入到被注入的对象中，从而达到 IOC Service Provider 为被注入对象服务的目的。

所以 IoC 就是这么简单！原来是需要什么东西自己去拿，现在是需要什么东西让别人（IOC Service Provider）送过来

再回到刚刚的四个问题，答案不言自明
- 谁控制谁？ IOC容器控制对象
- 反转是什么？ 主动创建对象是正向，IOC容器注入这个过程就是反转
- 在哪里反转了？ 所依赖的对象的获取反转了

哪么该如何将被依赖的对象注入到依赖对象中呢？

### 注入

注入的方法有3种
- 构造方式注入
- setter注入
- 接口注入

#### 构造方式注入

构造器注入，顾名思义就是被注入的对象通过在其构造方法中声明依赖对象的参数列表，让外部知道它需要哪些依赖对象。

#### setter方法注入

对于 JavaBean 对象而言，我们一般都是通过 getter 和 setter 方法来访问和设置对象的属性。所以，当前对象只需要为其所依赖的对象提供相对应的 setter 方法，就可以通过该方法将相应的依赖对象设置到被注入对象中。

#### 接口方式注入（不推荐）


## Spring Boot Bean
![Bean的生命周期](https://raw.githubusercontent.com/pppppkun/pppppkun.github.io/master/img/SpringOne.jpg)

Bean就是IOC所管理的Java对象。

### 装载Bean的三种方法

- 在xml中进行显式配置
- 在java中进行显式配置
- 隐式的bean发现机制和自动装配

### 在Java中进行显示配置

- `@ComponentScan`
- `@Bean`
- `@Import`

#### `@ComponentScan`

Spring容器会扫描`@ComponentScan`配置的包路径，找到标记`@Component`注解的类加入到Spring容器。

我们经常用到的类似的（注册到IOC容器）注解还有如下几个：

- @Configuration：配置类
- @Controller ：web控制器
- @Repository ：数据仓库
- @Service：业务逻辑

#### `@Bean`

注解`@Bean`被声明在方法上，方法都需要有一个返回类型，而这个类型就是注册到IOC容器的类型，接口和类都是可以的，介于面向接口原则，提倡返回类型为接口。

#### `@Import`

这种方法最为直接，直接把指定的类型注册到IOC容器里，成为一个java bean，可以把`@Import`放在程序的入口，它在程序启动时自动完成注册bean的过程。


使用Bean之前要声明你的使用`@Autowired`
---
layout:     post
title:      Spring Boot入门之启动原理解析
subtitle:   Spring Boot学习之旅
date:       2019-10-25
author:     Pkun
header-img: img/SpringBootITR.jpg
catalog: true
tags:
    - Spring Boot
    - Java
    - 注解
---

# 前言

讲了那么多跟SpringBoot相关的配置和插件，但是一直不太理解SpringBoot到底是如何启动的哈！当我们创建了一个SpringBoot项目后，在`Application.class`中自动帮我们配好了一个启动类
```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(AdviApplication.class, args);
    }

}
```
从这段代码中可以看出SpringBoot的全部精髓都在这个`@SpringBootApplication`注解里面了。

## `@SpringBootApplication`

点开这个注解，我们会发现`@SpringBootApplication`就像一个俄罗斯套娃一样里面还是一堆注解
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```

虽然使用了这么多注解，但是实际上比较重要的只有三个
- `@Configuration`
- `@EnableAutoConfiguration`
- `@ComponentScan`
所以，实际上我们把`@SpringBootApplication`替换成上面三个注解后也可以正常的使用启动类

### `@Configuration`

`@Configuration`其实是我们经常会使用的一个注解，它是JavaConfig形式的SpringIOC容器的配置类使用的那个注解，SpringBoot社区推荐使用基于JavaConfig的配置形式，所以这列启动类标注了`@Configuration`之后，本身其实也是一个IOC容器的配置类

任何一个标注了`@Configuration`的Java类定义都是一个JavaConfig配置类。

如果一个Bean的定义依赖其他Bean,则直接调用对应的JavaConfig类中依赖Bean的创建方法就可以了。

### `@ComponentScan`

`@ComponentScan`这个注解在Spring中很重要，它对应XML配置中的元素，`@ComponentScan`的功能其实就是自动扫描并加载符合条件的组件（比如`@Component`和`@Repository`等）或者bean定义，最终将这些Bean定义加载到IOC容器中。

我们可以通过basePackages等属性来细粒度的定制`@ComponentScan`自动扫描的范围，如果不指定，则默认Spring框架实现会从声明`@ComponentScan`所在类的package进行扫描。

> 注：所以SpringBoot的启动类最好是放在root package下，因为默认不指定basePackages。

### `EnableAutoConfiguration`

在Spring框架中有许多名字为`@Enable`开头的Annotation定义，比如
- `@EnableScheduling`
- `@EnableCaching`
- `@EnableMBeanExport`
- ...

`@EnableAutoConfiguration`的理念和做事方式就是借助`@Import`的支持，收集和注册特定场景相关的Bean定义。

>@EnableScheduling是通过@Import将Spring调度框架相关的bean定义都加载到IoC容器。
@EnableMBeanExport是通过@Import将JMX相关的bean定义加载到IoC容器。

而@EnableAutoConfiguration也是借助@Import的帮助，将所有符合自动配置条件的bean定义加载到IoC容器，仅此而已！

同样，`@EnableAutoConfiguration`也是一个复合的Annotaion
```java
@SuppressWarnings("deprecation")
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
    ...
}
```
其中，最关键的要属@Import(EnableAutoConfigurationImportSelector.class)，借助EnableAutoConfigurationImportSelector，@EnableAutoConfiguration可以帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IoC容器。就像一只“八爪鱼”一样

借助于Spring框架原有的一个工具类：SpringFactoriesLoader的支持，@EnableAutoConfiguration可以智能的自动配置功效才得以大功告成！

![EnableAutoConfiguration关键组件关系图](https://raw.githubusercontent.com/pppppkun/pppppkun.github.io/master/img/Springboot6-1.png)

#### 自动配置关键SpringFactoriesLoader

SpringFactoriesLoader属于Spring框架私有的一种扩展方案，其主要功能就是从指定的配置文件META-INF/spring.factories加载配置。

```java
public abstract class SpringFactoriesLoader {
    //...
    public static <T> List<T> loadFactories(Class<T> factoryClass, ClassLoader classLoader) {
        ...
    }


    public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
        ....
    }
}
```

配合@EnableAutoConfiguration使用的话，它更多是提供一种配置查找的功能支持，即根据@EnableAutoConfiguration的完整类名org.springframework.boot.autoconfigure.EnableAutoConfiguration作为查找的Key,获取对应的一组@Configuration类


上图就是从SpringBoot的autoconfigure依赖包中的META-INF/spring.factories配置文件中摘录的一段内容，可以很好地说明问题。

![EnableAutoConfiguration关键组件关系图](https://raw.githubusercontent.com/pppppkun/pppppkun.github.io/master/img/Springboot6-2.png)

所以，@EnableAutoConfiguration自动配置的魔法骑士就变成了：从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中org.springframework.boot.autoconfigure.EnableutoConfiguration对应的配置项通过反射（Java Refletion）实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。


### 全过程
SpringApplication的run方法的实现是我们本次旅程的主要线路，该方法的主要流程大体可以归纳如下：

- 1） 如果我们使用的是SpringApplication的静态run方法，那么，这个方法里面首先要创建一个SpringApplication对象实例，然后调用这个创建好的SpringApplication的实例方法。在SpringApplication实例初始化的时候，它会提前做几件事情：
    - 根据classpath里面是否存在某个特征类（org.springframework.web.context.ConfigurableWebApplicationContext）来决定是否应该创建一个为Web应用使用的ApplicationContext类型。
    - 使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationContextInitializer。
    - 使用SpringFactoriesLoader在应用的classpath中查找并加载所有可用的ApplicationListener。
    - 推断并设置main方法的定义类。

- 2） SpringApplication实例初始化完成并且完成设置后，就开始执行run方法的逻辑了，方法执行伊始，首先遍历执行所有通过SpringFactoriesLoader可以查找到并加载的SpringApplicationRunListener。调用它们的started()方法，告诉这些SpringApplicationRunListener，“嘿，SpringBoot应用要开始执行咯！”。

- 3） 创建并配置当前Spring Boot应用将要使用的Environment（包括配置要使用的PropertySource以及Profile）。

- 4） 遍历调用所有SpringApplicationRunListener的environmentPrepared()的方法，告诉他们：“当前SpringBoot应用使用的Environment准备好了咯！”。

- 5） 如果SpringApplication的showBanner属性被设置为true，则打印banner。

- 6） 根据用户是否明确设置了applicationContextClass类型以及初始化阶段的推断结果，决定该为当前SpringBoot应用创建什么类型的ApplicationContext并创建完成，然后根据条件决定是否添加ShutdownHook，决定是否使用自定义的BeanNameGenerator，决定是否使用自定义的ResourceLoader，当然，最重要的，将之前准备好的Environment设置给创建好的ApplicationContext使用。

- 7） ApplicationContext创建好之后，SpringApplication会再次借助Spring-FactoriesLoader，查找并加载classpath中所有可用的ApplicationContext-Initializer，然后遍历调用这些ApplicationContextInitializer的initialize（applicationContext）方法来对已经创建好的ApplicationContext进行进一步的处理。

- 8） 遍历调用所有SpringApplicationRunListener的contextPrepared()方法。

- 9） 最核心的一步，将之前通过@EnableAutoConfiguration获取的所有配置以及其他形式的IoC容器配置加载到已经准备完毕的ApplicationContext。

- 10） 遍历调用所有SpringApplicationRunListener的contextLoaded()方法。

- 11） 调用ApplicationContext的refresh()方法，完成IoC容器可用的最后一道工序。

- 12） 查找当前ApplicationContext中是否注册有CommandLineRunner，如果有，则遍历执行它们。

- 13） 正常情况下，遍历执行SpringApplicationRunListener的finished()方法、（如果整个过程出现异常，则依然调用所有SpringApplicationRunListener的finished()方法，只不过这种情况下会将异常信息一并传入处理）

![EnableAutoConfiguration关键组件关系图](https://raw.githubusercontent.com/pppppkun/pppppkun.github.io/master/img/Springboot6-3.png)


## 总结

到此，SpringBoot的核心组件完成了基本的解析，综合来看，大部分都是Spring框架背后的一些概念和实践方式，SpringBoot只是在这些概念和实践上对特定的场景事先进行了固化和升华，而也恰恰是这些固化让我们开发基于Sping框架的应用更加方便高效。



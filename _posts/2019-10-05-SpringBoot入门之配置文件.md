---
layout:     post
title:      Spring Boot入门之配置文件
subtitle:   Spring Boot学习之旅
date:       2019-10-04
author:     Pkun
header-img: img/Springboottwo.jpg
catalog: true
tags:
    - Java
    - Spring Boot
---


# Spring Boot 与 配置文件(application.properties)

## 前言

上一篇介绍了Spring Boot的核心概念IOC和IOC容器中的bean，这一篇讲讲Spring Boot的另一个特点，即“习惯优于配置”。

## 正文

Spring Boot的配置文件是`application.properties`，在初始化完一个Spring项目后就可以看到了，一般在目录`resources`里面，Spring Boot的全局配置文件的作用一般是对一些默认配置的值进行修改。

### 自定义属性

我们可以将一些常量写在里面

```
pkun.name = "pkun"
pkun.want = "make a web"
```

然后再需要使用的地方通过注解`@Value`绑定一下就可以了

```
@RestController
public class UserController {

    @Value("${pkun.name}")
    private  String name;
    @Value("${pkun.want}")
    private  String want;

    @RequestMapping("/")
    public String hexo(){
        return name+want;
    }
}
```

然后我们在`localhost:8080`就可以看到`"pkun""make a web"`了。

有时候常量太多，一个个用`@Value`绑定到属性字段上太麻烦了，官方提倡绑定一个对象的bean，这里我们建一个`ConfigBean.java`类，然后在顶部使用注解`@ConfigurationProperties(prefix = "pkun")`来指明使用哪个常量

配置完后还需要在Spring Boot的入口类处或者`ConfigBean`类名前面加上`@EnableConfigurationProperties({ConfigBean.class})`，指明要加载哪个Bean

最后根据上一节知识，在`UserController`中添加
```
@Autowired
ConfigBean configBean;
```
即可

### 参数间引用
在`application.properties`中的各个参数之间也可以相互引用
```
pkun.name = "pkun"
pkun.want = "make a web"
pkun.conn = ${pkun.name} + ${pkun.want};
```

### 使用自定义配置文件

可以在`resources`目录下面新建一个`test.properties`，然后如`application.properties`和上面所说的在里面设置一下常量，然后绑定到某个对象的Bean中，最后再加上注解`@PropertySource`即可

注：作者自己使用的时候从未成功过。


### 随机值配置
在配置文件中可以通过`${random}`来生成各种不同类型的随机值。

```
pkun.age = ${random.int}
```

### 外部配置

Spring Boot 是基于Jar包运行的，打包成Jar的包可以直接通过下面命令运行
```
java -jar xx.jar
```
也可以通过下面的指令修改tomcat端口号
```
java -jar xx.jar --server.port = 9090
```

在命令行中，两个连续的减号`--`表示对`application.properties`中的属性进行赋值，如果你害怕命令行有风险，可以使用`SpringApplication.setAddCommandLineProperties(false)`来禁用它


实际上，Spring Boot应用程序有多种设置途径，能从多重属性源获得属性，包括如下几种:
>- 根目录下的开发工具全局设置属性(当开发工具激活时为`~/.spring-boot-devtools.properties`)。
>- 测试中的`@TestPropertySource`注解。
>- 测试中的`@SpringBootTest#properties`注解特性。
>- 命令行参数
>- `SPRING_APPLICATION_JSON`中的属性(环境变量或系统属性中的内联JSON嵌入)。
>- `ServletConfig`初始化参数。
>- `ServletContext`初始化参数。
>- `java:comp/env`里的`JNDI`属性
>- `JVM`系统属性
>- 操作系统环境变量
>- 随机生成的带`random.*` 前缀的属性（在设置其他属性时，可以应用他们，比如`${random.long}`）
>- 应用程序以外的`application.properties`或者`appliaction.yml`文件
>- 打包在应用程序内的`application.properties`或者`appliaction.yml`文件
>- 通过`@PropertySource`标注的属性源
>- 默认属性(通过`SpringApplication.setDefaultProperties`指定).

这里列表按组优先级排序，也就是说，任何在高优先级属性源里设置的属性都会覆盖低优先级的相同属性，列如我们上面提到的命令行属性就覆盖了application.properties的属性。

### 多环境配置

当应用程序需要部署到不同运行环境时，一些配置细节通常会有所不同，最简单的比如日志，生产日志会将日志级别设置为WARN或更高级别，并将日志写入日志文件，而开发的时候需要日志级别为DEBUG，日志输出到控制台即可。
如果按照以前的做法，就是每次发布的时候替换掉配置文件，这样太麻烦了，Spring Boot的Profile就给我们提供了解决方案，命令带上参数就搞定。

这里我们来模拟一下，只是简单的修改端口来测试
在Spring Boot中多环境配置文件名需要满足`application-{profile}.properties`的格式，其中`{profile}`对应你的环境标识，比如：

>- `application-dev.properties`：开发环境
>- `application-prod.properties`：生产环境

想要使用对应的环境，只需要在`application.properties`中使用`spring.profiles.active`属性来设置，值对应上面提到的`{profile}`，这里就是指dev、prod这2个。
当然你也可以用命令行启动的时候带上参数：`java -jar xxx.jar --spring.profiles.active=dev`

除了可以用profile的配置文件来分区配置我们的环境变量，在代码里，我们还可以直接用`@Profile`注解来进行配置，例如数据库配置，这里我们先定义一个接口`public  interface DBConnector { public  void  configure(); }`

分别定义俩个实现类来实现它
```
/**
  * 测试数据库
  */
@Component
@Profile("testdb")
public class TestDBConnector implements DBConnector {
    @Override
    public void configure() {
        System.out.println("testdb");
    }
}
/**
 * 生产数据库
 */
@Component
@Profile("devdb")
public class DevDBConnector implements DBConnector {
    @Override
    public void configure() {
        System.out.println("devdb");
    }
}
```
通过配置文件选择激活那个类`spring.profiles.active=testdb`后就可以用了

```
@RestController
@RequestMapping("/task")
public class TaskController {

    @Autowired DBConnector connector ;

    @RequestMapping(value = {"/",""})
    public String hellTask(){

        connector.configure(); //最终打印testdb     
        return "hello task !! myage is " + myage;
    }
}
```

`spring.profiles.include`可以用来叠加profile
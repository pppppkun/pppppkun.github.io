---
layout:     post
title:      Spring Boot入门之Swagger2配置
subtitle:   Spring Boot学习之旅
date:       2019-10-13
author:     Pkun
header-img: img/SpringBootITR.jpg
catalog: true
tags:
    - Spring Boot
    - Java
    - Swagger2
---


# 前言

现如今，前后端分离已经逐渐成为互联网项目一种标准的开发方式，前端与后端交给不同的人员开发，

但是项目开发中的沟通成本也随之升高，这部分沟通成本主要在于前端开发人员与后端开发人员对WebAPI接口的沟通，Swagger2 就可以很好地解决，它可以动态生成Api接口文档，降低沟通成本，促进项目高效开发。

## 第零步 在pom.xml中添加依赖

```xml
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
```

## 第二步 添加配置类

在项目中新建一个目录`config`，并在下面新建一个`SwaggerConfig`类，我们在将在里面进行配置
```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket customDocket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("包名"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        Contact contact = new Contact("团队名", "www.my.com", "my@my.com");
        return new ApiInfoBuilder()
                .title("文档标题")
                .description("文档描述")
                .contact(contact)   // 联系方式
                .version("1.1.0")  // 版本
                .build();
    }
}
```
常规套路 不解释

## 第三步 使用Swagger2注解

以UserController为例
```java
package ns.mental.advi.controller;


import io.swagger.annotations.*;
import ns.mental.advi.entity.UserEntity;
import ns.mental.advi.service.impl.UserServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
@RequestMapping("/testBoot")
public class UserController {

    @Autowired
    private UserServiceImpl userService;


    @ApiOperation(value = "查询用户", response = UserEntity.class, httpMethod = "GET" ,notes = "返回一个User对象")
    @RequestMapping("getUser/{id}")
    public String GetUser(@PathVariable int id){
        return userService.Sel(id).toString();
    }
}
```
在这里讲一些常用的Swagger2注解

- `@Api` 用于注解类，标识这个类为Swagger的资源
    - 常用的参数有Value 和 Tags，都是说明的作用，但是value可以用Tags替代，当有多个Tags的时候会生成多个list
- `@ApiOperation()` 用于注解方法
    - 常用的参数有value(描述方法),notes(提示内容),response(返回对象),httpMethod(具体是什么请求)
- `@ApiParam()` 用于方法参数等的说明，表示对参数的添加元数据，不用的话会自动检索代码中的信息
    - name(参数名),value(参数说明),required(是否必填)
- `@ApiModel()`用于类 ；表示对类进行说明，用于参数用实体类接收 
    - value(表示对象名),description(描述),都可省略 
- `@ApiModelProperty()`用于方法，字段； 表示对model属性的说明或者数据操作更改 
    - value(字段说明),name(重写属性名字),dataType(重写属性类型),required(是否必填),example(举例说明),hidden(隐藏)
- `@ApiImplicitParam()`用于方法 表示单独的请求参数 
- `@ApiImplicitParams()`用于方法，包含多个 @ApiImplicitParam 
    - name(参数名字),value(参数说明),dataType(数据类型) ,paramType(参数类型) ,example(举例说明)


## 最后

运行项目，然后打开 http://localhost:8080/swagger-ui.html 就可以看到你的API了
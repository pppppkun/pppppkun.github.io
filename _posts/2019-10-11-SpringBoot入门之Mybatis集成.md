---
layout:     post
title:      Spring Boot入门之Mybatis集成
subtitle:   Spring Boot学习之旅
date:       2019-10-11
author:     Pkun
header-img: img/SpringBootITR.jpg
catalog: true
tags:
    - Spring Boot
    - Java
    - 数据库
---

# 介绍

>MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github。
iBATIS一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。iBATIS提供的持久层框架包括SQL Maps和Data Access Objects（DAOs）

>MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Ordinary Java Object,普通的 Java对象)映射成数据库中的记录。 

说了这么多，反正MyBatis很吊就是了！！

但是如果要使用MyBatis，首先你要有一些数据库和JDBC的基础才行，相关内容在这里不做介绍了

本文中使用的数据库是MySQL，版本是8.0.16，MyBatis的版本是2.1.0

## 第零步 MySQL 建库建表

本文中database的名字是springboot

```SQL

create database springboot

CREATE TABLE `user` (
  `id` int(32) NOT NULL AUTO_INCREMENT,
  `userName` varchar(32) NOT NULL,
  `passWord` varchar(50) NOT NULL,
  `realName` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

```


## 第一步 创建Spring Boot项目

[Spring Initializr](https://start.spring.io/)

相信都看到这篇博客了，你应该已经会初始化项目了吧？？

我们在初始化的依赖中添加
`Lombok`，`Spring Web`，`JDBC API`，`MySQL Driver`，`MyBatis Framework`
然后就可以愉快的import我们的项目了。

## 第二步，修改项目结构

作为一个Demo，我们只需要最小限度支持测试的文件就行了。
`controller.UserController.class`

`dao.UserMapper.class`

`entity.UserEntity.class`

`service.UserService.class`

`service.impl.UserServiceImpl.class`

看名字就知道这几个类是干啥的了，你都看到这篇博客了，应该也不用我在解释了吧！

## 第二步，设置application

我用的是`application.yaml`，因为他提供了缩进，看起来舒服一点，但是如果你有`.properties`文件的话，它会覆盖`.yaml`文件，这点需要注意。

在设置的时候，我们首先配置`JDBC`
```xml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/<database_name>?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
    username: ****
    password: ****
    driver-class-name: com.mysql.cj.jdbc.Driver
```
`spring`下还有一些别的配置，比如`max-file-size`等，可以自己去了解

然后我们配置mybatis的环境
```xml
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: ns.mental.advi.entity
```
第一句表示我们的SQL语句都写在同目录的mapper文件夹下，且都是.xml格式。第二句我们等下再解释！

## 第三步，写.xml文件

我们在`resources`下建立一个文件夹`mapper`，然后在里面新建一个`UserEntityMapper.xml`文件

内容大概这样
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="ns.mental.advi.dao.UserMapper">
    <resultMap id="UserResultMap" type="ns.mental.advi.entity.UserEntity">
        <result column="id" jdbcType="INTEGER" property="id" />
        <result column="userName" jdbcType="VARCHAR" property="userName" />
        <result column="passWord" jdbcType="VARCHAR" property="passWord" />
        <result column="realName" jdbcType="VARCHAR" property="realName" />
    </resultMap>


    <select id="Sel" parameterType="java.lang.Integer" resultMap="UserResultMap">

        select * from user where id = #{id}

    </select>

    
</mapper>
```

我们来一个个解释

`<mapper namespace="ns.mental.advi.dao.UserMapper">`
namespace是指将查询语句映射到哪个类中，这个类中要有和sql语句同名的方法。

`<resultMap id = "UserResultMap" type="ns.mental.advi.entity.UserEntity">`

`id`就是id, `type`是sql映射文件中定义的返回值类型
`resultMap`是一个Mybatis最强大的元素，它可以将查询到的复杂数据(比如几个表中的数据)全部映射到type上
`<result column="" jdbcType="" property=""/>`中，`property`必须是type指向的pojo对象的一个属性。
同时我们也可以设置`<association property="" javaType="" >`，其中`property`是pojo对象的一个属性，`javaType`是pojo关联的java对象。这部分内容将不会在本文中介绍。

还有`<collection>`这个暂时不介绍（其实是我不会）

`<select id="Sel" parameterType="java.lang.Integer" resultMap="UserResultMap"></select>` 就是我们的查询语句了，在UserMapper中我们要使用同样的ID。
`parameterType`是方法中传入的参数的类型，resultMap是上面那个，赋予了返回值的类型。


### 回溯

回过头来看刚刚的`type-aliases-package: ns.mental.advi.entity`

不难理解他的意思是限制了resultMap的type，这样我们将mabatis自动扫描到POJO上。（我知道你们可能没听懂）

## 写码！

`UserMapper`
```java
package ns.mental.advi.dao;

import ns.mental.advi.entity.UserEntity;
import org.springframework.stereotype.Repository;

@Repository
public interface UserMapper {


    UserEntity Sel(int id);

}

```

`UserService.class`

```java
package ns.mental.advi.service;

import ns.mental.advi.entity.UserEntity;

public interface UserService {

    public UserEntity Sel(int id);
}

```

`UserServiceImpl.class`

```java
package ns.mental.advi.service.impl;

import ns.mental.advi.dao.UserMapper;
import ns.mental.advi.entity.UserEntity;
import ns.mental.advi.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;


@Service
public class UserServiceImpl implements UserService {

    @Autowired
    UserMapper userMapper;

    public UserEntity Sel(int id){
        return userMapper.Sel(id);
    }
}
```

`UserEntity`
```java
package ns.mental.advi.entity;

import lombok.Data;


@Data
public class UserEntity {

    private Integer id;
    private String userName;
    private String passWord;
    private String realName;

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", userName='" + userName + '\'' +
                ", passWord='" + passWord + '\'' +
                ", realName='" + realName + '\'' +
                '}';
    }

}

```

`UserController`

```java
package ns.mental.advi.controller;


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

    @RequestMapping("getUser/{id}")
    public String GetUser(@PathVariable int id){
        return userService.Sel(id).toString();
    }
}
```

然后就可以启动项目啦!

输入网址 http://localhost:8080/testBoot/getUser/4 就可以看到数据啦！

## 总结

用过JDBC的同学一定都能感觉到MyBatis的强大（如果你配置成功了的话），整合好Mybatis之后前后端的交互就很简单啦，提供一个url然后返回一个数据即可

下一篇博客将会介绍Swagger2的运用，他是一个提供图形化接口的界面
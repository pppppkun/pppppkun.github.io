---
layout:     post
title:      Spring Boot入门之banner
subtitle:   Spring Boot学习之旅
date:       2019-10-08
author:     Pkun
header-img: img/SpringBootITR.jpg
catalog: true
tags:
    - Java
    - Spring Boot
---


# banner

大家在启动Spring boot的项目的时候，会看到控制台打印了一个SpringBoot的字符串出来，这个字符串是可以被修改的。

对于一个喜欢整活的程序⚪来说，不自己DIY一些臊皮的话就太没有意思啦！！！

## 如何设置？

根据上一节的内容，我们可以推断要修改这个SpringBoot是需要在`application`配置文件附近设置的

so！我们只要在resources设置一个`banner.txt`就可以了！

至于为什么？当然是我查的

## 如何定制`banner`

http://patorjk.com/software/taag

http://www.network-science.de/ascii/

http://www.degraeve.com/img2txt.php

 ## `banner`中的配置

```
${AnsiColor.BRIGHT_RED}：设置控制台中输出内容的颜色

${application.version}：用来获取MANIFEST.MF文件中的版本号

${application.formatted-version}：格式化后的${application.version}版本信息

${spring-boot.version}：Spring Boot的版本号

${spring-boot.formatted-version}：格式化后的${spring-boot.version}版本信息
```

这些配置都来源于`springboot`参考手册：
[Common application properties](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#common-application-properties)
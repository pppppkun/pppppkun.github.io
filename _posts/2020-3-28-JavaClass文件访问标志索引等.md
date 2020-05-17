---
layout:     post
title:      Java .class文件结构与常量池
subtitle:   深入理解Java虚拟机
date:       2020-03-26
author:     Pkun
header-img: 
catalog: true
tags: Java虚拟机
---

### 访问标志

在常量池结束之后紧接着的两个字符代表访问标志，这个标志用于识别一些类或者接口的访问信息，比如是个类还是一个接口，是否是public等

![\begin{array}[b]{|c|c|}  \hline 标志名称& 标志值&含义&针对的对像\\  \hline \text{ACC_PUBLIC}&0x0001&\text{是否为public类型}&所有类型\\ \hline \text{ACC_FINAL}& 0x0010&\text{是否为声明为final} &类 \\ \hline \text{ACC_SUPER}&0x0020 &\text{使用新的invokespecial语义} &类和接口 \\ \hline \text{ACC_INTERFACE}&0x0200 &\text{标识是一个接口类型} & 接口\\ \hline \text{ACC_ABSTRACT}&0x0400 &\text{是否为抽象类型} &{接口或抽象类，此标识符为真，其他类型为假} \\ \hline \text{ACC_SYNTHETIC}&0x1000 &\text{该类不由用户代码生成} &所有类型 \\ \hline \text{ACC_ANNOTATION}& 0x2000&\text{注解类型} & 注解\\ \hline \text{ACC_ENUM}&0x4000 &\text{枚举类型} & 枚举\\ \hline \text{ACC_MODULE}&0x8000&\text{模块类型}  \\ \hline  \end{array}](https://www.zhihu.com/equation?tex=%5Cbegin%7Barray%7D%5Bb%5D%7B%7Cc%7Cc%7C%7D%20%20%5Chline%20%E6%A0%87%E5%BF%97%E5%90%8D%E7%A7%B0%26%20%E6%A0%87%E5%BF%97%E5%80%BC%26%E5%90%AB%E4%B9%89%26%E9%92%88%E5%AF%B9%E7%9A%84%E5%AF%B9%E5%83%8F%5C%5C%20%20%5Chline%20%5Ctext%7BACC_PUBLIC%7D%260x0001%26%5Ctext%7B%E6%98%AF%E5%90%A6%E4%B8%BApublic%E7%B1%BB%E5%9E%8B%7D%26%E6%89%80%E6%9C%89%E7%B1%BB%E5%9E%8B%5C%5C%20%5Chline%20%5Ctext%7BACC_FINAL%7D%26%200x0010%26%5Ctext%7B%E6%98%AF%E5%90%A6%E4%B8%BA%E5%A3%B0%E6%98%8E%E4%B8%BAfinal%7D%20%26%E7%B1%BB%20%5C%5C%20%5Chline%20%5Ctext%7BACC_SUPER%7D%260x0020%20%26%5Ctext%7B%E4%BD%BF%E7%94%A8%E6%96%B0%E7%9A%84invokespecial%E8%AF%AD%E4%B9%89%7D%20%26%E7%B1%BB%E5%92%8C%E6%8E%A5%E5%8F%A3%20%5C%5C%20%5Chline%20%5Ctext%7BACC_INTERFACE%7D%260x0200%20%26%5Ctext%7B%E6%A0%87%E8%AF%86%E6%98%AF%E4%B8%80%E4%B8%AA%E6%8E%A5%E5%8F%A3%E7%B1%BB%E5%9E%8B%7D%20%26%20%E6%8E%A5%E5%8F%A3%5C%5C%20%5Chline%20%5Ctext%7BACC_ABSTRACT%7D%260x0400%20%26%5Ctext%7B%E6%98%AF%E5%90%A6%E4%B8%BA%E6%8A%BD%E8%B1%A1%E7%B1%BB%E5%9E%8B%7D%20%26%7B%E6%8E%A5%E5%8F%A3%E6%88%96%E6%8A%BD%E8%B1%A1%E7%B1%BB%EF%BC%8C%E6%AD%A4%E6%A0%87%E8%AF%86%E7%AC%A6%E4%B8%BA%E7%9C%9F%EF%BC%8C%E5%85%B6%E4%BB%96%E7%B1%BB%E5%9E%8B%E4%B8%BA%E5%81%87%7D%20%5C%5C%20%5Chline%20%5Ctext%7BACC_SYNTHETIC%7D%260x1000%20%26%5Ctext%7B%E8%AF%A5%E7%B1%BB%E4%B8%8D%E7%94%B1%E7%94%A8%E6%88%B7%E4%BB%A3%E7%A0%81%E7%94%9F%E6%88%90%7D%20%26%E6%89%80%E6%9C%89%E7%B1%BB%E5%9E%8B%20%5C%5C%20%5Chline%20%5Ctext%7BACC_ANNOTATION%7D%26%200x2000%26%5Ctext%7B%E6%B3%A8%E8%A7%A3%E7%B1%BB%E5%9E%8B%7D%20%26%20%E6%B3%A8%E8%A7%A3%5C%5C%20%5Chline%20%5Ctext%7BACC_ENUM%7D%260x4000%20%26%5Ctext%7B%E6%9E%9A%E4%B8%BE%E7%B1%BB%E5%9E%8B%7D%20%26%20%E6%9E%9A%E4%B8%BE%5C%5C%20%5Chline%20%5Ctext%7BACC_MODULE%7D%260x8000%26%5Ctext%7B%E6%A8%A1%E5%9D%97%E7%B1%BB%E5%9E%8B%7D%20%20%5C%5C%20%5Chline%20%20%5Cend%7Barray%7D)\begin{array}[b]{|c|c|}  \hline 标志名称& 标志值&含义&针对的对像\\  \hline \text{ACC_PUBLIC}&0x0001&\text{是否为public类型}&所有类型\\ \hline \text{ACC_FINAL}& 0x0010&\text{是否为声明为final} &类 \\ \hline \text{ACC_SUPER}&0x0020 &\text{使用新的invokespecial语义} &类和接口 \\ \hline \text{ACC_INTERFACE}&0x0200 &\text{标识是一个接口类型} & 接口\\ \hline \text{ACC_ABSTRACT}&0x0400 &\text{是否为抽象类型} &{接口或抽象类，此标识符为真，其他类型为假} \\ \hline \text{ACC_SYNTHETIC}&0x1000 &\text{该类不由用户代码生成} &所有类型 \\ \hline \text{ACC_ANNOTATION}& 0x2000&\text{注解类型} & 注解\\ \hline \text{ACC_ENUM}&0x4000 &\text{枚举类型} & 枚举\\ \hline \text{ACC_MODULE}&0x8000&\text{模块类型}  \\ \hline  \end{array} 

两个字节一共有16个标志位可以使用，目前只定义了9个，没有标志位的时候要按照0处理，按照上一接的内容进行查找可以发现访问标志是0x0021，能够读出来是0x0001 | 0x0020，所以他对应的是public + class，和我们的代码相吻合

### 类索引、父类索引、接口索引集合

类索引和父类索引都是u2类型的数据，接口索引集合是一组u2类型的集合，class文件通过这三项来确定该类型的继承关系。

类索引用来确定这个类的全限定名，父类索引用来确定这个类的父类的全限定名，注意到所有的类都至少有一个父类java.lang,Object，所以除了这个类以外所有的类的父类索引都不是0

接口索引用来描述这个类实现了哪些接口，这些接口按照implements关键字后的接口顺序从左到右排列在接口索引集合中。

类索引、父类索引和接口索引集合都按顺序排列在访问标志之后，类索引和父类索引用两个u2类型的索引值表示，它们各自指向一个类型为CONSTANT_Class_info的类描述符常量，通过CONSTANT_Class_info类型的常量中的索引值可以找到定义在CONSTANT_Utf8_info类型的常量中的全限定名字符串。

![img](https://pic2.zhimg.com/v2-388821a853cabfefda0be0d92b6584a9_b.png)

还是同样的.class文件，我们可以读到访问标志后面的3个索引为3，4，0，其中0表示没有接口

![img](https://pic3.zhimg.com/v2-017c749b3fab471ff5391f2adf68895e_b.png)

然后我们可以找到对应的第3和第4个常量，两个class，这两个class又可以继续索引到第十七和第十八个常量，正好是我们的类和父类的名字

### 字段表集合

字段表用于描述接口或者类中声明的变量。Java语言中的字段包括类级变量以及实例级变量（类变量也叫静态变量，也就是在变量前加了static 的变量。实例变量也叫对象变量，即没加static 的变量），但是不包括在方法内部声明的变量。

对一个字段，它的描述符包括作用域，实例变量还是类变量，可变性(final)，并发可见性(volatile)，可否被序列化(transient)等等

>  上述这些信息中，各个修饰符都是布尔值，要么有某个修饰符，要么没有，很适合使用标志位来表示。而字段叫做什么名字、字段被定义为什么数据类型，这些都是无法固定的，只能引用常量池中的常量来描述。 

![img](https://pic1.zhimg.com/v2-63b93ec64c08c32e9e0832bfdff41738_b.png)

字段表结构

跟随access_flags标志的是name_index和descriptor_index，它们都是简单的对常量池的引用，代表它的名字和描述符。

>  “org/fenixsoft/clazz/TestClass”是这个类的全限定名，仅仅是把类全名中的“.”替换成了“/”而已，为了使连续的多个全限定名之间不产生混淆，在使用时最后一般会加入一个“；”号表示全限定名结束。简单名称则就是指没有类型和参数修饰的方法或者字段名称，这个类中的inc()方法和m字段的简单名称分别就是“inc”和“m”。 

描述符的作用是用来描述字段的数据类型、方法的参数列表（包括数量、类型以及顺序）和返回值。根据描述符规则，基本数据类型以及代表无返回值的void类型都用一个大写字符来表示，而对象类型则用字符L加对象的全限定名来表示。

![img](https://pic3.zhimg.com/v2-33068b256860fa0ff44611f83f4c1bd2_b.png)

描述符标识字符含义

对于数组类型，java.lang.String[][]会被记录成[[Ljava/lang/String;，int[]将会被记录成[I

用描述符来描述方法的时候，先参数列表后返回值，参数列表按照参数的顺序放在一组小括号()只中，比如方法int indexOf(char[]source，int sourceOffset，int sourceCount，char[]target，int targetOffset，int targetCount，int fromIndex)的描述符为([CII[CIII)I

![img](https://pic2.zhimg.com/v2-8a98c535cce6e782a4ef13364813834d_b.png)

这几个字节表示了

```
01 -> 1个字段
02 -> access_flags -> private
05 -> name_index -> 常量池第五个，m
06 -> descriptor_index -> 字符描述符 ,指向第六个常量，I，表示是int型
```

### **方法表集合**

Class文件存储格式中方法的描述对字段的描述采用了几乎完全一致的方式，方法表的结构如同字段表一样。

![img](https://pic2.zhimg.com/v2-28e99d1f2f902e13428c9ef3d1542cdd_b.png)

注意到一件事情，那就是到现在我们都没有提方法里面的代码去哪里了，实际上Javac编译，翻译成字节码指令之后，代码都被存访在方法属性表集合中一个名为code的属性里面。

![img](https://pic1.zhimg.com/v2-57f91f30c2f83edcd7052fbac9477244_b.png)

从我们的class文件中可以分析出两个方法，一个是<init>，另一个是()V 

### 参考

- [1] 深入理解Java虚拟机 周志明
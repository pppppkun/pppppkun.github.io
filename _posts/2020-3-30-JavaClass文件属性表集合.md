---
layout:     post
title:      Java .class文件属性表集合
subtitle:   深入理解Java虚拟机
date:       2020-03-30
author:     Pkun
header-img: 
catalog: true
tags: Java虚拟机
---

### 属性表集合

Class文件，字段表和方法表都可以拥有自己的属性表集合

属性表集合不再要求各个属性表具有严格的顺序，同时只要不重复，可以在编译的时候向属性表中写入自己定义的属性信息。

对于每一个属性，它的名称都要从常量池中引入一个CONSTANT_UTF8_info类型的常量来表示，而属性表的结构则是完全自定义的，只需要通过一个u4的长度属性去说明属性值所占用的位数即可。

![img](https://pic2.zhimg.com/v2-1590171ac7d672077e18ee7ebd0fc819_b.png)

接下来讲解几个比较重要的属性，所有属性表会在最后面贴出来

### code属性

code属性出现在方法表的属性集合只中，但是并非所有的方法都必须存在这个属性，接口或者抽象类中的方法就不存在code属性。code属性的结构如下

![img](https://pic4.zhimg.com/v2-1925111d219485876a271ba58c17debb_b.png)

max_stack代表了操作数栈深度的最大值，虚拟机会根据这个值来分配栈帧中的操栈深度，max_locals代表了局部变量所需的存储空间，单位是变量槽slot，是虚拟机为局部变量分配内存所使用的最小单位，对于长度不超过32位的数据类型，每个局部变量占一个变量槽，64位的占两个变量槽。方法参数（包括this），显示异常处理程序的参数（try-catch），方法体中定义的局部变量都需要依赖局部变量表来存访。

>  注意，并不是在方法中用了多少个局部变量，就把这些局部变量所占变量槽数量之和作为max_locals的值，操作数栈和局部变量表直接决定一个该方法的栈帧所耗费的内存，不必要的操作数栈深度和变量槽数量会造成内存的浪费。Java虚拟机的做法是将局部变量表中的变量槽进行重用，当代码执行超出一个局部变量的作用域时，这个局部变量所占的变量槽可以被其他局部变量所使用，Javac编译器会根据变量的作用域来分配变量槽给各个变量使用，根据同时生存的最大局部变量数量和类型计算出max_locals的大小。 

code和code_length用来存储编译后生成的字节码指令。每一个指令都对应一个u1类型的单字节，当虚拟机读取到code中的一个字节码的时候，就可以对应找出这个字节码代表的是什么指令，然后进行解析。

虽然code_length是一个u4类型的长度值，理论上可以达到$2^{32}$次方，但是《Java虚拟机规范中》规定一个方法不能超过65535条字节码指令。

学习了这些内容后，我们可以继续分析之前那个class文件了。

![img](https://pic3.zhimg.com/v2-744ffce56ebbf630fce69e2e5e8df62a_b.png)

```
0001 -> 1个属性
0009 -> 常量区第九，为code 
0000001D -> 长度为29 
0001->栈深度 
0001->变量槽的个数 
00000005 -> 代码长度 
2A B7 00 01 B1 -> 字节码指令 
00000001 -> 异常表的长度
```

接下来我们来翻译一下字节码

2A -> aload_0 将第0个变量槽中的引用类型的本地变量push到栈顶

B7 -> invokespecial 将栈顶的引用类型所指向的对象作为方法的接收者，调用对象的实例构造方法，或者private方法，或者它父类的方法。 这个方法有一个u2类型的参数说明具体调用哪一个方法，它指向常量池中一个`CONSTANT_Methodref_info`变量 

0001 -> invokespecial的参数，查看可知是<init>()方法的符号引用 

B1 -> return指令，返回值为void

![img](https://pic2.zhimg.com/v2-a8ccaf40d6ae3c7b8bfbd4a0bd9983b9_b.png)

![img](https://pic1.zhimg.com/v2-5c76a96847c3c029b5cc72d2c6037d38_b.png)

因为所有方法几乎都会传入一个this参数，所以args_size=1，如果是static方法，那么这里是0

在字节码指令后面还会跟着这个方法里面的显式异常处理表集合，它的格式是这样的

![img](https://pic4.zhimg.com/v2-ea38dd659ae3183d344e9db5f6961983_b.png)

它的含义是：当字节码从第start_pc行到第end_pc行（左闭右开）之间出现了类型为catch_type或者子类的异常（catch_type为指向一个CONSTANT_Class_info型的常量的索引），则转到handler_pc行处理。当catch_type的值为0的时候，表示任何异常情况都要转到handler_pc行处理

### **Exceptions属性**

Exceptions属性就是方法throws关键字后面列举的异常，它的结构为

![img](https://pic3.zhimg.com/v2-3df05e6651c5d357ca8e830d937249c6_b.png)

exception_index_table是一个指向常量池种CONSTANT_Class_info型常量的索引，代表了该受查异常的类型

### **LineNumberTable属性**

这个属性表明了Java源码行号与字节码行号（字节码的偏移量）之间的对应关系。主要的用途是在程序抛出异常的时候，堆栈中会显示出错的行号

![img](https://pic1.zhimg.com/v2-965563ca466352a4f04ed3481f63d8b8_b.png)

line_number_info表包含start_pc和line_number两个u2类型的数据线，前者是字节码行号，后者是Java源码行号

### **LocalVariableTable及LocalVariableTypeTable属性**

LocalVariable属性用于描述栈帧中局部变量和Java源码中定义的变量之间的关系，如果没有这项属性，那么别人引用这个方法的时候，所有的参数名称都将丢失，在调试期间无法根据参数名称从上下文中获取参数值

![img](https://pic3.zhimg.com/v2-ddfa1186823e49f994322d79565dc9aa_b.png)

LocalVariableTable

![img](https://pic1.zhimg.com/v2-76db4eb5508b6d29aae90a2b822754ec_b.png)

local_variable_info

start_pc和length属性分别代表了这个局部变量的生命周期开始的字节码偏移量及其作用范围覆盖的长度，两者的结合就是这个局部变量在在字节码中的作用域范围

index是局部变量在栈帧的局部变量表中变量槽的位置

> 顺便提一下，在JDK 5引入泛型之后，LocalVariableTable属性增加了一个“姐妹属性” —— LocalVariableTypeTable。这个新增的属性结构与LocalVariableTable非常相似，仅仅是把记录的字段描述 符的descriptor_index替换成了字段的特征签名（Signature）。对于非泛型类型来说，描述符和特征签名 能描述的信息是能吻合一致的，但是泛型引入之后，由于描述符中泛型的参数化类型被擦除掉，描述符就不能准确描述泛型类型了。因此出现了LocalVariableTypeTable属性，使用字段的特征签名来完成泛型的描述。

### **SourceFile及SourceDebugExtension属性**

SourceFile属性用于记录生成这个Class文件的源码文件名称。如果不生成这项属性，当抛出异常时，堆栈中将不会显示出错代码所属的文件名。

![img](https://pic4.zhimg.com/v2-0572aff6cec728b07e7c23c7ea88cbc7_b.png)

sourcefile_index数据项是指向常量池中CONSTANT_Utf8_info型常量的索引，常量值是源码文件的 文件名。

为了方便在编译器和动态生成的Class中加入供程序员使用的自定义内容，在JDK 5时，新增加SourceDebugExtension属性用于存储额外的代码调试信息。

![img](https://pic1.zhimg.com/v2-6869a61ba1d08c81436290393d743498_b.png)

### **ConstantValue属性**

ConstantValue属性的作用是通知虚拟机自动为静态变量赋值。对于非static的变量的赋值是在构造方法里面进行的，对于静态变量则有两种方式：在类构造器<clinit>()方法中或者使用ConstantValue属性。

目前Oracle公司的选择是如果final和static同时修饰的基本类型或者String变量就会用ConstantValue属性来初始化，如果没有被final修饰，或者并非基本类型及字符串，就会在<clinit>()方法中进行初始化

![img](https://pic3.zhimg.com/v2-247cf07781082dcf6170d616ea5ea7ce_b.png)

从数据结构中可以看出ConstantValue属性是一个定长属性，它的attribute_length数据项值必须固定为2。constantvalue_index数据项代表了常量池中一个字面量常量的引用，根据字段类型的不同，字面量可以是CONSTANT_Long_info、CONSTANT_Float_info、CONSTANT_Double_info、CONSTANT_Integer_info和CONSTANT_String_info常量中的一种。

### **InnerClasses属性**

这个属性是记录累不累（内部类）和宿主类之间的关系的。如果一个类中定义了内部类，那编译器就会给它以及它所包含的内部内生成InnerClasses属性

![img](https://pic4.zhimg.com/v2-f3438fc8b706e8af05c105a08ba18973_b.png)

![img](https://pic2.zhimg.com/v2-b53a653815fb115038929cd97f484e8d_b.png)

inner_class_info表结构

Inner_class_info_index和outer_class_info_index都是指向常量池中CONSTANT_Class_info型常量的索引，分别表示内部类和宿主类。Inner_name_index指向名称，如果是匿名类，那么这项为0

![img](https://pic3.zhimg.com/v2-748d199d41e04ffab6bf753cc7497bd2_b.png)

inner_class_access_flags标志

### **Deprecated及Synthetic属性**

这两个都是标志类型的布尔属性

Deprecated表示某个类、字段、方法已经被程序作者定为不再推荐使用，可以通过注解@deprecated进行设置

Synthetic属性代表此字段或者方法并不是由Java源码直接产生的，而是由编译器自行添加的。

> 编译器通过生成一些在源代码中不存在的Synthetic方法、字段甚至是整个类的方式，实现了越权访问（越过private修饰器）或其他绕开了语言限制的功能，这可以算是一种早期优化的技巧，其中最典型的例子就是枚举类中自动生成的枚举元素数组和嵌套类的桥接方法（Bridge Method）。

唯一的例外是<init>()和类构造器<clinit>()

![img](https://pic2.zhimg.com/v2-f0702c2084f94adffe9026b523bbd001_b.png)

### **StackMapTable属性**

这个属性位于Code属性的属性表中，它会在JVM加载的字节码验证阶段被新类型检查验证器使用，目的在于代替以前比较消耗性能的基于数据流分析的类型推导验证器

> 这个类型检查验证器最初来源于Sheng Liang（听名字似乎是虚拟机团队中的华裔成员）实现为Java ME CLDC实现的字节码验证器。新的验证器在同样能保证Class文件合法性的前提下，省略了在运行期通过数据流分析去确认字节码的行为逻辑合法性的步骤，而在编译阶段将一系列的验证类型（Verification Type）直接记录在Class文件之中，通过检查这些验证类型代替了类型推导过程，从而大幅提升了字节码验证的性能。这个验证器在JDK 6中首次提供，并在JDK 7中强制代替原本基于类型推断的字节码验证器。关于这个验证器的工作原理，《Java虚拟机规范》在Java SE 7版中新增了整整120页的篇幅来讲解描述，其中使用了庞大而复杂的公式化语言去分析证明新验证方法的严谨性。

StackMapTable属性中包含0-多个栈映射帧，每个栈映射帧都显式或者隐式地代表了一个字节码偏移量，用于表示执行到该字节码时局部变量表和操作数栈的验证类型。类型检查验证器会通过检查目标方法的局部变量和操作数栈所需要的类型来确定一段字节码指令是否符合逻辑约束。

![img](https://pic3.zhimg.com/v2-d1d56cc45be8f8abfe3516f06d06c826_b.png)

> 在Java SE 7版之后的《Java虚拟机规范》中，明确规定对于版本号大于或等于50.0的Class文件，如果方法的Code属性中没有附带StackMapTable属性，那就意味着它带有一个隐式的StackMap属性，这个StackMap属性的作用等同于number_of_entries值为0的StackMapTable属性。一个方法的Code属性最多只能有一个StackMapTable属性，否则将抛出ClassFormatError异常。

### **Signature属性**

Signature属性出现在类、字段表、方法表结构的属性表中，会为这些属性记录泛型签名信息。

> Java语言的泛型采用的是擦除法实现的伪泛型，字节码（Code属性）中所有的泛型信息编译（类型变量、参数化类型）在编译之后都通通被擦除掉，运行期就无法像C#等有真泛型支持的语言那样，将泛型类型与用户定义的普通类型同等对待，例如运行期做反射时无法获得泛型信息。

现在Java的反射API可以获取泛型类型，最终的数据来源也是这个属性。

![img](https://pic4.zhimg.com/v2-f6e16e55fab9b167db7740acb71f0dff_b.png)

signature_index的值必须是一个对常量池的有效索引，这个索引项必须是CONSTANT_Utf8-info结构，表示类签名或者方法签名或者字段类型签名。

### **BootstrapMethods属性**

这个属性用于保存invoked dynamic指令引用的引导方法限定符。如果某个类文件结构的常量池中曾经出现过CONSTANT_InvokeDynamic_info属性的常量，那么这个类文件的属性表中必须存在一个明确的BootstrapMethod属性，另外，即使CONSTANT_InvokeDynamic_info类型的常量在常量池中出现过多次，类文件的属性表中最多也只能有一个BootstrapMethods属性。

> invokedynamic指令是JDK 7实现“动态类型语言”（Dynamically Typed Language）支持而进行的改进之一，也是为JDK 8可以顺利实现Lambda表达式做技术准备。 动态类型是指，在运行的时候（runtime）才检查数据类型，比如php、javascript。

![img](https://pic1.zhimg.com/v2-8c15ddb66266315260b175436063b68c_b.png)

num_bootstrap_methods项的值给出了bootstrap_methods[]数组中的引导方法限定符的数量。而bootstrap_methods[]数组的每个成员包含了一个指向常量池CONSTANT_MethodHandle结构的索引值，它代表了一个引导方法，还包含了这个引导方法静态参数的序列（可能为空）

![img](https://pic3.zhimg.com/v2-b2bc01ec495726baf211f584038d9aaa_b.png)

bootstrap_method属性结构

bootstrap_method_ref项的值必须是一个对常量池的有效索引。

bootstrap_arguments[]数组的每个成员必须是一个对常量池的有效索引。

### **MethodParameters属性**

它记录了方法的各个形参名称和信息，是方法表的属性，与Code平级，能够在运行时通过反射API获取

![img](https://pic4.zhimg.com/v2-86b71b78c086072545a66db76fa79a37_b.png)

![img](https://pic4.zhimg.com/v2-9e7a2a6f305405fb2ae5defd46018607_b.png)

parameter属性结构

access_flags是参数的状态指示器，它可以包含以下三种状态中·的一种或者多种：

- 0x0010：被final修饰
- 0x1000：参数未出现在源文件中，编译器自动生成的
- 0x8000：参数在源文件中隐式定义

### **模块化相关属性**

这个是Java9的功能，除了表示该模块的名称，版本，标志信息以外，还存储了这个模块的requires，exports，opens，uses，和provides定义的全部内容

> 个模块是一个被命名的，代码和数据的自描述的集合 一个或更多个requires项可以被添加到其中，它通过名字声明了这个模块依赖的一些其他模块，在编译期和运行期都依赖的 exports 仅仅使指定包（package）中的公共类型可以被其他的模块使用 opens 用于声明该模块的指定包在runtime允许使用反射访问 uses 指定服务接口的名字，当前模块就会发现它，使用 java.util.ServiceLoader类进行加载 provider 指定一个或多个服务接口的实现类

![img](https://pic4.zhimg.com/v2-bebbb714c321adb9500f12bd60f21d37_b.png)

module_flags是模块的状态指示器，它可以包含以下三种状态中的一种或多种：

- 0x0020（ACC_OPEN）：表示该模块是开放的。
- 0x1000（ACC_SYNTHETIC）：表示该模块并未出现在源文件中，是编译器自动生成的。
- 0x8000（ACC_MANDATED）：表示该模块是在源文件中隐式定义的。

### **RuntimeVisibleAnnotations属性**

它记录了类、字段或方法的声明上的运行时可见注解，当我们使用反射API来获取类、字段或方法上的注解时，返回值就是通过这个属性来取到的。

![img](https://pic2.zhimg.com/v2-d9e961045c9c302c8449bb073351ecf1_b.png)

annotations中每个元素都代表了一个运行时可见的注解，注解在Class文件中以annotation结构来存储

![img](https://pic4.zhimg.com/v2-0f7e5e1485888fc3d9e80a3018d27027_b.png)

annotation属性结构

element_value_pairs中每个元素都是一个键值对，代表该注解的参数和值。

### 参考

- [1] 深入理解Java虚拟机 周志明
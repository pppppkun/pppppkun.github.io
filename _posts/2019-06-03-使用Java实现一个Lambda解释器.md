---
layout:     post
title:      使用Java实现一个lambda解释器
subtitle:   Lambda解释器
date:       2019-03-15
author:     Pkun
header-img: 
catalog: true
tags:
    - Lambda演算
    - 函数式编程
    - Java
---

## Lambda Interpreter

[A λ-calculus interpreter](https://tadeuzagallo.com/blog/writing-a-lambda-calculus-interpreter-in-javascript/)

[interpreter used Java by Pkun](https://github.com/pppppkun/LambdaInterpreter)

---

我们利用自顶向下的思考方式，首先，输入是一个lambda表达式，为了方便起见，我们将lambda写作\，所以输入的式子应该是这样。
`
(\x.\y (x y)) (\p.p)(\q.q)
`
那么改如何去解释它呢？我们先用常规的方法计算一下。很容易看出我们只需要把后面两个抽象带入到前面的抽象就可以了。如果是计算机做的话，上面这句话可以分解成
- 识别出三个抽象
- 将后面的抽象按照顺序带入到前面的抽象中

我们先复习一下之前学过的lambda演算的知识
```
t1 t2   # Application
 
\x. t1  # Abstraction
 
x       # Identifier
```
如果要一般化，第一步可以理解成
- ### 计算机必须能够翻译lambda表达式中的所有项，分别是App，Abs和Ide。

那么该如何翻译出所有的项呢？我们注意到他们三个各有特点，Abs中有lambda，Application有两支，Identifier只有一个值，另一方面，注意到我们读入的是一个字符串，所以我们可以采用从头到尾遍历整个字符串的方法，将字符串中的所有项都读出来。
到这里，我们要注意到的一件事情就是Application，Abstraction是可以相互嵌套的，Identifier也可以是Application的左右两支，最后出来的应该是一个树状的结构，到这里，我们不难想到让三个类都继承一个节点类，通过节点访问节点的节点，来遍历整棵树，避免嵌套带来的类型转换的麻烦。

## 我们规定一个抽象语法树，它的节点有三个，抽象，应用和原子。

既然要遍历整个字符串，我们采用枚举的方式，一旦符合某种模式，就将他归类为某种语法，所以我们需要规定一个lambda表达式中会出现的所有字符，其实也很简单，是下面六种
```
LPAREN: '('
RPAREN: ')'
LAMBDA: '\' // 为了方便使用 “\”
DOT: '.'
LCID: /[a-z][a-zA-Z]*/ 
EOF: null
```
我们将这些字符命名为Token，并且通过识别不同的Token，把一个Lambda表达式解析成一个抽象语法树。到这里，我们已经构建出一个计算机可以阅读的Lambda表达式了。接下来我们要让计算机在遍历的过程中，把整个抽象语法树构建出来。为了方便起见，我们制定三条语法规则：
```
Term ::= Application| LAMBDA LCID DOT Term

Application ::= Application Atom| Atom

Atom ::= LPAREN Term RPAREN| LCID

```
整体的思路就是不断读入下一个字符，判断它是Token中的哪一种，然后在Term Application Atom三个方法之中跳转，构建出整个树。

## 接下来让我们考虑第二步，也就是做beta规约的事情。

其实想想还是挺容易的一件事情，因为在我们的抽象语法树里面，如果某个节点的左支是Abstraction，你可以直接把右支带入到左支里面去，替换左支body中的所有和param一样的东西。写成伪码的话应该是这样子
```
while(hasnext(Abs.Body)){
  if(charInBody==Abs.Param) charInBody = Ide;
}
```
笔者认为也这是一种比较好的处理方式，但是很多细节问题没有考虑，也不知道这种处理方式最后效果怎么样。按道理来说是不会出现因为两个Abs的param相同而导致代入出现问题的。接下来我要介绍老师钦定的方法来构造求值的函数。

- 2019.9.29注：确实会出现两个ABS.param相同的情况，原因是因为在做beta规约的时候，计算机会误解内层和外层

## De Bruijn index

关于De Bruijn,你可以在下面网站中找到你想要的 [De Bruijn Sequence](https://en.wikipedia.org/wiki/De_Bruijn_sequence)（无法查看的话可以百度百科）

用De Bruijin Index，我们可以给Abs或者Ide中的项打上自己的标签，比如说\x.\y x y 可以看成 \x.\y 1 0，但是对于\x.a 这样的变量没有绑定的时候，默认换成\x. 1（如果博客没写错的话）。

引入De Bruijn Index最主要的原因是alpha替换，lambda表达式中的符号是可以随意替换的，它并没有具体的含义，是一种假变量，就算重复也没有关系，当然，在同一个abs或者其他的一些式子之中，我们为了区分和计算还是应该将变量取上不同的名字。

当然，运用了De Bruijn Index之后，我们的前一个阶段又出现了一个问题，我们考虑这样的一个抽象`\x.\y.x (\x. x)`
如果你要给上述的抽象进行De Bruijn转换的话，内层的x的序号应该是多少呢？我们说，它转换后应该是这个样子
`\.\.1 (\. 0)`

因为内层的abs绑定的变量就算名字和外层的一样，但是我们认为名字是无关的，所以应该重新计算内层abs的De Bruijn值。

这里我们要引入一个上下文存储的结构，叫做ctx，运用它我们就可以很好的解决这个内外层param相同，De Bruijn不同的问题。

使用De Bruijn Index之后，当你替换了之后，你可能还存在于方法栈的顶端，意思就是被替换的Abs还没有被完全改变，这时候，有可能产生一种误解，比如下面这样：
```
Abs = \x.\y.x = \.\.1
ide = \t.a = \t.1
```
替换后Abs 将会在这个方法还没跳出栈之前被设置为`\x.\y.\t.1`这个时候就会产生误解，这个1到底是指的x还是指的a呢？

所以我们要引入一些新的方法，来解决这个问题。我们将\t.1升序，升到\t.2，然后带入到Abs中，然后我们考虑代入的深度，因为只进行一次带入的话我们只需要考虑最外层会不会产生冲突，如果被你替换的那个值正好是最外层的值，这表明你需要再对你的\t.2进行一次升序，升到不会产生冲突，升多少呢？保险起见，我们应该升到超过它的深度，这样就不会产生任何误解。最后，我们再将一开始升序的那一次降下来，这样整个替换的过程就完成了，也不会出现任何问题。

这样，整个lambda解释器的基本原理就已经讲解完了，其他就是代码怎么写了，不过代码难度其实也挺小的，你甚至都不需要知道代码背后是怎么运行的，而是按部就班的按照上面所写的慢慢敲出来，最后也可以通过OJ。



---
layout:     post
title:      JavaScript-对象
subtitle:   前端新手上路-JavaScript
date:       2019-11-12
author:     Pkun
header-img: img/JS.jpg
catalog: true
tags:
    - 前端
    - JavaScript
    - 脚本语言
---

## 对象的基本操作

### 创建对象

使用new关键字调用的函数，是构造函数constructor。**构造函数是专门用来创建对象的函数**。

例如：

```javascript
	var obj = new Object();
```

另外，使用`typeof`检查一个对象时，会返回`object`。

### 向对象中添加属性

在对象中保存的值称为属性。

向对象添加属性的语法：

```javascript
	对象.属性名 = 属性值;
```

举例：

```javascript
    var obj = new Object();

    //向obj中添加一个name属性
    obj.name = "孙悟空";

    //向obj中添加一个gender属性
    obj.gender = "男";

    //向obj中添加一个age属性
    obj.age = 18;

    console.log(JSON.stringify(obj)); // 将 obj 以字符串的形式打印出来
```


打印结果：

```
	{
		"name":"孙悟空",
		"gender":"男",
		"age":18
	}
```

**补充1**：对象的属性值可以是任何的数据类型，也可以是个**函数**：（也称之为方法）

```javascript
    var obj = new Object();
    obj.sayName = function () {
        console.log('smyhvae');
    };

    console.log(obj.sayName);  //没加括号，获取的是对象
    console.log('-----------');
    console.log(obj.sayName());  //加了括号，执行函数内容，并执行函数体的内容

```

打印结果：

![](http://img.smyhvae.com/20180314_2109.png)

**补充2**：js中的属性值，也可以是一个**对象**。

举例：

```javascript
    //创建对象 obj1
    var obj1 = new Object();
    obj1.test = undefined;

    //创建对象 obj2
    var obj2 = new Object();
    obj2.name = "smyhvae";

    //将整个 obj2 对象，设置为 obj1 的属性
    obj1.test = obj2;

    console.log(obj1.test.name);
```

打印结果为：smyhvae

### 获取对象中的属性

**方式1**：

语法：

```javascript
	对象.属性名
```

如果获取对象中没有的属性，不会报错而是返回`undefined`。

举例：

```javascript
    var obj = new Object();

    //向obj中添加一个name属性
    obj.name = "孙悟空";

    //向obj中添加一个gender属性
    obj.gender = "男";

    //向obj中添加一个age属性
    obj.age = 18;

    // 获取对象中的属性，并打印出来
    console.log(obj.gender); // 打印结果：男
    console.log(obj.color);  // 打印结果：undefined
```


**方式2**：可以使用`[]`这种形式去操作属性

对象的属性名不强制要求遵守标识符的规范，但是我们使用是还是尽量按照标识符的规范去做。

但如果要使用特殊的属性名，就不能采用`.`的方式来操作对象的属性。比如说，`123`这种属性名，如果我们直接写成`obj.123 = 789`，是会报错的。那怎么办呢？办法如下：

语法格式如下：（读取时，也是采用这种方式）

```
对象["属性名"] = 属性值

```


上面这种语法格式，举例如下：

```javascript
 obj["123"] = 789;
```


**重要**：使用`[]`这种形式去操作属性，更加的灵活，因为，我们可以在`[]`中直接传递一个**变量**，这样变量值是多少就会读取那个属性。


### 修改对象的属性值

语法：

```javascript
	对象.属性名 = 新值
```


```javascript
	obj.name = "tom";
```


### 删除对象的属性

语法：

```javascript
	delete obj.name;
```


### in 运算符

通过该运算符可以检查一个对象中是否含有指定的属性。如果有则返回true，没有则返回false。

语法：

```javascript
	"属性名" in 对象
```

举例：

```javascript
	//检查obj中是否含有name属性
	console.log("name" in obj);
```


我们平时使用的对象不一定是自己创建的，可能是别人提供的，这个时候，in 运算符可以派上用场。

## 对象字面量

如果要创建一个对象，我们可以使用下面这种方式：

```javascript
	var obj = new Object();
```


但是上面的这种方式，比较麻烦，我们还有更简洁的方式来创建一个对象。如下。

使用对象字面量来创建一个对象：

```javascript
	var obj = {};
```


使用对象字面量，可以在创建对象时，直接指定对象中的属性。语法：{属性名:属性值,属性名:属性值....}

例如：

```javascript
	var obj2 = {

		name: "猪八戒",
		age: 13,
		gender: "男",
		test: {
			name: "沙僧"
		}
		//我们还可以在对象中增加一个方法。以后可以通过obj2.sayName()的方式调用这个方法
		sayName: function(){
			console.log('smyhvae');
		}
	};
```


对象字面量的属性名可以加引号也可以不加，建议不加。如果要使用一些特殊的名字，则必须加引号。

属性名和属性值是一组一组的键值对结构，键和值之间使用`:`连接，多个值对之间使用`,`隔开。如果一个属性之后没有其他的属性了，就不要写`,`，因为它是对象的最后一个属性。


## 遍历对象中的属性：for in

语法：

```javascript
	for (var 变量 in 对象) {

	}
```

解释：对象中有几个属性，循环体就会执行几次。每次执行时，会将对象中的**每个属性的 属性名 赋值给变量**。

举例：


```html
<!DOCTYPE html>
<html>

<head>
	<meta charset="UTF-8">
	<title></title>
	<script type="text/javascript">
		var obj = {
			name: "smyhvae",
			age: 26,
			gender: "男",
			address: "shenzhen"
		};

		//枚举对象中的属性
		for (var n in obj) {
			console.log("属性名:" + n);
			console.log("属性值:" + obj[n]); // 注意，因为这里的属性名 n 是变量，所以，如果想获取属性值，不能写成 obj.n，而是要写成 obj[n]
		}
	</script>
</head>

<body>
</body>

</html>

```

打印结果：

![](http://img.smyhvae.com/20180314_2125.png)



## 创建自定义对象的几种方法

### 方式一：对象字面量

**对象的字面量**就是一个{}。里面的属性和方法均是**键值对**。

例如：

```javascript
var o = {
            name: "生命壹号",
            age: 26,
            isBoy: true,
            sayHi: function() {
                console.log(this.name);
            }
        };

console.log(o);
```

控制台输出：

![](http://img.smyhvae.com/20180125_1834.png)

### 方式二：工厂模式

通过该方法可以大批量的创建对象。

```javascript
        /*
         * 使用工厂方法创建对象
         *  通过该方法可以大批量的创建对象
         */
        function createPerson(name, age, gender) {
            //创建一个新的对象
            var obj = new Object();
            //向对象中添加属性
            obj.name = name;
            obj.age = age;
            obj.gender = gender;
            obj.sayName = function() {
                alert(this.name);
            };
            //将新的对象返回
            return obj;
        }

        var obj2 = createPerson("猪八戒", 28, "男");
        var obj3 = createPerson("白骨精", 16, "女");
        var obj4 = createPerson("蜘蛛精", 18, "女");
```

**弊端：**

使用工厂方法创建的对象，使用的构造函数都是Object。**所以创建的对象都是Object这个类型，就导致我们无法区分出多种不同类型的对象**。

### 方式三：利用构造函数

```javascript
        //利用构造函数自定义对象
        var stu1 = new Student("smyh");
        console.log(stu1);
        stu1.sayHi();

        var stu2 = new Student("vae");
        console.log(stu2);
        stu2.sayHi();


        // 创建一个构造函数
        function Student(name) {
            this.name = name;    //this指的是构造函数中的对象实例
            this.sayHi = function () {
                console.log(this.name + "厉害了");
            }
        }
```

打印结果：

![](http://img.smyhvae.com/20180125_1350.png)

接下来，我们专门来讲一下构造函数。

## 构造函数

### 代码引入


```javascript
      // 创建一个构造函数，专门用来创建Person对象
      function Person(name, age, gender) {
        this.name = name;
        this.age = age;
        this.gender = gender;
        this.sayName = function() {
          alert(this.name);
        };
      }

      // 创建一个构造函数，专门用来创建 Dog 对象
      function Dog() {}

      var per = new Person("孙悟空", 18, "男");
      var per2 = new Person("玉兔精", 16, "女");
      var per3 = new Person("奔波霸", 38, "男");

      var dog = new Dog();
```


### 构造函数和普通函数的区别

构造函数就是一个普通的函数，创建方式和普通函数没有区别，不同的是构造函数习惯上首字母大写。

构造函数和普通函数的区别就是**调用方式**的不同：普通函数是直接调用，而构造函数需要使用new关键字来调用。

**this的指向也有所不同**：

- 1.以函数的形式调用时，this永远都是window。比如`fun();`相当于`window.fun();`

- 2.以方法的形式调用时，this是调用方法的那个对象

- 3.以构造函数的形式调用时，this是新创建的那个对象

### new 一个构造函数的执行流程

（1）开辟内存空间，存储新创建的对象

（2）将新建的对象设置为构造函数中的this，在构造函数中可以使用this来引用 新建的对象

（3）执行函数中的代码（包括设置对象属性和方法等）

（4）将新建的对象作为返回值返回

因为this指的是new一个Object之后的对象实例。于是，下面这段代码：

```javascript
        // 创建一个函数
        function createStudent(name) {
            var student = new Object();
            student.name = name;      //第一个name指的是student对象定义的变量。第二个name指的是createStudent函数的参数。二者不一样
        }
```

可以改进为：

```javascript
        // 创建一个函数
        function Student(name) {
            this.name = name;       //this指的是构造函数中的对象实例
        }

```

注意上方代码中的注释。

### 类、实例

使用同一个构造函数创建的对象，我们称为一类对象，也将一个构造函数称为一个**类**。

通过一个构造函数创建的对象，称为该类的**实例**。

### instanceof

使用 instanceof 可以检查**一个对象是否为一个类的实例**。

**语法如下**：

```javascript
对象 instanceof 构造函数
```

如果是，则返回true；否则返回false。

**代码举例**：

```javascript
      function Person() {}

      function Dog() {}

      var person1 = new Person();

      var dog1 = new Dog();

      console.log(person1 instanceof Person); // 打印结果： true
      console.log(dog1 instanceof Person); // 打印结果：false

      console.log(dog1 instanceof Object); // 所有的对象都是Object的后代。因此，打印结果为：true
```

根据上方代码中的最后一行，需要补充一点：**所有的对象都是Object的后代，因此 `任何对象 instanceof Object` 的返回结果都是true**。


## others

### json的介绍

> 对象字面量和json比较像，这里我们对json做一个简单介绍。

JSON：JavaScript Object Notation（JavaScript对象表示形式）。

JSON和对象字面量的区别：JSON的属性必须用双引号引号引起来，对象字面量可以省略。

json举例：

```
      {
            "name" : "zs",
            "age" : 18,
            "sex" : true,
            "sayHi" : function() {
                console.log(this.name);
            }
        };
```

注：json里一般放常量、数组、对象等，但很少放function。

另外，对象和json没有长度，json.length的打印结果是undefined。于是乎，自然也就不能用for循环遍历（因为遍历时需要获取长度length）。

**json遍历的方法：**

json 采用 `for...in...`进行遍历，和数组的遍历方式不同。如下：


```html
<script>
    var myJson = {
        "name": "smyhvae",
        "aaa": 111,
        "bbb": 222
    };

    //json遍历的方法：for...in...
    for (var key in myJson) {
        console.log(key);   //获取 键
        console.log(myJson[key]); //获取 值（第二种属性绑定和获取值的方法）
        console.log("------");
    }
</script>
```

打印结果：

![](http://img.smyhvae.com/20180203_1518.png)


> 在看本文之前，我们可以先复习上一篇文章：《03-JavaScript基础/12-对象的创建&构造函数.md》

## 原型对象

### 原型的引入

```javascript
        function Person(name, age, gender) {
            this.name = name;
            this.age = age;
            this.gender = gender;
            //向对象中添加一个方法
            this.sayName = function () {
                console.log("我是" + this.name);
            }
        }

        //创建一个Person的实例
        var per = new Person("孙悟空", 18, "男");
        var per2 = new Person("猪八戒", 28, "男");
        per.sayName();
        per2.sayName();

        console.log(per.sayName == per2.sayName);  //打印结果为false
```

**分析如下**：

上方代码中，我们的sayName方法是写在构造函数 Person 内部的，然后在两个实例中进行了调用。造成的结果是，**构造函数每执行一次，就会给每个实例创建一个新的 sayName 方法**。也就是说，每个实例的sayName都是唯一的。因此，最后一行代码的打印结果为false。

按照上面这种写法，假设创建10000个对象实例，就会创建10000个 sayName 方法。这种写法肯定是不合适的。我们为何不让所有的对象共享同一个方法呢？

还有一种方式是，将sayName方法在全局作用域中定义：（不建议。原因看注释）

```javascript
        function Person(name, age, gender) {
            this.name = name;
            this.age = age;
            this.gender = gender;
            //向对象中添加一个方法
            this.sayName = fun;
        }

        //将sayName方法在全局作用域中定义
        /*
         * 将函数定义在全局作用域，污染了全局作用域的命名空间
         *  而且定义在全局作用域中也很不安全
         */
        function fun() {
            alert("Hello大家好，我是:" + this.name);
        };
```

比较好的方式是，在原型中添加sayName方法：

```javascript
    Person.prototype.sayName = function(){
        alert("Hello大家好，我是:"+this.name);
    };
```

这也就引入了我们本文要讲的「原型」。

### 原型prototype的概念

**认识1**：

我们所创建的每一个函数，解析器都会向函数中添加一个属性 prototype。这个属性对应着一个对象，这个对象就是我们所谓的原型对象。

如果函数作为普通函数调用prototype没有任何作用，当函数以构造函数的形式调用时，它所创建的实例对象中都会有一个隐含的属性，指向该构造函数的原型，我们可以通过__proto__来访问该属性。

代码举例：

```javascript
	// 定义构造函数
	function Person() {}

	var per1 = new Person();
	var per2 = new Person();

	console.log(Person.prototype); // 打印结果：[object object]

	console.log(per1.__proto__ == Person.prototype); // 打印结果：true
```

上方代码的最后一行：打印结果表明，`实例.__proto__` 和 `构造函数.prototype`都指的是原型对象。

**认识2**：

原型对象就相当于一个公共的区域，所有同一个类的实例都可以访问到这个原型对象，我们可以将对象中共有的内容，统一设置到原型对象中。

以后我们创建构造函数时，可以将这些对象共有的属性和方法，统一添加到构造函数的原型对象中，这样就不用分别为每一个对象添加，也不会影响到全局作用域，就可以使每个对象都具有这些属性和方法了。

**认识3**：

使用 `in` 检查对象中是否含有某个属性时，如果对象中没有但是**原型中**有，也会返回true。

可以使用对象的`hasOwnProperty()`来检查**对象自身中**是否含有该属性。

### 原型链

原型对象也是对象，所以它也有原型，当我们使用或访问一个对象的属性或方法时：

- 它会先在对象自身中寻找，如果有则直接使用；

- 如果没有则会去原型对象中寻找，如果找到则直接使用；

- 如果没有则去原型的原型中寻找，直到找到Object对象的原型。

- Object对象的原型没有原型，如果在Object原型中依然没有找到，则返回 null

### 总结

第一次接触「原型」和「原型链」的时候，会比较难理解。多接触几次，再回过头来看，就慢慢熟悉了。

## 对象的 toString() 方法

我们先来看下面这段代码：

```javascript
	function Person(name, age, gender) {
	this.name = name;
	this.age = age;
	this.gender = gender;
	}

	var per1 = new Person("vae", 26, "男");

	console.log("per1 = " + per1);
	console.log("per1 = " + per1.toString());
```

打印结果：

```
per1 = [object Object]
per1 = [object Object]
```

上面的代码中，我们尝试打印实例 per1 的内部信息，但是发现，无论是打印 `per1` 还是打印 `per1.toString()`，结果都是`object`，这是为啥呢？分析如下：

- 当我们直接在页面中打印一个对象时，其实是输出了对象的toString()方法的返回值。

- 如果我们希望在打印对象时，不输出[object Object]，可以手动为对象添加一个toString()方法。意思是，重写 toString() 方法。

重写 toString() 方法，具体做法如下：

```javascript
	function Person(name, age, gender) {
	this.name = name;
	this.age = age;
	this.gender = gender;
	}

	//方式一：重写 Person 原型的toString方法。针对 Person 的所有实例生效
	Person.prototype.toString = function() {
		return (
		  "Person[name=" +
		  this.name +
		  ",age=" +
		  this.age +
		  ",gender=" +
		  this.gender +
		  "]"
		);
	};

	// 方式二：仅重写实例 per1 的 toString方法。但是这种写法，只对 per1 生效， 对 per2 无效
	/*
	per1.toString = function() {
		return (
		  "Person[name=" +
		  this.name +
		  ",age=" +
		  this.age +
		  ",gender=" +
		  this.gender +
		  "]"
		);
	};
	*/

	var per1 = new Person("smyh", 26, "男");

	var per2 = new Person("vae", 30, "男");

	console.log("per1 = " + per1);
	console.log("per2 = " + per2.toString());
```

打印结果：

```javascript
per1 = Person[name=smyh,age=26,gender=男]
per2 = Person[name=vae,age=30,gender=男]
```

代码分析：

上面的代码中，仔细看注释。我们重写了 Person 原型的 toString()，这样的话，可以保证对 Person 的所有实例生效。

从这个例子，我们可以看出 `prototype` 的作用。

## JS的垃圾回收（GC）机制

程序运行过程中会产生垃圾，这些垃圾积攒过多以后，会导致程序运行的速度过慢。所以我们需要一个垃圾回收的机制，来处理程序运行过程中产生垃圾。

当一个对象没有任何的变量或属性对它进行引用时，此时我们将永远无法操作该对象，此时这种对象就是一个垃圾，这种对象过多会占用大量的内存空间，导致程序运行变慢，所以这种垃圾必须进行清理。

上面这句话，也可以这样理解：如果堆内存中的对象，没有任何变量指向它时，这个堆内存里的对象就会成为垃圾。

JS拥有自动的垃圾回收机制，会自动将这些垃圾对象从内存中销毁。我们不需要也不能进行垃圾回收的操作。我们仅仅需要做的是：如果你不再使用该对象，那么，将改对象的引用设置为 null 即可。




---
layout: post
title:  javascript中的this
date:   2017-10-29 21:19:00 +0800
categories: javascript
tag: note
---

* content
{:toc}


this是Javascript语言的一个关键字它代表函数运行时，自动生成的一个内部对象，只能在函数内部使用。通俗地讲即this的指向在函数定义的时候是确定不了的，只有函数执行的时候才能确定this到底指向谁，实际上this的最终指向的是那个调用它的对象


javscript中函数调用的四种方式
====================================

this的最终指向的是那个调用它的对象称为运行期绑定，它可以是全局对象、当前对象或者任意对象，这完全取决于函数的调用方式。JavaScript中函数的调用有以下几种方式：作为对象方法调用，作为函数调用，作为构造函数调用，和使用apply或call

> **PS:**JavaScript 中的所有事物都是对象

作为对象方法调用
------------------------------------

在JavaScript中，函数也是对象，因此函数可以作为一个对象的属性，此时该函数被称为该对象的方法，在使用这种调用方式时，this被自然绑定到该对象。

```javascript
const person = {
	name: 'Frank',
    eat: function() {
		console.log(`${this.name} is having dinner`);
	}
}
person.eat(); // Frank is having dinner
```

作为函数调用
------------------------------------

函数也可以直接被调用，此时this绑定到全局对象。在浏览器中，window就是该全局对象。比如下面的例子：函数被调用时，this被绑定到全局对象，接下来执行赋值语句，相当于隐式的声明了一个全局变量，这显然不是调用者希望的

```javascript
var x = 1; 
function test(){ 
　　alert(this.x); 
} 
test(); // 等价于window.test();
alert(x); // 1


var x = 1; 
function test(){ 
　　this.x = 0; 
} 
test(); // 等价于window.test();
alert(x); // 0 
```

> **PS:**此时test函数的调用属于全局调用，因此this就代表全局对象Global。

函数内部的调用

```javascript
var point = { 
    x : 0, 
	y : 0, 
	moveTo : function(x, y) { 
    	// 内部函数
    	var moveX = function(x) { 
    		this.x = x; // this 绑定到了哪里？
    	}; 
    	// 内部函数
    	var moveY = function(y) { 
    		this.y = y; // this 绑定到了哪里？
    	}; 
 
    	moveX(x); 
    	moveY(y); 
    } 
}; 
point.moveTo(1, 1); 
point.x; // 0 
point.y; // 0 
x; // 1 
y; // 1
```

> 结果是不是很令人吃惊？这属于JavaScript的设计缺陷，正确的设计方式是内部函数的 this应该绑定到其外层函数对应的对象上，为了规避这一设计缺陷，聪明的avaScript程序员想出了变量替代的方法，约定俗成，该变量一般被命名为that。

```javascript
var point = { 
    x : 0, 
	y : 0, 
	moveTo : function(x, y) { 
		// 此时的this指向point对象
		var that = this; 
    	// 内部函数
    	var moveX = function(x) { 
    		that.x = x; 
    	}; 
    	// 内部函数
    	var moveY = function(y) { 
    		that.y = y; 
    	}; 
 
    	moveX(x); 
    	moveY(y); 
    } 
}; 
point.moveTo(1, 1); 
point.x; // 1 
point.y; // 1 
```

作为构造函数调用
------------------------------------

JavaScript支持面向对象式编程，与主流的面向对象式编程语言不同，JavaScript并没有类（class）的概念（在ES6中出现了class的语法糖），而是使用基于原型（prototype）的继承方式。相应的，JavaScript中的构造函数也很特殊，如果不使用new调用，则和普通函数一样。作为又一项约定俗成的准则，构造函数以大写字母开头，提醒调用者使用正确的方式调用。如果调用正确，this绑定到新创建的对象上。

```javascript
function Point(x, y){ 
   this.x = x; 
   this.y = y; 
}
```

使用apply或all调用
------------------------------------

让我们再一次重申，在JavaScript中函数也是对象，对象则有方法，apply和call就是函数对象的方法。这两个方法异常强大，他们允许切换函数执行的上下文环境（context），即this绑定的对象。很多JavaScript中的技巧以及类库都用到了该方法。让我们看一个具体的例子：

```javascript
function Point(x, y){ 
   this.x = x; 
   this.y = y; 
   this.moveTo = function(x, y){ 
       this.x = x; 
       this.y = y; 
   } 
} 
 
var p1 = new Point(0, 0); 
var p2 = {x: 0, y: 0}; 
p1.moveTo(1, 1); 
p1.moveTo.apply(p2, [10, 10]);
```

> 在上面的例子中，我们使用构造函数生成了一个对象p1，该对象同时具有moveTo方法；使用对象字面量创建了另一个对象p2，我们看到使用apply可以将p1的方法应用到p2上，这时候this也被绑定到对象p2上。另一个方法call也具备同样功能，不同的是最后的参数不是作为一个数组统一传入，而是分开传入的。

函数的执行环境
====================================

JavaScript中的函数既可以被当作普通函数执行，也可以作为对象的方法执行，这是导致this含义如此丰富的主要原因。一个函数被执行时，会创建一个执行环境（ExecutionContext），函数的所有的行为均发生在此执行环境中，构建该执行环境时，JavaScript首先会创建 arguments变量，其中包含调用函数时传入的参数。接下来创建作用域链。然后初始化变量，首先初始化函数的形参表，值为arguments变量中对应的值，如果arguments变量中没有对应值，则该形参初始化为undefined。如果该函数中含有内部函数，则初始化这些内部函数。如果没有，继续初始化该函数内定义的局部变量，需要注意的是此时这些变量初始化为undefined，其赋值操作在执行环境（ExecutionContext）创建成功后，函数执行时才会执行，这点对于我们理解JavaScript中的变量作用域非常重要，鉴于篇幅，我们先不在这里讨论这个话题。最后为this变量赋值，如前所述，会根据函数调用方式的不同，赋给this全局对象，当前对象等。至此函数的执行环境（ExecutionContext）创建成功，函数开始逐行执行，所需变量均从之前构建好的执行环境（ExecutionContext）中读取。

```javascript
this.a = 1;
var module = {
    a : 2,
    getA:function() {
    return this.a;    
    }
};
module.getA(); // 2 this指向module

var getA1 = module.getA;
// getA在外部调用，此时的this指向了全局对象
getA1(); // 1 等价于window.getA1

// 再把getA1方法绑定到module环境上
var getA2 = getA1.bind(module);
getA2(); // 2
```

> 函数的bind方法可以用来绑定函数执行时this对象，点到为止其他的就不多说的，感兴趣的话可以查阅相关文档

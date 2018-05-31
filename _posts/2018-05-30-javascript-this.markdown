---
layout: post
title:  "JavaScript确定this的值"
date:   2018-05-30 22:08:34 +0800
categories: jekyll update
---
先说结论：
> * 全局代码中：this指向全局对象。
> * 通过apply或call显式指定this对象：this指向该显式指定的对象。
> * 函数代码中：找到函数实际调用的地方，如果函数是通过访问属性来调用的，则this指向该属性的所有者；其余情况（比如通过标示符（identifier）来调用的），this都指向全局对象。

#  this的概念
This是执行上下文（Execution Contexts）的一个属性。
```javascript
activeExecutionContext = {
  VO: {...},
  this: thisValue
};
```
This与上下文的可执行代码类型有关，其值在进入上下文阶段就确定了，并且在执行代码阶段是不能改变的。
关于执行上下文的更多内容可以看 [这篇博客]({% post_url 2018-05-29-javascript-execution-contexts-and-variable-object %}) 。

下面分别对结论做解释：
#  全局代码中
这种情况很简单，this总是指向全局对象。
```javascript
// 显式定义全局对象的属性
this.a = 10; // global.a = 10
console.log(a); // 10

// 通过赋值给不受限的标识符来进行隐式定义
b = 20;
console.log(this.b); // 20

// 通过变量声明来进行隐式定义
// 因为全局上下文中的变量对象就是全局对象本身
var c = 30;
console.log(this.c); // 30
```
#  通过apply或call显式指定this对象
这种情况也很简单，看例子：
```javascript
var b = 10;
 
function a(c) {
  console.log(this);
}

// 通过标示符a来调用的，函数中的this指向全局对象
a(20); // global

// 显式指定this指向对象{b: 20}
a.call({b: 20}, 30); // {b: 20}
// 显式指定this指向对象{b: 30}
a.apply({b: 30}, [40]) // {b: 30}
```
#  函数代码中
直接举例说明：
```javascript
var foo = {x: 10};
var bar = {
  x: 20,
  test: function () {
    console.log(this);
  }
};
 
// 通过访问属性test来调用的，函数中的this指向属性test的所有者，即bar对象
bar.test(); // bar
 
foo.func = bar.test;
// 通过访问属性func来调用的，函数中的this指向属性func的所有者，即foo对象
foo.func(); // foo

var exampleFunc = bar.test;
// 通过标示符exampleFunc来调用的，函数中的this指向全局对象
exampleFunc(); // global
```
全局函数的例子：
```javascript
function foo() {
  console.log(this);
}

// 通过标示符foo来调用的，函数中的this指向全局对象
foo(); // global
 
console.log(foo === foo.prototype.constructor); // true
// 通过访问属性foo.prototype.constructor来调用的，函数中的this指向属性constructor的所有者，即foo.prototype对象
foo.prototype.constructor(); // foo.prototype
```
既不是标识符也不是属性访问来调用的例子：
```javascript
(function () {
  // this在函数代码中的其余情况，this都指向全局对象
  console.log(this); // global
})();
```
一个更为复杂的例子：
```javascript
var foo = {
  bar: function () {
    console.log(this);
  }
};

// 通过访问属性bar来调用的，函数中的this指向属性bar的所有者，即foo对象
foo.bar(); // foo

// ()是组操作符，不会影响()内的值，跟foo.bar()是一样的
(foo.bar)(); // foo

// 赋值操作符、逻辑运算符、逗号操作符 最终都是返回了函数对象，相当于直接调用函数对象<Function: bar>()
(foo.bar = foo.bar)(); // global
(false || foo.bar)(); // global
(foo.bar, foo.bar)(); // global
```
内部子函数在父函数中被调用的例子：
```javascript
function foo() {
  function bar() {
    console.log(this); // global
  }
  bar(); // 通过标示符bar来调用的，函数中的this指向全局对象
}
```
try-catch的例子：
```javascript
try {
  throw function () {
    console.log(this);
  };
} catch (e) {
  e(); // 通过标示符e来调用的，函数中的this指向全局对象
}
```
再来看一个递归调用的例子：
```javascript
(function foo(bar) {
 
  console.log(this);
 
  !bar && foo(1); // 此处，通过标示符foo来调用的，函数中的this指向全局对象
 
})(); // 此处，既不是标识符也不是属性访问来调用，函数中的this都指向全局对象
```
构造函数中的例子：
因为new操作符是对一系列操作的封装，内部使用了apply或call的方式显式指定this。
关于new操作符，可以看 [这篇博客]({% post_url 2018-05-28-javascript-basic-concepts %}) 。
```javascript
function A() {
  console.log(this); // a
  this.x = 10;
}

// 相当于显式指定this指向新创建的对象a
var a = new A();
console.log(a.x); // 10
```
   
----

###  参考链接
[ECMA-262-3 in detail. Chapter 3. This.](http://dmitrysoshnikov.com/ecmascript/chapter-3-this/)

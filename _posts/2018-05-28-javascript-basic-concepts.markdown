---
layout: post
title:  "JavaScript语言的几个基本概念"
date:   2018-05-28 20:34:53 +0800
categories: jekyll update
---
JavaScript是面向对象，基于“原型”的解释型语言。

# 对象（Object）

一个对象就是一系列属性的集合。
```javascript
// 例如一个简单的point对象：
let point = {
  x: 10,
  y: 20,
};
```

# 原型（Prototype）
为了实现对象之间的‘继承’关系，引入了‘原型’的概念。
每个对象都隐式含有自己的‘原型’，可以是另一个对象或者null，默认值为Object.prototype。用属性'\_\_proto\_\_'表示。
```javascript
// point对象，隐式含有值为Object.prototype的__proto__属性
let point = {
  x: 10,
  y: 20,
};

// 显示指定point3D的‘原型’为point对象，即point3D继承了point
let point3D = {
  z: 30,
  __proto__: point,
};

console.log(
  point3D.x, // 10, 继承了point的x属性
  point3D.y, // 20, 继承了point的y属性
  point3D.z  // 30, 自己的z属性
);
```
对象有‘原型’；‘原型’本身也是对象，也有自己的‘原型’；由此就产生了一条‘原型链’，直到\_\_proto\_\_是null为止。
point3D的例子可以表示为下图：

![prototype-chain](/assets/images/prototype-chain.png)
# 类（Class）
‘类’是JavaScript对‘原型’机制的一种‘语法糖’封装。
```javascript
// 类的一个例子
class Letter {
  constructor(number) {
    this.number = number;
  }

  getNumber() {
    return this.number;
  }
}

let a = new Letter(1);
let b = new Letter(2);
let z = new Letter(26);

console.log(
  a.getNumber(), // 1
  b.getNumber(), // 2
  z.getNumber(), // 26
);
```
还原成‘类型’机制的写法大致就是：
```javascript
// Letter是构造函数
function Letter(number) {
  this.number = number;
}

// Letter的原型
let prototypeOfLetter = {
  getNumber: function() {
    return this.number;
  }
}
Letter.prototype = prototypeOfLetter

// new操作的伪码实现
function pseudoNew(f, ...args) {
  var obj = {};
  obj.__proto__ = f.prototype;
  obj.__proto__.constructor = f;
  var ret = f.apply(obj, args);
  return typeof ret === 'object' ? ret : obj;
}

let a = pseudoNew(Letter, 1);
let b = pseudoNew(Letter, 2);
let z = pseudoNew(Letter, 26);

console.log(
  a.getNumber(), // 1
  b.getNumber(), // 2
  z.getNumber(), // 26
  Letter.prototype === prototypeOfLetter, // true
  prototypeOfLetter.constructor === Letter, // true
  a.constructor === Letter, // true
);
```
class封装了构造函数和原型；而new调用构造函数生成一个继承于该原型的新对象。
构造函数用隐式属性prototype指向‘原型’；‘原型’用隐式属性constructor指向构造函数。
如下图：

![prototype-chain](/assets/images/js-constructor.png)
# 函数（Function）
JavaScript中，函数是一等公民（first-class function）。函数可以作为其他函数的参数或者返回值，赋值给变量或存储在数据结构中。这将会遇到两个‘函数类型参数问题’（funarg problem）。
### 下传函数类型参数（downwards funarg problem）
```javascript
let x = 10;

function foo() {
  console.log(x);
}

function bar(funArg) {
  let x = 20;
  funArg(); // 结果是10, 不是 20！
}

// 将'foo'作为参数传给'bar'。
bar(foo);
```
在foo函数中，x既不是内部变量，也不是函数参数，而是作为**‘自由变量’**出现。
最后结果是10，说明函数是**‘静态作用域’（Static scope）**或者叫**‘词法作用域’（lexical scope）**。即函数**‘自由变量’**的引用取决于函数定义的时候，而跟函数的调用没有关系。
### 上传函数类型参数（upwards funarg problem）
这个涉及到**‘闭包’（Closure）**的问题：
```javascript
function foo() {
  let x = 10;

  // 闭包, 捕获了`foo`的执行环境。
  function bar() {
    return x;
  }

  // 上传函数类型参数。
  return bar;
}

let x = 20;

// 调用`foo`返回`bar`闭包。
let bar = foo();

bar(); // 结果是10, 不是 20！
```
函数在执行的时候，会创建一个新的**执行环境（activation environment）**用于存储本地变量和函数参数。
在‘下传函数类型参数问题’中，调用foo时创建的**执行环境**，在foo运行结束后就被销毁了。
在‘上传函数类型参数问题’中，bar捕获了foo创建的**执行环境**，并向上传递给了foo的调用者。又因为是**‘静态作用域’**，所以foo创建的**执行环境**，在foo运行结束后将不会销毁而保存起来。
# This
可以将this看作是一个只能读取，不能修改的隐式传递的参数。  
this主要用于基于类的面向对象编程（class-based OOP），例子如下：
```javascript
class Point {
  constructor(x, y) {
    this._x = x;
    this._y = y;
  }

  getX() {
    return this._x;
  }

  getY() {
    return this._y;
  }
}

let p1 = new Point(1, 2);
let p2 = new Point(3, 4);

// 类Point的两个实例p1，p2都能访问'getX'，'getY'（被当作this传递）
console.log(
  p1.getX(), // 1
  p2.getX(), // 3
);
```

this是**动态作用域（dynamic scopes）**，即this的值取决于调用方式。
分为几种情况：
### 全局代码中this的值
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
###  函数代码中This的值
```javascript
var foo = {x: 10};
var bar = {
  x: 20,
  test: function () {
    console.log(this);
    console.log(this.x);
  }
};
bar.test(); // bar, 20
bar.test.prototype.constructor() // bar.test.prototype, undefined

var exampleFunc = bar.test;
exampleFunc(); // global, undefined

foo.test = bar.test;
foo.test(); // foo, 10
```
###  手动设置函数调用时this的值(构造函数中的情况类似)
```javascript
var b = 10;
function a(c) {
  console.log(this.b);
  console.log(c);
}

a(20); // this === global, this.b == 10, c == 20

a.call({b: 20}, 30); // this === {b: 20}, this.b == 20, c == 30
a.apply({b: 30}, [40]) // this === {b: 30}, this.b == 30, c == 40
```  
  
  
------------------  
  
  
###  参考链接
[JavaScript. The Core: 2nd Edition](http://dmitrysoshnikov.com/ecmascript/javascript-the-core-2nd-edition/#execution-context)  
[ECMA-262-3 in detail. Chapter 3. This.](http://dmitrysoshnikov.com/ecmascript/chapter-3-this/)

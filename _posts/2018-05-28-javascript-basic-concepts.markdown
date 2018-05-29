---
layout: post
title:  "JavaScript语言的几个基本概念"
date:   2018-05-28 20:34:53 +0800
categories: jekyll update
---
JavaScript是面向对象，基于“原型”的解释型语言。  
ES6引入Class，Class的继承，实际上可以理解为语法糖。虽然看起来更清晰和方便了，但对理解语言本身却带来了不少麻烦。  
下面是对几个基础概念的整理。

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

---
layout: post
title:  "JavaScript的执行上下文（Execution Contexts）、变量对象（Variable Object）"
date:   2018-05-29 21:11:42 +0800
categories: jekyll update
---
代码在执行的时候，总会有一个对应的代码执行环境，称之为执行上下文（Execution Contexts，以下简称EC）。   
一般来说，有三种类型的代码：全局代码，函数代码，Eval代码。   
不同类型的代码，对应不同类型的EC。   
程序一开始运行，就有了全局EC。每当进入函数时，会创建一个新的函数EC，该函数又可以调用其他函数（或递归调用自己），创建新的更多函数EC。   
由此形成一个EC栈，栈底总是全局EC，调用函数（或执行eval）时，往上push一个新的函数EC。函数执行完（或抛出未捕获的异常），pop出该函数EC。   
例如执行下面会递归调用自身的函数：
```javascript
(function foo(flag) {
  if (flag) {
    return;
  }
  foo(true);
})(false);
```
EC栈的变化情况：
```javascript
// 第一次执行foo
ECStack = [
 <foo> functionContext,
 globalContext
];
// 递归调用foo
ECStack = [
 <foo> functionContext - recursively 
 <foo> functionContext
 globalContext
];
```
#  EC有什么用呢
代码所声明的变量、函数，以及函数的形参，可以理解为存储在一个称为变量对象（Variable Object，以下简称VO）的结构中,而VO正是EC的一个属性。   
```javascript
ExecutionContext = {
 VO: {
   // 上下文中的数据 (变量声明, 函数声明, 函数形参)
 }
};
```
这就是变量的申明、存储、查找的机制。   
#  全局EC中的VO
全局EC中的VO就是全局对象（Global object）。全局对象是在进入任何EC前就创建出来的；以单例形式存在；它的属性在程序任何地方都可以直接访问，其生命周期随着程序的结束而终止。   
在引用全局对象的属性时，前缀通常可以省略。通过全局代码上的this值，以及通过如DOM中的window对象这样递归引用的方式都可以访问到全局对象： 
```javascript
String(10); // 等同于 global.String(10);
// 带前缀
window.a = 10; // === global.window.a = 10 === global.a = 10;
this.b = 20; // global.b = 20;
```
#  函数EC中的VO
函数EC中的VO，即所谓的活跃对象（Activation Object，以下简称AO）。AO是进入函数EC时创建的。
#  处理EC的两个阶段
不管是处理全局EC，还是处理函数EC，都分为两个阶段：   
1.进入EC  
2.执行代码  
##  进入EC
一旦进入EC，在执行代码前，VO中的以下属性就会被填充：   
> * 函数的形参（当进入函数EC时）  
--其属性名就是形参的名字，其值就是实参的值；对于没有传递的实数，其值为undefined。
> * 函数声明   
--其属性名就是函数名,其值就是函数对象；如果VO已经包含了相同名字的属性，则覆盖它的值为函数对象。
> * 变量声明   
--其属性名就是变量名，其值统一初始化为undefined。如果VO已经包含了相同名字的形参或函数声明，则不会影响已经存在的属性。   
  
##  执行代码
此时，VO中的属性已经填充好，尽管大部分属性都还没有被赋予真正的值，都只是初始化时候的undefined。接着就开始执行代码。   
看下面这个例子：   
```javascript
function test(a, b) {
  if (false) {
    var f = 30;
  }
  var c = 10;
  function d() {}
  console.log(e); // undefined
  var e = function _e() {};
  console.log(e); // <reference to FunctionDeclaration "_e">
  (function y() {});

  console.log(x); // <reference to FunctionDeclaration "x">
  var x = 10;
  console.log(x); // 10
  x = 20;
  function x() {}
  console.log(x); // 20
}

test(10); // 调用test()
```
当以10为参数进入“test”函数EC的时候，即进入EC阶段，对应的AO如下所示：  
```javascript
AO(test) = {
 f: undefined,
 a: 10,
 b: undefined,
 c: undefined,
 d: <reference to FunctionDeclaration "d">,
 e: undefined,
 x: <reference to FunctionDeclaration "x">,
};
```
注意：   
虽然if语句从不会执行，但是AO仍然包含f，其值被初始化为undefined。   
AO并不包含函数y和函数_e,因为它们都是函数表达式，而不是函数声明。只是_e被赋值给了e，即可以通过e来访问函数_e。   
在test()中的任何位置访问y（或者_e），都会提示出错: y(或者_e) is not defined。   
   
接着就是执行代码阶段：
```javascript
AO['c'] = 10
AO['e'] = <reference to FunctionDeclaration "_e">
AO['x'] = 10 //执行到第一条console.log(x)语句之后，AO的x属性的值被修改为10
AO['x'] = 20
```
为什么第一条console.log(x)语句的输出指向函数x，为什么不是undefined或者10？因为在进入EC阶段，x就已作为函数声明填充进AO，并赋值为函数x的引用。尽管存在相同名字的x变量声明，但并不会影响已存在的x函数申明。   

----  
  
  
###  参考链接
[ECMA-262-3 in detail. Chapter 1. Execution Contexts.](http://dmitrysoshnikov.com/ecmascript/chapter-1-execution-contexts/)  
[ECMA-262-3 in detail. Chapter 2. Variable object.](http://dmitrysoshnikov.com/ecmascript/chapter-2-variable-object/)

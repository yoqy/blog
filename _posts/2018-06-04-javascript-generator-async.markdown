---
layout: post
title:  "JavaScript的Generator异步操作"
date:   2018-06-04 20:21:57 +0800
categories: jekyll update
---
#   异步操作
JavaScript必然会遇到异步操作，例如有这么一个需求：从文件file1中读取数据，数据内容是下一个文件的文件名，再读取并输出下一个文件的内容。以Generator的方式实现，代码如下：
```javascript
var fs = require('fs');

function readFileThunkify(file) {
  return function(done) {
    fs.readFile(file.replace('\r', '').replace('\n', ''), 'utf8', done);
  }
}

function* gen() {
  let r1 = yield readFileThunkify('file1');
  let r2 = yield readFileThunkify(r1);
  console.log(r2);
};
```
readFileThunkify()是将读取文件接口readFile()封装为Thunk函数，gen()是Generator函数，封装了两个读取文件的异步操作。要求第二个文件名是从第一个文件里读取出来的，就需要等第一个读取操作执行完成后，再执行第二个读取操作。因此Generator函数的执行代码如下：
```javascript
function run() {
  var g = gen();
  var r1 = g.next();
  r1.value(function(err, data){
    var r2 = g.next(data);
    r2.value(function(err, data){
      console.log(data);
    })
  });
}

run();
```
也可以用类似CO模块的方式自动执行Generator函数，下面是CO模块实现原理的伪码：
```javascript
function fakeCo(fn) {
  var gen = fn();

  function next(err, data) {
    var result = gen.next(data);
    if (result.done) {
      return
    }
    result.value(next);
  }

  next(null, gen.value);
}

fakeCo(gen);
```
#   Generator黑魔法   
JavaScript是单线程运行的，那如何用Generator和yield实现异步操作的呢？   
Generator最初是用于便利的产生迭代器的工具，从名字‘Generator’和‘yield’就能看出来。典型应用是这样的：
```javascript
function* idMaker() {
  var index = 0;
  while(true)
    yield index++;
}

var gen = idMaker();

console.log(gen.next().value); // 0
console.log(gen.next().value); // 1
console.log(gen.next().value); // 2
// ...
```
Generator是用状态机的实现的，通过这个[在线地址](http://www.alloyteam.com/2016/02/generators-in-depth/)，可以将上面的idMaker()转换为：
```javascript
var _marked = /*#__PURE__*/regeneratorRuntime.mark(idMaker);

function idMaker() {
  var index;
  return regeneratorRuntime.wrap(function idMaker$(_context) {
    while (1) {
      switch (_context.prev = _context.next) {
        case 0:
          index = 0;

        case 1:
          if (!true) {
            _context.next = 6;
            break;
          }

          _context.next = 4;
          return index++;

        case 4:
          _context.next = 1;
          break;

        case 6:
        case "end":
          return _context.stop();
      }
    }
  }, _marked, this);
}
```
#   Generator异步操作原理
将开始例子中的gen函数转换为不含Generator的方式，再简化，就可以看到Generator异步操作的原理：
```javascript
function generate() {
  var _context = {
    next: 0
  };

  return {
    next: function(done) {
      switch(_context.next) {
        case 0 :
          _context.next = 1;
          return {
            done: false,
            value: readFileThunkify('file1')
          }
        case 1:
          _context.next = 2;
          return {
            done: false,
            value: readFileThunkify(done)
          }
        case 2:
          console.log(done);
          return {
            done: true,
            value: done
          }
      }
    }
  }
}
```
generate以闭包的方式，生成了一个状态机，用_context维护上下文状态。   
上个状态通过next()返回，下个状态的初始值也可以用参数通过next(done)传入。正好用前后两个状态，对应了异步操作的执行阶段和完成阶段，很巧妙。   
   
    
    
----

###  参考链接
[深入理解Generators](http://www.alloyteam.com/2016/02/generators-in-depth/)
[Iterators and generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators)

title: '[JavaScript] 链式调用学习'
author:
  name: 石头
  work: Web Developer &amp; Designer
  location: 'Beijing, China'
tags: []
categories:
  - front-end
  - ''
date: 2019-10-13 10:09:00
---
### 什么是链式调用？
链式调用（operate of responsibility）是一种很常见的代码设计。

像Jquery：
```javascript
$('div').on('click', callback)
           .on('mousedown', callback);
```

d3.js:
```javascript
d3.select(DOM.svg(width, height * years.length))
      .style("font", "10px sans-serif")
      .style("width", "100%")
      .style("height", "auto");
```
---

### 链式调用的优势
1. 美观：提高代码优雅度 操作语法化 增加可阅读性
2. 简洁：通过链式调用显得代码更加简洁
3. 提效：简化对同一对象多个方法多次调用的方式
4. 回调地狱的解决方案, [例子](https://medium.com/backticks-tildes/understanding-method-chaining-in-javascript-647a9004bd4f)
---

### 实现链式调用
这里我们借鉴jQuery、d3.js和`《JavaScript设计模式》第 27 章 - 永无尽头——链模式`的思路，

#### - jQuery 基本实现方式

该例子已对构造器进行强化
```javascript
var A = function (selector){
  return new A.fn.init(selector)
};
A.fn = A.prototype = {
  // 强化构造器
  constructor: A,
  init: function(selector) {
    const list = document.querySelectorAll(selector);
    list.forEach((item, index) => {
      this[index] = item;
    })
    this.length = list.length;
    return this;
  },
  length: 0,
  size: function() {
    return this.length
  }
}
// 强化构造器
A.fn.init.prototype = A.fn
```

具体使用：
```javascript
A('div').size();
/* 
  > Number
*/
```

#### - d3.js的实现方式

这是改编于d3js的代码
```javascript
function func1 () {
  console.log('func1');
  return this;
}

function getArg () {
  return this.arg;
}

// -- 底层 -- //
function A(arg) {
  this.arg = arg;
}

// -- 用于 instenceOf 判断 -- //
function _A(){
  return new A();
}

// -- 强化构造器 -- //
A.prototype = _A.prototype = {
  constructor: A,
  func1: func1,
  getArg: getArg
}

// -- 二层封装 -- //
function B(arg) {
  return new A(arg)
}
```

具体使用：
```javascript
B('arg').func1().getArg();
/* 
  > 'func1'
  > 'arg'
*/
```




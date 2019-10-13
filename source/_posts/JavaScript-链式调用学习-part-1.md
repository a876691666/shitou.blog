title: '[JavaScript] 链式调用学习 - part 1'
author:
  name: 石头
  work: Web Developer &amp; Designer
  location: 'Beijing, China'
date: 2019-10-13 10:09:35
tags:
---
### 关于链式调用
链式调用是一种很常见的代码设计。

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
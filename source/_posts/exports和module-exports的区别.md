---
title: exports和module.exports的区别
date: 2017-07-06 17:46:54
tags: 
---

之前一直比较困惑exports和module.exports有什么区别。这里做一个比较详细的解释
<!--more-->

首先我们来看一段代码：

```js
var a = {name: 1};
var b = a;

console.log(a);
console.log(b);

b.name = 2;
console.log(a);
console.log(b);

var b = {name: 3};
console.log(a);
console.log(b);
```

运行 test.js 结果为：
```
{ name: 1 }
{ name: 1 }
{ name: 2 }
{ name: 2 }
{ name: 2 }
{ name: 3 }
```
**解释** ：a 是一个对象，b 是对 a 的引用，即 a 和 b 指向同一块内存，所以前两个输出一样。当对 b 作修改时，即 a 和 b 指向同一块内存地址的内容发生了改变，所以 a 也会体现出来，所以第三四个输出一样。当 b 被覆盖时，b 指向了一块新的内存，a 还是指向原来的内存，所以最后两个输出不一样。

明白了上述例子后，我们只需知道三点就知道 exports 和 module.exports 的区别了：

- `module.exports` 初始值为一个空对象 {}

- `exports` 是指向的 `module.exports` 的引用

- `require()` 返回的是 `module.exports` 而不是 `exports`

- `exports.xxx`，相当于在导出对象上挂属性，该属性对调用模块直接可见

- `exports =` 相当于给exports对象重新赋值，调用模块不能访问exports对象及其属性,因为require返回的是`module.exports`

- 如果此模块是一个类，就应该直接赋值module.exports，这样调用者就是一个类构造器，可以直接new实例；如果要导出一个模块实例，那么就直接挂在exports上

- 如果module.exports已经存在(即使是module.exports = {})，那么exports上面添加的任何属性都将被忽略


```js
module.exports = 'ROCK IT!';
exports.name = function() {
    console.log('My name is Lemmy Kilmister');
};
```
```js
var rocker = require('./rocker.js');
rocker.name(); // TypeError: Object ROCK IT! has no method 'name'
```

## 参考文章

[Node.js Module – exports vs module.exports](http://www.hacksparrow.com/node-js-exports-vs-module-exports.html)

[Node.js module.exports与exports](https://github.com/chemdemo/chemdemo.github.io/issues/2)

[exports 和 module.exports 的区别](https://cnodejs.org/topic/5231a630101e574521e45ef8)


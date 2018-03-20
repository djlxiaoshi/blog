---
title: JavaScript模块加载 - AMD
date: 2017-04-04 17:58:55
tags:
    - AMD
    - require.js
---

随着js代码的增加，将所有的js代码放在一个js文件里面，会使得这个js代码非常臃肿，所以我们需要把js代码模块化。于是我们会按如下方法来做：
<!--more-->

```js
<script src="a.js"></script>
<script src="b.js"></script>
<script src="c.js"></script>
<script src="d.js"></script>
```

这样做有两个缺陷：1 随着js文件的增多，加载js文件会阻塞浏览器的渲染；2 这些js文件之间可能存在严格的依赖关系，导致js文件的加载必须遵从顺序。

为了解决这个问题，JavaScript引入了模块的概念，目前分为浏览器端和服务器端。浏览器端模块加载规范一般有AMD（异步加载js文件）和CMD（同步加载js文件），服务器端就是common.js规范。而一般情况下，浏览器端更适合使用异步加载，因为在浏览器端，js文件的加载取决于网络状况，采用异步加载，可以避免因为网络问题带来的浏览器阻塞问题。在这里将简单介绍[require.js](http://requirejs.org/),AMD规范的一种实现方式。

# 引入require.js

首先我们在页面中引入require.js，并且设置data-main属性。

```js
<script src="js/require.js" data-main="js/main"></script>
```
其中data-main属性指定所有js文件的一个入口。为了在引入require.js的时候不阻塞浏览器的渲染，我们可以将`script`标签放在body底部，当然也可以利用script异步加载属性。
```js
<script src="js/require.js" defer async="true" ></script>
```
`defer`是为了兼容IE

# 引入模块

现在我们就可以在main.js入口文件中书写自己的js代码了，一般情况下入口文件会依赖很多其他模块，那么我们在使用时需要引入这些模块：
```js
require(['a', 'b'],function (a, b) {
    alert(a.add(1,2))   // 3
    alert(b.dec(1,2))   // -1
})
```
根据AMD规范引入模块的写法，我们在main.js这个模块中引入了a.js和b.js这两个模块。这两个模块和main.js在同一个目录之下，但是如果不在一个目录中，那么我们就需要通过require.config定义模块路径：

```js
require.config({
    paths: {
        'a': 'test/a',
        'b': 'test/b'
    }
})

require(['a', 'b'],function (a, b) {
    window.alert(a.add(1,2))
    window.alert(b.dec(1,2))
})
```
require.config要放在main.js的头部。当然也可以改变js文件路径的基目录
```js
require.config({
    baseUrl:'js/lib',
    paths: {
        'a': 'a',
        'b': 'b'
    }
})

require(['a', 'b'],function (a, b) {
    window.alert(a.add(1,2))
    window.alert(b.dec(1,2))
})
```

# 定义模块

根据AMD规范，我们定义模块要是`define`。例如a.js

## 定义键值对

如果一个模块不依赖于其他模块并且只是一些键值对的容器

```js
define({
    color: "black",
    size: "unisize"
});
```

## 定义函数

如果一个模块不依赖于其他模块，但是又要做一些设置工作。
```js
define(function () {
    //Do setup work here

    return {
        color: "black",
        size: "unisize"
    }
});
```

## 定义有依赖的函数
一般情况下，一个模块都会依赖一个或者多个其他模块，在定义这些模块时，我们要首先引入它所依赖的模块。
```js
define(["a"], function(a) {
    return {
       result : a.add(1 + 2)
    }
});
```

# 加载不符合AMD规范的模块

有一些模块是不符合AMD规范的，我们在使用的时候，要进行如下配置
```js
require.config({
　　shim: {
　　　'underscore':{
　　　　　exports: '_'
　　　 },
　　   'backbone': {
　　　　　　deps: ['underscore', 'jquery'],
　　　　　　exports: 'Backbone'
　　　　}
　　}
});
```
即要在shim属性中定义这个模块导出后的变量名和依赖。

以上介绍的是require.js的最基本用法，当然目前也就用到了这么多。

# 参考文章

[Javascript模块化编程（三）：require.js的用法](http://www.ruanyifeng.com/blog/2012/11/require_js.html)
[require.js](http://requirejs.org/)





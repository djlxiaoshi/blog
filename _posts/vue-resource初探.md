---
title: vue-resource初探
date: 2017-02-12 16:33:33
tags: 
    - vue
    - vue-resource
---

# 简介
vue-resource是用来发送ajax请求或者解决jsonp跨域问题，既可以在全局Vue中使用（即Vue.$http）也可以在vue实例中使用（即this.$http）
<!--more-->

# 安装
npm `$ npm install vue-resource`
cdn  `<script src="https://cdn.jsdelivr.net/vue.resource/1.0.3/vue-resource.min.js"></script>`

# 基本用法
最常见的用法，目前还需要掌握这些
```js
  // GET /someUrl
  this.$http.get('/someUrl').then((response) => {
    // success callback
  }, (response) => {
    // error callback
  });
```
如果没有es6解析环境就不要写成箭头函数的形式

```js
// global Vue object
Vue.http.get('/someUrl', [options]).then(successCallback, errorCallback);
Vue.http.post('/someUrl', [body], [options]).then(successCallback, errorCallback);
```
也可以vue实例中使用
```js
// in a Vue instance
this.$http.get('/someUrl', [options]).then(successCallback, errorCallback);
this.$http.post('/someUrl', [body], [options]).then(successCallback, errorCallback);
```

一般我们请求的数据都会是一个对象或者字符串，我们通过response.body来获取响应内容，如果我们请求的是图片这样的二进制文件，我们通过response.blob来获取内容主题。

```js
{
  // POST /someUrl
  this.$http.post('/someUrl', {foo: 'bar'}).then(response => {
  
    // get body data
    this.someData = response.body;

  }, response => {
    // error callback
  });
}
```
或者图片等等二进制大文件

```js
  this.$http.get('/image.jpg').then(response => {
    // resolve to Blob
    return response.blob();
    
    }).then(blob => {
    // use image Blob
    });  
```

[response对象](https://github.com/pagekit/vue-resource/blob/develop/docs/http.md)

# 学习文档
[官方文档](https://github.com/pagekit/vue-resource)
[参考资料](http://www.cnblogs.com/keepfool/p/5657065.html)





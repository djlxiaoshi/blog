---
title: angular1学习之directive
date: 2017-04-06 18:53:24
tags:
    - angular
    - directive
---

作为只学过Vue的前端来说，angular的概念还真是比较多的，但是每接触一个概念我都会和Vue相关概念做一个对比，这样的对比对我理解angular起了非常大的作用。这节主要讲解directive，类比Vue就好比Vue里面的组件。
<!--more-->

# 自定义一个指令

首先angular定义一个指令的方式如下：
```js
var app = angular.module('container',[])
app.controller('wrap', function ($scope) {
    $scope.nickName = 'DJL箫氏'
})
app.directive('myDirective',function() {
    return {
        restrict: 'A',
        template : '<h1>自定义指令{{nickName}}</h1>'
    }
})
```
我们通过app.directive方法来创建指令，指令名称遵从驼峰命名规则，在使用的时候用`-`分开
```html
<div my-directive></div>
```

作为刚学angular的人来说，为什么有时候使用 
```js
app.controller('wrap', function ($scope) {
    $scope.nickName = 'DJL箫氏'
})
```
有时候
```js
app.controller('wrap', ['$scope', function ($scope) {
    $scope.nickName = 'DJL箫氏'
}])
```
如果服务名和function中的参数名一致，则可以不写成数组的形式，如果要更换名字则写成数组的形式。例如：
```js
app.controller('wrap', ['$scope','myService', function (a,b) {
    $scope.nickName = 'DJL箫氏'
}])
```
那么a即`$scope`服务，b即`myService`服务

# 常用属性介绍
## restrict
自定义指令会返回一个对象，里面有很多属性`restrict:A`属性表示在html代码中调用这个属性只能通过属性的形式来调用，当然值还有E（只能通过标签元素的方式调用）,C(只能通过class的方式调用)，EAC（以上三种方式都可以）
```html
<div my-directive></div>  
<my-directive></my-directive>
<div class="my-directive"></div>
```

## template & templateUrl
template属性则是这个指令的DOM结构，也就是说指令适用于与DOM有关的场景。我们可以看到在指令中可以拿到包含它的controller中的数据（$scope）。当然如果指令的DOM结构非常复杂，这个时候我们可以使用templateUrl属性来替代template属性，templateUrl的值为dom模板的地址。当然我们可以使用`ng-template`
```
<script type="text/ng-template" id="part.html">
    <h1>通过ng-template封装的part.html</h1>
    <p>这里是part.html中的内容</p>
</script>
```
```js
templateUrl: 'part.html'
```

## replace
该属性表明这个指令是直接插入到DOM结构中（值为TRUE），还是先把`<my-directive></my-directive>`这个标签去除后只插入指令中template或者templateUrl中的内容。
![值为false](http://images.djl.pub/17-4-6/52136146-file_1491473056558_dddc.png)
![值为true](http://images.djl.pub/17-4-6/30517686-file_1491472930303_7e1e.png)

## scope
在前面我们知道directive可以直接拿到所在controller中的数据，但是如果在同一个controller中有多个directive，我们希望通过传入directive不同的值来展示不同的结果，这时我们就要使用scope属性，这个属性类似于Vue组件中props属性。
```html
<my-directive user="userA"></my-directive>
```
```js
scope:{
    user: "="
},
template : "<p>姓名:{{user.name}}</p><p>性别：{{user.sex}}</p>"

App.controller("wrap", function ($scope) {
    $scope.userA = {
        name: "DJL箫氏",
        sex : "男"
    };
});
```
在这里`user: '='`,其中‘=’表示传入一个引用值（如果在directive中改变数据，controller中的数据也会跟随改变），当然值还可以是‘@’（传入一个字符串，那么此时传入的就是userA这个字符串了），&（传入一个回调函数）

## link
link属性就类似于Vue里面的生命周期钩子函数，是会自动执行的，所以可以把一些初始化任务放在里面。link函数有3个参数scope（暂时理解为directive中scope属性）,element（指令中template对应的DOM）,attrs。

由于Angular几乎照搬了jQuery的DOM操作源码，所以我们在操作DOM时，可以基本舒勇jQuery的语法，当然如果我们要使用jQuery来操作也非常简单，我们只需要安装jQuery然后在index.html中引入jQuery（注意顺序）
```js
<!-- 保证在angular.js之前引入jquery.js -->
<script type="text/javascript" src="components/jquery/dist/jquery.js"></script>
<script type="text/javascript" src="components/angular/angular.js"></script>
```

## transclude
有时候我们想在指令中添加一些自定义的元素，例如：
```html
<my-directive user="userA">
    <h1>hello world</h1>
</my-directive>
```
这时我们就需要设置transclude属性值为true。
同时在指令的DOM结构合适的位置中加入`<ng-transclude></ng-transclude>`或者以属性的方式`<div ng-transclude></div>`
```js
template : '<h1>自定义指令<ng-transclude></ng-transclude></h1>'
```

所以我们会发现ng-app这些指令的transclude属性都设置成了true

ok 目前常用的就是这些，下节会讲directive之间的通信问题。
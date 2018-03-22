---
title: angular1学习之控制器通信
date: 2017-04-07 17:26:22
tags:
    - angular
    - 转载
---

指令与控制器之间通信，无非是以下几种方法：

- 基于scope继承的方式
- 基于event传播的方式
- service的方式
 <!--more-->

# 基于scope继承的方式

最简单的让控制器之间进行通信的方法是通过scope的继承。假设有两个控制器Parent、Child，Child 在 Parent 内，那Child 可以称为控制器Parent的子控制器，它将继承父控制器Parent的scope。这样，Child就可以访问到Parent的scope中的所有函数和变量了。

需要注意的是，由于scope的继承是基于Js的原型继承，如果变量是基本类型的，那在Child中的修改（写），有可能会导致Parent中的数据变脏。如下：

基本类型以及引用类型变量的继承

此[DEMO](http://jsbin.com/qezot/4/edit?html,js,output)代码中我们看到，两个_value，其中一个 _value 属性是直接被注册到\$scope 中，另一个 _value 是注册到 parent 控制的 $scope.obj 中，DEMO效果如下：

1 child能读取到parent中的_value值，所以默认页面 显示的是4个 default值

2 如果先改变了直接注册在child上 $scope 上的 _value 属性，则直接注册在 parent.$scope 的 _value 跟直接注册在 chile.$scope的_value失去了联系，页面上的表现：就是如果先点击了child的按钮，点击parent的按钮 child.$scope 上的 _value 则不会变化。

3 反过来，如果未对直接注册在 chile.$scope 的_value进行改写，则注册在 parent.$scope 的 _value 跟chile.$scope 的 _value还有联系，页面上表现跟以上相反。

4 而注册在 obj 上的 _value 属性，则一直是有联系的。

经过以上实验，我们得出一下结论：

> **子级scope改写的属性不要直接注册在$scope对象上，而应该尽可能注册在$scope上的引用类型上，以免污染$scope。**

# 基于event传播的方式

通过 scope 继承 能处理父子级控制器之间的通信问题，但是不能处理兄弟/相邻控制器之间的通信问题。而基于 event 传递的方式进行通信可以解决父子级的通信问题。angular提供了三个方法：$on , $emit , $broadcast

## 子–>父：$emit

event传播过程是这样的：

子scope中的控制器通过 $scope.$emit注册一个向上传播的事件,该事件会经过每一层的父scope，但是每一层父scope不会去处理,如果要处理就在想要处理的父scope中使用$scope.$on监听[DEMO](http://jsbin.com/goxiw/5/edit?html,js,output)。跟JS中的DOM事件一样，如果你不想让你的事件再往更上层传播，在$on中的处理函数调用e.stopPropagation()即可。

## 父–>子：$broadcast

从父到子，跟子集到父级一样，使用同样用$broadcast注册时间，用 $on 监听着，[DEMO](http://jsbin.com/gidomu/10/edit?html,js,output)。

## 同级之间

拥有同个父scope的子级scope之间，也就是兄弟/相邻scope之间的通信，其实是借助共同父级传递消息的：

子级scope中有谁想传消息了，$emit一个给“奶爸”,然后通过“奶爸”$broadcast给所有孩子这个相同的信息，当然发出信息的那个可以选择是否要忽略掉自己发出的信息

# angular服务的方式

在angular中服务是一个单例，在服务中生成一个对象，该对象就可以利用依赖注入的方式在所有的控制器中共享。参照以下例子，在一个控制器修改了服务对象的值，在另一个相邻控制器中获取到修改后的值：

[DEMO](http://jsbin.com/hopazo/5/edit?html,css,js,output)

**特别注明：此文为转载，原文链接 [AngularJs开发——控制器间的通信](http://www.html5jscss.com/angular-between-controller.html)**





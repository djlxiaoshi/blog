---
title: Vue学习之生命周期
date: 2017-01-21 16:19:44
tags: 
    - Vue
    - 生命周期
categories: Vue学习

---

生命周期在组件开发总非常重要，尤其是能在适当的时机取得数据,做一些初始化准备。下面我们来简单的聊聊Vue生命周期。
<!--more-->
这里首先来看看Vue官方文档给出的生命周期示意图。

![Vue生命周期](http://images.djl.pub/17-1-21/75404643-file_1484986273994_1820a.png)

在这里我主要讨论init、created、beforeCompile、compiled、ready、beforeDestroy、destroyed这几个生命周期钩子函数。

init：在实例开始初始化时同步调用。此时数据观测、事件和 watcher 都尚未初始化。也是说在这里this.$el和this.$data都是拿不到的。

created:在实例创建之后同步调用。此时实例已经结束解析选项，这意味着已建立：数据绑定，计算属性，方法，watcher/事件回调。但是还没有开始 DOM 编译，$el还不存在。也就是在这里已经可以访问this.$data了了。但是此时还没有进行模板编译。

beforeCompile：编译模板前，$el已经存在了。

compiled：模板编译完成，此时Vue指令已经生效，但是不能保证此时已经插入到文档中。

ready：在编译结束后和$el第一次插入到文档中的时候。这时可以从整个文档中获取到模板中的元素。

beforeDestroy：在销毁实例时调用，所有功能都还保持。

destroyed：在实例被销毁之后调用。此时所有的绑定和实例的指令已经解绑，所有的子实例也已经被销毁。如果有离开过渡，destroyed 钩子在过渡完成之后调用。与挂载元素解除绑定

代码：
```js
init: function () {
    console.log('-----init----')
    //  也就是说在这里$el获取不到，数据观测也没有建立，事件绑定等等都没有完成  实例初始化时同步调用
    console.log(this.$el);
    console.log(this.$data.msg);
    console.log(document.getElementById("title"))
},
created: function () {
    // 这里$el依然获取不到，但是数据观测已经建立，也就是说可以获得数据了，但是依然没有开始编译，也就是说Vue
    // 的一些指令是不起作用的
    // 实例创建后同步调用
    console.log('----created---')
    console.log(this.$el);
    console.log(this.$data.msg);
    console.log(document.getElementById("title"))
},
beforeCompile: function () {
    console.log('---beforeCompile---')
    console.log(this.$el);
    console.log(this.$data.msg);
    console.log(document.getElementById("title"))
},
compiled: function () {
    console.log('---compiled---')
    console.log(this.$el);
    console.log(this.$data.msg);
    // 此时指令已经生效，但是不确保插入到文档中
    console.log(document.getElementById("title"))
},
ready: function () {
    console.log('---ready---')
    console.log(this.$el);
    console.log(this.$data.msg);
    // 在编译结束后和$el第一次插入到文档中的时候
    console.log(document.getElementById("title"))  //这时可以从整个文档中获取到模板中的元素
    console.log('--在ready钩子函数中销毁实例--')
    this.$destroy(true);
},
beforeDestroy: function () {
    // 在销毁实例时调用，所有功能都还保持
    console.log('---beforeDestroy---')
    console.log(this.$el);
    console.log(this.$data.msg);
    console.log(document.getElementById("title"))
},
destroyed: function () {
    //在实例被销毁之后调用。此时所有的绑定和实例的指令已经解绑，所有的子实例也已经被销毁。
    // 与挂载元素解除绑定
    console.log('---destroyed---')
    console.log(this.$el);
    console.log(this.$data.msg);
    console.log(document.getElementById("title"))
}
```

![结果](http://images.djl.pub/17-1-21/56048235-file_1484985260986_9fc2.jpg)

生命周期在组件开发总非常重要，尤其是能在适当的时机取得数据。

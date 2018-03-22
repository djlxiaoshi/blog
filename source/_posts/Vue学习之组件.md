---
title: Vue学习之组件
date: 2017-01-21 22:28:37
tags: 
    - Vue
    - Vue组件
categories: Vue
---

组件在Vue中占有举足轻重的地位，在开发*高仿饿了么外卖APP*中也见识到了组件化开发的便捷和灵活。下面来总结一下。
<!--more-->

## 参考资料

[官方Vue组件文档](http://v1-cn.vuejs.org/guide/components.html)
[Vue.js——60分钟组件快速入门（上篇）](http://www.cnblogs.com/keepfool/p/5625583.html)
[Vue.js——60分钟组件快速入门（下篇）](http://www.cnblogs.com/keepfool/p/5637834.html)

## 注册

### 全局注册
```html
<div id="example">
  <my-component></my-component>
</div>
```
```js
// 定义
var MyComponent = Vue.extend({
  template: '<div>A custom component!</div>'
})
// 注册
Vue.component('my-component', MyComponent)
// 创建根实例
new Vue({
  el: '#example'
})
```
这个组件是全局注册。

### 局部注册

有时我们需要在一个父组件下面注册另一个子组件，而这个子组件只能被这个父组件使用，那么我们就需要使用局部注册。
```js
var Child = Vue.extend({ /* ... */ })
var Parent = Vue.extend({
  template: '...',
  components: {
    // <my-component> 只能用在父组件模板内
    'my-component': Child
  }
})
```
可以看到就是在一个父组件中的**components**属性中添加子组件。

## 注意事项

 1. 在子组件中data属性值是一个匿名函数的返回值

```js
 var MyComponent = Vue.extend({
  data: function () {
    return { a: 1 }
  }
})
```

如果不这样，那么每个实例都将共享这一个data，就会造成混乱。

 2. 我们在子组件模板时，既可以以下面这种方式指定
 
```js
var MyComponent = Vue.extend({
  template: '<div>A custom component!</div>'
})
```
但是我们更推荐使用H5的template标签来做，尤其是在子组件模板比较大的时候
```html
<template id="tmpl">
    <h1 id="title">这是一个组件</h1>
</template>
```
```js
var component = Vue.extend({
    template: "#tmpl"
})
```
当然对子组件模板也有一些要求，不然会解析出错

>   
 - a 不能包含其它的交互元素（如按钮，链接）
 - ul 和 ol 只能直接包含 li
 - select 只能包含 option 和 optgroup
 - table 只能直接包含 thead, tbody, tfoot, tr, caption, col,   colgroup
 - tr 只能直接包含 th 和 td 

 [更多模板注意事项](http://v1-cn.vuejs.org/guide/components.html#模板解析)

## 组件通信

组件使用中最难的就是解决通信问题。一般有一下几种方式和情况
### props

第一种就是在子组件中利用props属性，来实现父子组件中的通信。
```html
<div id="wrap">
    <tmlp-a :props-num="num"></tmlp-a>
    <h1>{{num}}</h1>
</div>
<template id="tmlp-a">
    <h1>{{propsNum}}</h1>
    <button @click="add">add</button>
</template>
```

```js
Vue.component('tmlp-a',{
    template:"#tmlp-a",
    props:['propsNum'],
    methods:{
        add:function () {
            this.propsNum++;
        }
    }
})
new Vue({
    el:"#wrap",
    data:{
        num:1
    }
})
```
通过props这个属性我们就可以拿到父组件的data数据，然后我们就可以操作这个数据，但是这里是一个单项问题，即我们在子组件中操作这个数据，并不会影响父组件中对应的数据，当然有一个办法就是通过props传递过来的是一个引用类型数据而不是一个基本变量，但是这无疑很麻烦。那么Vue提供了另外一种方法，那就是通过**.sync**修饰符来做。把代码做一点修改即可：
```html
<tmlp-a :props-num.sync="num"></tmlp-a>
```
### Vue.$dispatch

第二种方式就是使用Vue.$dispatch（事件配发）来做，这种用法主要用在两个平级的子组件之间，它们有共同的父组件。也就是来解决兄弟组件间的通信问题，因为在一般情况下是不直接通信的（为了组件的独立性），那么有时候我们又有这样的需求。首先来看看通信的流程图：

![dispatch原理图](http://ok3x4ia9b.bkt.clouddn.com/17-1-21/79403842-file_1485003651843_8299.jpg)

上面的通信方式，就如下面的对话。
> 
大儿子小明：老爸，让弟弟帮我拿一下快递，快递号码为（3737） //派发事件并传递参数3737
老爸听到了大儿子的请求，并开始行动 //父组件事件响应函数做出响应
老爸跑去找小儿子小明 //v-ref指令找到另一个子组件
爸爸：小华，快去拿一个快递，快递号码为（3737）
小儿子小明腾腾腾就去拿快递，告诉别人快递号码。

代码示例：
```html
<div id="container">
    <child-a ></child-a>
    <child-b  v-ref:other-child></child-b>

    <template id="child-a">
        <button @click="sendMsg">发送信息</button>
    </template>
    <template id="child-b">
        <h1>{{receiveMsg}}</h1>
    </template>
</div>
```
```js
new Vue({
    el: '#container',
    components: {
        'childA': {
            template: '#child-a',
            data: function () {
                return {
                    msg:'DJL箫氏'
                }
            },
            methods: {
                sendMsg: function () {
                    this.$dispatch('fatherTodo',this.msg);
                }
            }
        },
        'childB': {
            template: '#child-b',
            data:function () {
              return {
                  receiveMsg:''
              }
            },
            methods: {
                receiveMsgFromA: function (msg) {
                    this.receiveMsg = msg;
                    console.log('我接受到来自A的数据：'+msg);
                }
            }
        }
    },
    events: {
        fatherTodo: function (msg) {
            this.$refs.childB.receiveMsgFromA(msg);
        }
    }
});
```

这样就可以在兄弟组件实现通信

### Vuex

最后一种方式就是利用[Vuex](https://github.com/vuejs/vuex/tree/1.0/docs/zh-cn),关于这种方法，我会专门花一篇来写，敬请期待。

 
 

 
 

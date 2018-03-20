---
title: Vue2变化
date: 2017-05-01 21:09:03
tags:
---

之前学习的是Vue1.0，这次项目开发使用的Vue2.0，其中有一些不同和新功能在这里记录一下。[迁移文档](http://cn.vuejs.org/v2/guide/migration.html)
<!--more-->

## [过滤器](https://cn.vuejs.org/v2/guide/migration.html#过滤器)

1 Vue2 filter只能使用在双花括号中,在v-for等等指令中已经不能使用过滤器，转而使用计算属性和第三方库方法（也可以原声js写）例如：filterBy

2 现在Vue2已经移除了所有内置的过滤器，替代方案是使用第三方库（当然简单的也可以自己写一个）

3 参数形式改变，不再是使用空格来分隔参数而是
`<p>\{\{ date \| formatDate('YY-MM-DD', timeZone) \}\}</p>`

当然如果你还是习惯Vue的那种内置过滤器，这里有一个比较好的filters库[vue2-filters](https://github.com/freearhey/vue2-filters)。

## $emit

组件通信，在Vue1.0中采用的是`$dispatch`和`$boradcast `,Vue2.0弃用了，同一使用了$emit，但是并没有使得兄弟组件之间的通信变得简单，对于简单的兄弟组件间的通信，我们可以通过创建一个空的Vue实例作为中央事件总线。

```js
var bus = new Vue()
// 触发组件 A 中的事件
bus.$emit('id-selected', 1)
// 在组件 B 创建的钩子中监听事件
bus.$on('id-selected', function (id) {
  // ...
})
```

对于夸层级的组件（不是父子组件而是爷孙组件）通信，$emit并不能直接像事件冒泡一样，被爷爷组件监听到，这里我们也可以建立一个空的Vue实例作为中央事件总线，来进行通信。例如:

```js
// 将在各处使用该事件中心
// 组件通过它来通信
var eventHub = new Vue()
```

```js
// Son 孙子组件
// ...
methods: {
  addTodo: function () {
    eventHub.$emit('add-todo', { text: this.newTodoText })
    this.newTodoText = ''
  }
}
```

```js
// 爷爷组件
created: function () {
  eventHub.$on('add-todo', this.addTodo)
  eventHub.$on('delete-todo', this.deleteTodo)
},
```

以上主要针对于简单的组件通信，当然如果情况变得更加复杂，还是推荐使用Vuex。

## v-for
1 遍历对象时的参数顺序：原来是(key, value)，现在是 (value, key)
2 v-for="number in 10" 的 number 从 0 开始到 9 结束，现在从 1 开始，到 10 结束。
3 track-by 已经替换为 key

## props
1 现在只允许单项传递（父传子），如果子组件要改变props传递过来的值，并且使得父组件相应的得到改变，就必须利用$emit触发事件。当然还有一种情况就是子组件要求更改props传递过来的值，不需要父组件随之改变，这样貌似不会有什么影响，实际开发中也是一不小心就这样做了，但是这样是反模式的，更好的方式就是在data属性中建立一个字段dataA，然后props值来初始化这个dataA

2v-bind 的 .once和.sync 修饰符 

## v-on 
只能监听自定义事件，如果要监听根元素原生事件，添加.native修饰符
`<my-component v-on:click.native="doSomething"></my-component>`


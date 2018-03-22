---
title: Vuex初探
date: 2017-01-29 23:38:40
tags:
    - vuex
    - vue
---

前篇我们谈到在[Vue组件中进行通信的几种方式](http://djl.pub/2017/01/21/Vue%E5%AD%A6%E4%B9%A0%E4%B9%8B%E7%BB%84%E4%BB%B6/#组件通信)，父子组件中可以通过props属性，兄弟组件可以通过事件分发。当然很明显组件的独立性会有所降低，那么这里有另外一种方式就是通过Vuex。核心思想就是把子组件中需要共享的数据通过Vuex来进行管理。这里我使用的是Vuex1.0版本。
<!--more-->

[官方文档1.0](https://github.com/vuejs/vuex/tree/1.0/docs/zh-cn)

Vuex 1.0 和 2.0都完全支持 Vue 1.0 和 2.0。上面是1.0版本的文档，千万不要安装的2.0版本，查看的却是1.0的文档资料

## 安装

```
npm install vuex@1.0.0
```

这里我们以一个例子来讲解Vuex的基本使用。我们实现一个计数器，实现加一和减一功能。这里我们创建3个组件`Display.vue`、`Increment.vue`和`Decrement.vue`。很简单我们可以分析出这三个组件中会共享一个数据那就是`count`，都需要知道当前计数器的数值是多少。所以这个数据要放在Vuex中进行管理，以后`Display.vue`、`Increment.vue`和`Decrement.vue`这三个组件就是通过访问Vuex来操作`count`这个数据。


## Vuex基本流程

在这里首先借用官方文档中的一幅图片

![Vuex流程图](http://ok3x4ia9b.bkt.clouddn.com/17-1-29/63997298-file_1485700766566_d0e.png)

从这里我们会知道一下几个核心概念

## 核心概念
在使用Vuex时，我们一般会有如下文件结构：

![Vuex基本文件结构](http://ok3x4ia9b.bkt.clouddn.com/17-1-29/92859228-file_1485701346600_9217.jpg)

### state
state对象保存的就是共享数据的当前状态

### mutations
提供操作共享数据的具体方法，能够直接操作state对象。

### getters
在vue组件中，我们可以直接通过this.$store来访问共享数据，但是我们一般不会这么做，我们还是通过Vuex提供的getter属性来取得。

### actions
> action 是一种专门用来被 component 调用的函数。action 函数能够通过分发相应的 mutation 函数，来触发对 store 的更新。action 也可以先从 HTTP 后端或 store 中读取其他数据之后再分发更新事件。

actions不能直接操作state数据的，而是通过dispatch相应的mutations来实现。一般组件直接调用的就是actions中提供的函数接口，而不会去直接调用mutations。

![](http://ok3x4ia9b.bkt.clouddn.com/17-1-29/32989864-file_1485700769110_11d56.jpg)

上图是我自己画的一个调用关系图，从上面我们可以看到，调用过程还是比较复杂的，所以如果是比较小的项目，就不要使用Vuex了。

## 开始动手吧

首先我们通过vue-cli脚手架搭建一个vue工程，如果不是很清楚可以看看我的另一篇博客。然后我们构建`Display.vue`、`Increment.vue`和`Decrement.vue`三个组件。然后新建一个vuex文件夹下面有`store.js`、`actions.js`和`getters.js`。工程目录结构如下图所示。

![计数器工程目录结构](http://ok3x4ia9b.bkt.clouddn.com/17-1-29/90277051-file_1485702257444_17c2d.jpg)

### store.js
```js
import Vue from 'vue'
import Vuex from 'vuex'

//告诉Vue使用vuex
Vue.use(Vuex);

//创建一个对象来保存应用启动时的初始状态

const state = {
// 应用启动时，count为0
    count: 0
}

//创建一个对象存储一系列我们接下来要写的mutation函数
const mutations = {
//    放置我们状态变更函数
    INCREMENT (state,amount) {
         state.count = state.count + amount;
    },
    DECREMENT (state,amount) {
        state.count = state.count - amount;
    }
}

//整合初始状态和变更函数，我们就得到了我们所需的store
// 至此，这个store就可以连接到我们的应用中

export default new Vuex.Store({
    state,
    mutations
})
```
简单来说这个文件就做了几件事
1 告诉vue我们这个项目使用了vuex
2 定义添加共享数据count
3 提供操作共享数据的相关函数接口`INCREMENT`、`DECREMENT`
4 导出这个store供外部使用

### actions.js
action 是一种专门用来被 component 调用的函数。也就是说`Display.vue`、`Increment.vue`和`Decrement.vue`三个组件可能要使用actions.js。
```js
export const incrementCounter = function ({dispatch, state}) {
     dispatch('INCREMENT',1)
}

export const decrementCounter = function ({dispatch, state}) {
    dispatch('DECREMENT',1)
}
```
**dispatch中的字符串参数一定要和store.js中mutations对象中的名字对应**
这里我们可以通过dispatch给对应mutations中的函数提供参数，这里就是`1`。

### Increment.vue

```html
<template>
    <div>
        <div>
            <button @click='increment'>Increment +1</button>
        </div>
    </div>
</template>
<style>
</style>
<script>
    import { incrementCounter } from '../vuex/action'
    export default {
        vuex:{
            actions:{
                increment:incrementCounter
            }
        }
    }
</script>
```
当我们点击按钮时，会触发click事件调用`increment`方法，从而调用`incrementCounter`这个action，通过这个action调用直接操作数据的mutation。于是就这样实现了加一操作。

### Display.vue

接下来就是通过Display.vue这个组件来显示

```html
<template>
    <div>
        <h1>{{counterValue}}</h1>
    </div>
</template>
<style>

</style>
<script>
    import { getCount } from '../vuex/getters'
    export default{
        vuex:{
          getters:{
            counterValue:getCount
          }
        }
    }
</script>
```

我们可以看到并没有直接这样
```js
vuex:{
  getters:{
    counterValue (state) {
     return state.count
    }
  }
}
```
> 1 我们可能需要使用 getter 函数返回需经过计算的值（比如总数，平均值等）。
2 在大型应用里，很多组件之间可以复用同一个 getter 函数。
3 如果这个值的位置改变了（比如从 store.count 变成了 store.counter.value），你只需要改一个 getter 方法，而不是一堆组件。

### App.vue
```html
<template>
    <div>
        <Increment></Increment>
        <Decrement></Decrement>
        <Display></Display>
    </div>
</template>

<script>
import Increment from './components/Increment'
import Decrement from './components/Decrement'
import Display from './components/Display'
import store from './vuex/store'
export default {
  components: {
    Increment,
    Decrement,
    Display
  },
  store:store
}

</script>
```

这里没有什么好特别说明，就是一般我们在根组件中注入store，这样我们在每个组件中都可以访问。

## 最后

以上只实现了加一的功能，减一的功能类似，大家可以自己下来试试，如果觉得这个太容易，可以试一试这个[应用笔记项目](https://segmentfault.com/a/1190000005015164#articleHeader5)。

大年初二，23:30分。。。。。。








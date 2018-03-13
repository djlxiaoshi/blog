---
title: better-scroll初探
date: 2017-02-13 11:34:09
tags:
---

在做外卖APP项目时，用到了better-scroll这个插件，接口基本和iscroll一致，功能非常强大。
<!--more-->

# 文档资料
[官方文档](https://github.com/ustbhuangyi/better-scroll)
主要资料还是参考[iscroll](https://iiunknown.gitbooks.io/iscroll-5-api-cn/content/index.html)这个插件，API接口都是一致的

# 基本使用
一定要有两层
```html
<div class="outer">
    <div class="inner"></div>
</div>
```
```js
<script type="text/javascript">
    var myScroll = new IScroll('#wrapper');
</script>
```
或者
```js
var wrapper = document.getElementById('wrapper');
var myScroll = new IScroll(wrapper);
```

>注意：使用的是querySelector 而不是 querySelectorAll，所以iScroll只会作用到选择器选中元素的第一个。如果你需要对多个对象使用iScroll，你需要构建自己的循环机制。

outer层设置固定大小且为挂载点，overflow:hidden,inner层就是随内容增加。


# 相关设置参数：
**bounce**:当滚动器到达容器边界时他将执行一个小反弹动画，默认值为TRUE
**click:true**    允许点击事件，在这里如果这个设置为TRUE的话，在触发点击事件时，有可能会两次触发（在pc端），这里我们就要通过如下判断
```js
if(event._constructor){
    return ;
}
```
通过better-scroll包装后的event事件对象会都一个`_constructor`属性，如果不进行判断，可能会触发两次点击事件。
**scrollX:true**   横向滚动
**eventPassThrough:vertical** 这个参数是在元素在横向滚动的同时，也是允许外层容器纵向滚动的
**mouseWheel**:侦听鼠标滚轮事件
**scrollbars**:是否显示为默认的滚动条
监听滚动事件，用于获取滚动高度
```js
let scroll = new BScroll(document.getElementById('wrapper'),{
   probeType: 3  //调节在scroll事件触发中探针的活跃度
})

scroll.on('scroll', (pos) => {
  console.log(pos.x + '~' + posx.y)
  ...
});
```
[更多配置](https://iiunknown.gitbooks.io/iscroll-5-api-cn/content/basicfeatures.html)

# 滚动
**滚动到某个元素**
```js
let scroll = new BScroll(document.getElementById('wrapper'))
scroll.scrollToElement(el, 500)
// 第一个参数为X轴，第二个参数为Y轴
```
**滚动到某个坐标**
```js
let scroll = new BScroll(document.getElementById('wrapper'))
scroll.scrollTo(0, 500)
// 第一个参数为X轴，第二个参数为Y轴
```

# 坑  
1 在使用b-scroll的时候，能够准确把握b-scroll初始化的时机是非常重要的。有些时候滚动是由于里面的内容撑开所导致，所以一定要保证里面的内容中的异步数据充分拿到之后进行b-srcoll初始化，所以我们必须要通过Vue的watch监听机制，在监听到数据变化的时候也进行一次b-scroll初始化，当然在ready时候也要进行初始化，因为当我们通过Vue-router进行切换时，整个DOM是重新加载的，但是watch监听数据变化是不会触发的，所以我们就必要在ready的时候进行一次初始化（当然这里也可以使用keep-alive属性阻止DOM重绘）。
```js
    ready () {
        // b-scroll 初始化
    },
    watch: {
        'seller'() {
          // 检测异步数据seller是否获取到，获取到则scroll 初始化
        }
    }
```

2 我们在获取滚动条位置时，一定要保证DOM已经重新绘制。在js中我们一般要这样
```js
ajax('page.php', onCompletion);

function onCompletion () {
    // Update here your DOM

    setTimeout(function () {
        myScroll.refresh();
    }, 0);
};
```
当然在vue中直接使用`this.$nextTick()`。当然我们不需要每次都来初始化b-scroll，我们只需要初始化一次，后面只需要刷新即可。
```js
this.$nextTick(() => {
    if (!this.hScroll) {
        this.hScroll = new BScroll(this.$els.hScrollHook, {
            scrollX: true,
            eventPassthrough: 'vertical'
        });
    } else {
        this.hScroll.refresh();
    }
});
```


# 其他的一些scroll插件
[scroll.js](https://github.com/mkay581/scroll-js)  主要接口和better-scroll差不多，但是功能没有那么强大，不过容易使用

[smoothScroll](https://github.com/alicelieutier/smoothScroll)
```js
var smoothScroll = require('smoothscroll');

var exampleBtn = document.querySelector('.example-button');
var exampleDestination = document.querySelector('.example-destination');

// This function can easily be an onClick handler in React components
var handleClick = function(event) {
  event.preventDefault();

  smoothScroll(exampleDestination);
};

exampleBtn.addEventListener('click', handleClick);
```
使用很简单就像锚点一样，直接跳到某个元素那里。
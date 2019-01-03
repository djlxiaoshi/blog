---
title: vue-router初探
date: 2017-02-08 21:34:12
tags: 
    - vue
    - vue-router
---

最近在跟着视频学习开发一个外卖APP单页运用，开发中用到了vue-router来做路由管理，现在来进行简单的总结。
<!--more-->

# 基本用法

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="js/vue.js"></script>
    <script src="js/vue-router.js"></script>
</head>
<body>
    <div id="container">
        <a v-link="{ path: '/home' }">主页</a>

        <a v-link="{ path: '/about' }">关于我们</a>

        <router-view></router-view>
        <template id="home">
            <h1>home</h1>
        </template>

        <template id="about">
            <h1>about</h1>
        </template>
    </div>
    <script>
        var App=Vue.extend({});

        var home=Vue.extend({
            template:'#home'
        });

        var about=Vue.extend({
            template:'#about'
        })

        var vueRouter=new VueRouter();
        vueRouter.map({
            '/home':{
                component:home
            },
            '/about':{
                component:about
            }
        })

        vueRouter.start(App,'#container');

    </script>
</body>
</html>
```
**注意：我们在使用Vue-router时不需要显示的创建Vue实例，我们只需要通过`vueRouter.start(App,'#container');`将根组件
挂载到某个元素下就可以**

当然这里我们在打开*http://localhost:63342/VueTest/vue-router-study.html#!/*这个路径时router-view中的内容并不会显示
这里我们要定义一个主页,我们可以使用redirect进行重定向
```js
vueRouter.redirect({
    '/':'/home'
});
```
>官方解释:为路由器定义全局的重定向规则。全局的重定向会在匹配当前路径之前执行。
如果发现需要进行重定向，原本访问的路径会被直接忽略而且不会在浏览器历史中留下记录。

当然我们也可以使用go()方法直接导航到一个新的路由
```js
vueRouter.go('/home');
```

# 嵌套路由
在一个路由中嵌套路由是一种比较常见的情况。直接看实例代码
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="js/vue.js"></script>
    <script src="js/vue-router.js"></script>
</head>
<body>
    <div id="container">
        <a v-link="{ path: '/home' }">主页</a>
        <a v-link="{ path: '/about' }">关于我们</a>
        <router-view></router-view>
        <template id="home">
            <h1>home</h1>
            <a v-link="{path:'/home/article'}">我的文章</a>
            <router-view></router-view>
        </template>
        <template id="article">
            <p>这是一篇文章，DJL箫氏</p>
        </template>
        <template id="about">
            <h1>about</h1>
        </template>
    </div>
    <script>
        var App=Vue.extend({});

        var article=Vue.extend({
            template:'#article',
        });

        var home=Vue.extend({
            template:'#home',
        });

        var about=Vue.extend({
            template:'#about'
        })

        var vueRouter=new VueRouter();
        vueRouter.map({
            '/home':{
                component:home ,
                subRoutes:{
                    '/article':{
                        component:article
                    }
                }
            },
            '/about':{
                component:about
            }
        });

        vueRouter.go('/home');
        vueRouter.start(App,'#container');
    </script>
</body>
</html>
```
由示例可知，我们需要在子组件内部去使用v-link和router-view。

# v-link
v-link该指令接受一个 JavaScript 表达式，并会在用户点击元素时用该表达式的值去调用 router.go
```html
<!-- 字面量路径 -->
<a v-link="'home'">Home</a>

<!-- 效果同上 -->
<a v-link="{ path: 'home' }">Home</a>

<!-- 具名路径 -->
<a v-link="{ name: 'user', params: { userId: 123 }}">User</a>
```
当点击v-link所在的元素时，该元素会被添加一个`.v-link-active`的class，通过这个class我们可以用来表示当前选中激活状态。
当然如果我们不想用这个类名，我们可以通过linkActiveClass 来设置，或者在内联元素上单独设置`<a v-link="{ path: '/a', activeClass: 'custom-active-class' }"></a>`
```js
var router = new VueRouter({
    linkActiveClass: 'active'
});

```

# 具名路径
当我们一个路径过长时，我们就可以找一个别名来代替这个路径，然后还可以设置参数。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="js/vue.js"></script>
    <script src="js/vue-router.js"></script>
</head>
<body>
<div id="container">
    <a v-link="{name:'articleName',params:{id:1}}">我的文章</a>
    <a v-link="{name:'articleName',params:{id:2}}">我的文章</a>
    <a v-link="{name:'articleName',params:{id:3}}">我的文章</a>
    <router-view></router-view>
    <template id="article">
        <p>这是第{{$route.params.id}}篇文章，DJL箫氏</p>
    </template>
</div>
<script>
    var App=Vue.extend({});

    var article=Vue.extend({
        template:'#article',
    });

    var vueRouter=new VueRouter();
    vueRouter.map({
        '/article/:id':{
            name:'articleName',
            component:article
        }
    });

    vueRouter.go('/article');
    vueRouter.start(App,'#container');
</script>
</body>
</html>
```
我们可以通过vue-router提供的相关接口取到对应的值 [路由规则和路由匹配](https://github.com/vuejs/vue-router/blob/1.0/docs/zh-cn/route.md)

# keep-alive
```html
<router-view :seller='seller' keep-alive></router-view>
```
我们加上了`keep-alive`这个属性，当我们不加的时候，每次切换路由都会重新加载DOM，这样就会出现一些问题，例如下图

![keep-alive示例图](http://images.djl.pub/17-2-8/4536530-file_1486560148439_faff.jpg)

我们此时在*商品*路由下面有两个实物加入了购物车，这是此时的状态，加入我们不加`keep-alive`我们切换到*评论*路由，然后再切换到*商品*路由，由于DOM会刷新，那么我们此时的已经加入购物车的状态就会清空，这显示不是我们想要的。如下图：

![](http://images.djl.pub/17-2-8/65884667-file_1486560396173_c4d9.jpg)


# 坑
我使用Vue-router（0.7.3）来作路由管理，由于我们的Vue是1.0版本所以Vue-router要选用0.7.3而不是2.0的版本,对应关系如下：

>Vue 2.0  ---  Vue-router 2.0   [Vue-router 2.0 文档](https://router.vuejs.org/zh-cn/)
    
>Vue 1.0  ---  Vue-router 0.7.3 [Vue-router 0.7.3 文档](https://github.com/vuejs/vue-router/tree/1.0/docs/zh-cn)

# 最后

上面写的是我最基本的用法，也是我目前用到的，当然他还有很多用途，现在还没用到，等用到的时候再加上，最后附上另一篇参考文章。[Vue.js——vue-router 60分钟快速入门](http://www.cnblogs.com/keepfool/p/5690366.html)

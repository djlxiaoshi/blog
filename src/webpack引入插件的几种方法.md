---
title: webpack引入插件的几种方法
date: 2017-05-01 20:23:36
tags: 
    - webpack
---

通常webpack引入插件有4中方式：
1 安装后直接在某个模块中运用
2 使用webpack plugins功能加载ProvidePlugin插件
3 expose-loader加载器加载
4 包装插件
<!--more-->

## 直接使用

在需要使用的js文件中引入：
`var  $  = require('jquery')`或者`import $ form 'jquery'`

## 使用plugins
```js
plugins: [
    new webpack.ProvidePlugin({
      $: "jquery",
      jQuery: "jquery"
    })
]
```
使用这种方式，就可以全局使用jQuery，但是对于使用了ESlint就会报错：`'$' is not defined`

> ProvidePlugin: Automatically loaded modules. Module (value) is loaded when the identifier (key) is used as free variable in a module. The identifier is filled with the exports of the loaded module.

也就是说jQuery模块只会在使用`$`的模块中加载，并不是真正的全局加载

## [expose-loader](https://github.com/webpack-contrib/expose-loader)

```js
{
    test: require.resolve('jquery'),
    use: [{
        loader: 'expose-loader',
        options: '$'
    }]
}
```

在任意模块中引入`require("expose-loader?$!jquery");`

## 包装jQuery

vendor.js
```js
import $ from 'jquery'
window.$ = $
window.jQuery = $
export default $
```
模块引入：`import $ from '../assets/vendor.js'`
webpakck中设置别名：
```js
alias: {
    //将其指向vendor.js所在位置
    jquery : 'src/assets/jquery-vendor.js' 
} 
```

[参考文章](http://blog.csdn.net/yiifaa/article/details/51916560)







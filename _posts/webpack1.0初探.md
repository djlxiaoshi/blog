---
title: webpack1.0初探
date: 2017-02-07 08:49:08
tags: webpack
---

webpack是目前最热门的前端资源模块化管理和打包工具,它能把各种资源，包括 jxs、coffeeJS、less／sass，甚至图片，当作模块来加载和使用。当我们需要使用这些资源时只需要require进来，方便模块化开发。
<!--more-->

# [webpack 使用](https://webpack.github.io/docs/usage.html)

## 安装

首先全局安装`webpack $npm install webpack -g`

再安装`webpack-dev-server`这是一个webpack提供的静态开发环境服务器，用来开发调试

>注：如果是在开发环境中用到的包我们使用`--save-dev`的参数，如果生产环境用到的如jQuery，我们直接只用`save`参数

## 配置文件
使用webpack最重要的就是配置文件`webpack.config.js`，和gulp等等工具一样都有一个配置文件

## 常用配置选项
### entry

```js
entry : {
    'admin': './admin/admin.js',
    'custom':'./custom/custom.js'
}
```
entry表示webpack的入口文件，可以是多个，那么webpack会一次执行。

### output

```js
output:{
    filename:"[name].bundle.js",
    path:path.join(__dirname,'dist'),
    publicPath:'/dist/'
}
```
`filename`编译后输出文件的名字
`path`:编译后文件的存储位置

### plugins: []
#### UglifyJsPlugin
代码压缩 UglifyJsPlugin。这是一个webpack内置的插件，我们在使用时，只需要在webpack.config.js 引入，然后在plugins选项中注册

```js
var webpack = require('webpack');

{
    plugins:[new webpack.optimize.UglifyJsPlugin({
             compress: {
                 warnings: false
             }
         })]
}
```

他会将所有的js文件进行压缩，但是我们只希望生成环境的代码压缩，开发环境的文件并不希望压缩

如果我们只想在build时候uglify,那么我们可以设置一个参数，在build的启动项中，然后利用第三方包`node-argv`去读这个参数，然后就可以通过if来进行判断。

```js
var args = require('node-argv');
if(args.minify){// 这个minify就是自己设置的参数
    // toDo ...
}
```
还有一种是读取环境变量的方式。
```js
//webpack.config.js
var env = process.env.NODE_ENV;
if(env === 'production'){
    // toDo
}
```
```json
{
  "scripts": {
    "start": "NODE_ENV=dev webpack-dev-server --progress --colors --hot --inline --d",
    "bulid": "NODE_ENV=producrion webpack-dev-server --progress --colors --hot --inline --d"
  }
}
```
```js
// webpack.config.js

// definePlugin 会把定义的string 变量插入到Js代码中。
var definePlugin = new webpack.DefinePlugin({
  __DEV__: JSON.stringify(JSON.parse(process.env.BUILD_DEV || 'true')),
  __PRERELEASE__: JSON.stringify(JSON.parse(process.env.BUILD_PRERELEASE || 'false'))
});

module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  },
  plugins: [definePlugin]
};
```

配置完成后，就可以使用 `BUILD_DEV=1 BUILD_PRERELEASE=1` webpack来打包代码了。 值得注意的是，`webpack -p`会删除所有无作用代码，也就是说那些包裹在这些全局变量下的代码块都会被删除，这样就能保证这些代码不会因发布上线而泄露。

#### [Open Browser Webpack Plugin](https://github.com/baldore/open-browser-webpack-plugin)

使用
`npm i --save-dev open-browser-webpack-plugin`
```js
var OpenBrowserPlugin = require('open-browser-webpack-plugin');

module.exports = {
  entry: path.resolve(__dirname, 'lib/entry.js'),
  output: {
    path: __dirname + "/bundle/",
    filename: "bundle.js"
  },
  plugins: [
    new OpenBrowserPlugin({ url: 'http://localhost:3000' })
  ]
};
```
详细配置请见[官网](https://github.com/baldore/open-browser-webpack-plugin)

### loaders

[loader lists](https://webpack.github.io/docs/list-of-loaders.html)

#### css 样式文件
 css-loader的作用是将css文件写入一个js文件里面
 
 style-loader的作用就是将这个转换后的样式文件解析插入到HTML中
 
 loader 有两种配置方法，一种是给每个entry文件单独添加loader例如`require('style!css!./admin.css');`注意在这里每一个loader后面都有一个"!"。这个顺序是*从右往左*的
另一种方式是在webpack.config.js文件中整体配置使用
```js
 module: {
        loaders: [{
            test:/\.css$/,
            loaders:['style','css']
        }]
}
```
统一使用loaders:[]数组的方式

#### 图片
对于引用图片有两种引用模式，一种是直接require，另一种是转换成base64（一般针对小图标）
```js
loaders: [
      {test: /\.(png|jpg)$/, loader: 'file-loader'},
      {test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'} // 内联的base64的图片地址，图片要小于8k，直接的url的地址则不解析
    ]
```

#### sass
node-sass模块和sass-loader。
[参考资料](https://github.com/jtangelder/sass-loader)

#### ES6 加载
这个依赖了3个模块`Babel-loader加载器`、`babel-preset-es2015`、`babel-core`。当然
如果是在react中使用还要依赖`babel-preset-react`。两种方式使用：
```js
module: {
  loaders: [
    {
      test: /\.js?$/,
      exclude: /node_modules/,
      loader: 'babel',
      query: {
        presets: ['es2015']
      }
    }
  ]
}
```
或者
```js
 module: {
    loaders:[
      {
        test: /\.js[x]?$/,
        exclude: /node_modules/,
        loader: 'babel-loader?presets[]=es2015'
      },
    ]
  }
```
*NOTE:The query string is appended to the loader with ?. i.e. url-loader?mimetype=image/png.*

当然也可以配置一个.babelrc的配置文件
```json
 { "presets": [ "es2015" ] }
```


### resolve 配置选项
如果你希望在require文件时省略文件的扩展名，只需要在webpack.config.js中添加 resolve.extensions 来配置。

```js
  resolve: {
    // 现在你require文件的时候可以直接使用require('file')，不用使用require('file.coffee')
    extensions: ['', '.js', '.json', '.coffee']  // 但是必须要在前面加一个空的字符串，否则会导致无法加载的情况
  }
```

### 优化通用代码
Feed和Profile页面存在大量通用代码(比如React、公共的样式和组件等等)。webpack可以抽离页面间公共的代码，生成一个公共的bundle文件，供这两个页面缓存使用:
```js
// webpack.config.js

var webpack = require('webpack');

var commonsPlugin =
  new webpack.optimize.CommonsChunkPlugin('common.js'); // 引入插件

module.exports = {
  entry: {
    Profile: './profile.js',
    Feed: './feed.js'
  },
  output: {
    path: 'build',
    filename: '[name].js' // 为上面entry的key值
  },
  plugins: [commonsPlugin]
};
```

在上一步引入自己的bundle之前引入<script src="build/common.js"></script>


### 生成Source Maps（使调试更容易）

开发总是离不开调试，如果可以更加方便的调试当然就能提高开发效率，
不过打包后的文件有时候你是不容易找到出错了的地方对应的源代码的位置的，Source Maps就是来帮我们解决这个问题的。
通过简单的配置后，Webpack在打包时可以为我们生成的source maps，这为我们提供了一种对应编译文件和源文件的方法，
使得编译后的代码可读性更高，也更容易调试。

在webpack的配置文件中配置source maps，需要配置devtool，
它有以下四种不同的配置选项，各具优缺点，描述如下：

devtool选项   配置结果

`source-map`     在一个单独的文件中产生一个完整且功能完全的文件。这个文件具有最好的source map，但是它会减慢打包文件的构建速度；

`cheap-module-source-map`   在一个单独的文件中生成一个不带列映射的map，不带列映射提高项目构建速度，但是也使得浏览器开发者工具只能对应到具体的行，不能对应到具体的列（符号），会对调试造成不便；

`eval-source-map`   使用eval打包源文件模块，在同一个文件中生成干净的完整的source map。这个选项可以在不影响构建速度的前提下生成完整的sourcemap，但是对打包后输出的JS文件的执行具有性能和安全的隐患。不过在开发阶段这是一个非常好的选项，但是在生产阶段一定不要用这个选项；

`cheap-module-eval-source-map`  这是在打包文件时最快的生成source map的方法，生成的Source Map 会和打包后的JavaScript文件同行显示，没有列映射，和eval-source-map选项具有相似的缺点；

正如上表所述，上述选项由上到下打包速度越来越快，不过同时也具有越来越多的负面作用，较快的构建速度的后果就是对打包后的文件的的执行有一定影响。

在学习阶段以及在小到中性的项目上，eval-source-map是一个很好的选项，不过记得只在开发阶段使用它，继续上面的例子，进行如下配置

```
module.exports = {
  devtool: 'eval-source-map',//配置生成Source Maps，选择合适的选项
  entry:  __dirname + "/app/main.js",
  output: {
    path: __dirname + "/public",
    filename: "bundle.js"
  }
}
```

`cheap-module-eval-source-map`方法构建速度更快，但是不利于调试，推荐在大型项目考虑da时间成本是使用。

## 启动
我们可以直接通过webpack-dev-server（前提是全局安装）命令直接在命令窗口中启动，也可以通过设置package.json
一个启动项，例如：`"start": "webpack-dev-server --progress --colors --hot --inline"`这里就不需要webpack-dev-server
这个包是全局安装。这种启动方式，寻找包会到local环境中查找，找不到再去查找全局。

`--progress`这个参数是显示进度

`--colors` 显示颜色，这样命令窗口看起来比较爽

`--hot` 启动热加载，就是文件保存，浏览器自动刷新，类似于liveonload这个功能

`--inline` 把内容注入到相应文件中

`--d` 开启debug模式，这样在浏览器中我们就可以看到源码

在entry的js文件里面可以require任何一个文件(css,js等等)

# 坑
1  视频老师用的是mac系统，正则匹配文件路径时这样`/\/images\//`，但是我的是Windows的系统所以文件路径匹配应该是这样`/\\img\\/`

2 在webpack.config.js配置文件中，应该是不能使用ES6的语法的。

# 不错的webpack资料
[webpack-howto](https://github.com/petehunt/webpack-howto/blob/master/README-zh.md)

[视频资料](http://www.maiziedu.com/course/570/)

[webpack 入门实战](https://segmentfault.com/a/1190000008032524)

[阮一峰老师webpack](https://github.com/ruanyf/webpack-demos)

# 最后
感觉这篇写的并不是很好，然而webpack2.0出来了，我1.0才开始，嗯对Vue似乎也是这样。







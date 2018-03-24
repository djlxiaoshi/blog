---
title: webpack2实战
date: 2017-07-26 16:59:46
tags: 
    - webpack
---

一直对于webpack有一种莫名奇妙的恐惧感，一直在听说，从未有深入，这篇专门用来记录webpack实战笔记，
最终将写一个自己的脚手架工具。
<!-- more -->
在这里主要记录如何从0利用webpack构建一个项目。

```javascript
mkdir webpack-demo && cd webpack-demo
npm init -y
npm install --save-dev webpack
```
安装好webpack之后，我们要新建一个webpack.config.js，webpack编译项目依赖这个配置文件。我们可以利用如下命令来编译一个js文件`./node_modules/.bin/webpack src/index.js dist/bundle.js`，但是这里要写太长，所以我们可以在`package.json`文件里面配置一个script选项：
```javascript
{
  "scripts": {
    "build": "webpack"
  }
}
```
配置好后我们就可以在终端直接利用`npm run build`命令启动webpack，但是在这里webpack并不知道我们应该从哪个文件开始编译（即入口文件），也不知道编译后的文件应该存放在哪里，所以我们要在`webpack.config.js`中进行配置。
```javascript
var path = require('path');

module.exports = {
  entry: './src/index.js',   // 入口文件地址
  output: {
    filename: 'bundle.js',   // 编译输出后的文件名
    path: path.resolve(__dirname, 'dist')  // 编译后的文件存放路径
  }
};
```
配置完成后就可以`npm run build`了。

## Loader
webpack使用中对重要的就是如何使用各种loader来完成不同的任务，下面记录比较常见的loader使用。

### Loading CSS
webpack会把css、图片等等文件编译成js文件，这样我们就可以在js文件中`import`对应的CSS或者图片文件。为了实现这个功能webpack就必须利用各种loader来将非js文件编译成js文件。
```javascript
var path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
    module: {
         rules: [
           {
             test: /\.css$/,  // 匹配以.css结尾的文件
             use: [
               'style-loader', 
               'css-loader'
             ]
           }
         ]
       }
 };
```
我们必须要在module属性中配置各种加载器规则，`test`属性匹配文件规则，`use`指定当匹配到对应的文件时，应该使用什么加载器来处理，这里用到了`style-loader`和`css-loader`。所以我们必须为项目安装这两个加载器：
`npm install --save-dev style-loader css-loader`
> `--save-dev` 表示会把依赖写入到`package.json`的`devDependencies`字段中，这样的npm 包只会在开发中使用，生产环境中不需要，
> `--save`则会把依赖写入到`package.json`的`dependencies`字段中，这里面的包在生产环境也是需要的例如：jQuery、lodash等等，当然有时候我们直接`npm isntall 包名`，这样依赖就不会写入`package.json`中，所以这样别人安装你的项目可能会报错，原因就是找不到相应的包。

**这里有两个加载器，loader的处理顺序是从最后一个到第一个**

### sass-loader
`npm install sass-loader node-sass webpack --save-dev`

```javascript
rules: [{
    test: /\.scss$/,
    use: [{
        loader: "style-loader" // 将 JS 字符串生成为 style 节点
    }, {
        loader: "css-loader" // 将 CSS 转化成 CommonJS 模块
    }, {
        loader: "sass-loader" // 将 Sass 编译成 CSS
    }]
}]

```

然后在js文件中require这个scss文件（支持ES6的话可以使用import），但是这样有一个不好的地方在于最终编译解析的的CSS文件你是看不到的，它是存在于你引入的js文件之中，一般情况下我们希望CSS文件能够单独在一个文件夹里面。所以这里我们就要使用另外一个插件`extract-text-webpack-plugin`
```javascript
npm install --save-dev extract-text-webpack-plugin
```

```javascript
const ExtractTextPlugin = require("extract-text-webpack-plugin");

const extractSass = new ExtractTextPlugin({
    filename: "[name].[contenthash].css",
    // disable: process.env.NODE_ENV === "development"
});

module.exports = {
    ...
    module: {
        rules: [{
            test: /\.scss$/,
            use: extractSass.extract({
                use: [{
                    loader: "css-loader"
                }, {
                    loader: "sass-loader"
                }],
                // 在开发环境使用 style-loader
                fallback: "style-loader"
            })
        }]
    },
    plugins: [
        extractSass
    ]
};
```

### Babel ES6
ES6虽然在Node服务器端得到了较好的支持，但是在客户端中支持还不够多，为了使用ES6甚至是ES7，我们需要安装babel转码工具：
```javascript
npm install babel-core babel-preset-env babel-loader --save-dev
```
其中`babel-core`是核心的转码包，但是有些ES6的语法也是不能够转换的，所以我们要使用一些其他的扩展包，例如：`babel-preset-env`和`babel-preset-es2015`。`babel-preset-env`可以将ES2015/ES2016/ES2017都转换成ES5，当然如果我们只需要将ES6转换成ES5，那么我们也可以只安装`babel-preset-es2015`。
```javascript
{
   test: /\.js$/,
    // 排除node_modules目录下的文件, npm安装的包不需要编译
    exclude: /node_modules/,
    use: ['babel-loader']
}
```
然后新建一个`.babelrc`写入：
```javascript
{ "presets": [ "es2015" ] }
// 或者 { "presets": [ "env" ] }
```

### Typescript
1 安装typescript编译器和加载器
```javascript
npm install --save-dev typescript ts-loader
```

2 增加tsconfig.json配置文件
```javascript
{
  "compilerOptions": {
    "outDir": "./dist/",
    "sourceMap": true,
    "noImplicitAny": true,
    "module": "commonjs",
    "target": "es5",
    "jsx": "react",
    "allowJs": true
  }
}
```

[更多tsconfig配置 ](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)
[中文配置说明](https://github.com/hstarorg/HstarDoc/blob/master/%E5%89%8D%E7%AB%AF%E7%9B%B8%E5%85%B3/TypeScript%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6tsconfig%E7%AE%80%E6%9E%90.md)

3 配置webpack.config.js
```javascript
resolve: {
    extensions: [".tsx", ".ts", ".js"]
  },
  
  module: {
    loaders: [
      { test: /\.ts$/, loader: 'ts-loader' }
    ]
  }
```


### Loading Images
在module属性中配置：
```javascript
{
   test: /\.(png|svg|jpg|gif)$/,
   use: [
    'file-loader'
   ]
}
```
当然可以将图片编译成base64格式，这里我们要利用`url-loader`
```javascript
use: [
   {
     loader: 'url-loader',
     options: {
       limit: 10000
     }
   }
]
```
使用url-loader, 它接受一个limit参数, 单位为字节(byte)，当文件体积小于limit时, url-loader把文件转为Data URI的格式内联到引用的地方，当文件大于limit时, url-loader会调用file-loader, 把文件储存到输出目录, 并把引用的文件路径改写成输出后的路径。（为了达到这种效果只需要配置url-loader即可）

> 当我们在使用url-loader将html文件中的图片转换成base64的格式，我们一定要先加上[html-loader](https://doc.webpack-china.org/loaders/html-loader/)

```javascript
{
  test: /\.(html)$/,
  use: {
    loader: 'html-loader',
    options: {
      attrs: ['img:src']  // 一般在使用默认值就行attrs: ['img:src']
    }
  }
}
```

## ESLint
[在Vue+Babel+Webpack环境中使用ESLint](http://tutuxxx.github.io/2016/08/14/%E5%9C%A8Vue+Babel+Webpack%E7%8E%AF%E5%A2%83%E4%B8%AD%E4%BD%BF%E7%94%A8ESLint/)
[webpack2教程续之eslint检测](http://www.jianshu.com/p/4be321cbdc1e)
[webpack2集成eslint](https://segmentfault.com/a/1190000008575829)

## webpack-dev-server
我们项目一般需要启动一个静态服务器，并且希望修改文件后，就能够自动完成webpack编译、实时刷新浏览器等等操作，那么这里我们需要引入`webpack-dev-server`这个模块。`npm install webpack-dev-server --save-dev`。然后在`webpack.config.js`中配置`devServer`选项：
```javascript
devServer: {
  // 配置监听端口
  port: 8100,

  /*
   historyApiFallback用来配置页面的重定向
   配置为true, 当访问的文件不存在时, 返回根目录下的index.html文件
   */
  historyApiFallback: true
}
```
然后在`package.json`的`script`中增加一个任务：
```javascript
"dev": "webpack-dev-server -d --hot --colors --inline --env.dev"
```
然后执行`npm run dev`
```
--progress这个参数是显示进度
--colors 显示颜色，这样命令窗口看起来比较爽
--hot  开启热替换
--inline 监听文件变化，浏览器自动刷新，类似于liveonload这个功能
--d 开启debug模式，这样在浏览器中我们就可以看到源码
```


## plugin

### html-webpack-plugin
我们会碰到这样一种情况就是我们打包图片的时候，每一次build之后，图片的名称都会改变，这时在html文件中是无法正确引入图片路径的，为了解决这个问题，我们需要引入`html-webpack-plugin`这个插件。
```javascript
npm install html-webpack-plugin --save-dev
```
然后在`webpack.config.js`中引入`var HtmlWebpackPlugin = require('html-webpack-plugin')`,最后在plugins属性中进行配置：
```javascript
plugins: [
        /*
         html-webpack-plugin用来打包入口html文件
         */
        new HtmlWebpackPlugin({
            /*
             template参数指定入口html文件路径, 插件会把这个文件交给webpack去编译,
             webpack按照正常流程, 找到loaders中test条件匹配的loader来编译, 那么这里html-loader就是匹配的loader
             html-loader编译后产生的字符串, 会由html-webpack-plugin储存为html文件到输出目录, 默认文件名为index.html
             可以通过filename参数指定输出的文件名
             html-webpack-plugin也可以不指定template参数, 它会使用默认的html模板.
             */
           template: './dist/index.html' // 基础html模板
        })
    ],
```
使用这个插件以后，会自动创建一个index.html，并且把编译后的js文件，图片资源链接都插入到对应位置。

### favicon
可以利用html-webpack-plugin的favicon配置属性
```javascript
new HtmlWebpackPlugin({
   title: 'Webpack TypeScript',
   favicon: 'favicon.ico', //icon路径
   template: './dist/index.html' // 基础html模板
})
```

具体可以查看[示例](https://github.com/jantimon/html-webpack-plugin/tree/master/examples/favicon)。当然除了这个方法我们还可以使用`favicons-webpack-plugin`这个插件


### 提取公共代码（CommonsChunkPlugin）

有一些类库如bootstrap或者jQuery等等会被多个页面共享，最好的方式是讲这些公共的类库打包成一个通用的js文件。我们指定好生成文件的名字，以及想抽取哪些入口js文件的公共代码，webpack就会自动帮我们合并好。
```javascript
new webpack.optimize.CommonsChunkPlugin({
    name: "common",
    filename: "js/common.js",
    chunks: ['index', 'detail] //index和details为入口文件名
}),
```

### 代码压缩（UglifyJsPlugin）
webpack2默认开启了代码压缩，但是没有做到压缩到最小，所以我们可以通过手动配置覆盖默认配置
```javascript
new webpack.optimize.UglifyJsPlugin({
   // 最紧凑的输出
   beautify: false,
   // 删除所有的注释
   comments: false,
   compress: {
       // 在UglifyJs删除没有用到的代码时不输出警告  
       warnings: false,
       // 删除所有的 `console` 语句
       // 还可以兼容ie浏览器
       drop_console: true,
       // 内嵌定义了但是只用到一次的变量
       collapse_vars: true,
       // 提取出出现多次但是没有定义成变量去引用的静态值
       reduce_vars: true,
   }
})
```


## resolve
```javascript
resolve: {
    //查找module的话从这里开始查找
    root: 'E:/github/flux-example/src', //绝对路径

    //自动扩展文件后缀名，意味着我们require模块可以省略不写后缀名
    extensions: ['', '.js', '.json', '.scss'],

    //模块别名定义，方便后续直接引用别名，无须多写长长的地址
    alias: {
        AppStore : 'js/stores/AppStores.js',//后续直接 require('AppStore') 即可
        ActionType : 'js/actions/ActionType.js',
        AppAction : 'js/actions/AppAction.js'
    }
}
```

## hash
有时我们为了防止缓存，需要给编译生成的文件名设置一个hash，配置如下：
```javascript
output: {
    filename: '[name].[hash].bundle.js',
    path: path.resolve(__dirname, 'dist')
}
```

## webpack 引入第三方库
[详情见这篇文章](http://djl.pub/2017/05/01/webpack%E5%BC%95%E5%85%A5%E6%8F%92%E4%BB%B6%E7%9A%84%E5%87%A0%E7%A7%8D%E6%96%B9%E6%B3%95/)

即使不符合commonJs规范
```javascript
module.exports = {
    resolve: {
        root: [],
        alias: {
            'jquery': path.resolve(rootDir, './lib/jquery.min.js'); 
        }
    },
    plugins: [
        new webpack.ProvidePlugin({
            $: 'jquery'
        }),
    ]
};
```

## 环境配置
```json
"build": "set NODE_ENV=production&&webpack -p --progress --colors",
"dev": "set NODE_ENV=development&&webpack-dev-server -d --hot --colors --env.dev"
```

然后在`webpack.config.js`中就可以通过`process.env.NODE_ENV`拿到对应的环境变量值


## 参考文章
[webpack 2 打包实战](https://github.com/fenivana/webpack-in-action)

[Webpack 2 快速入门](https://github.com/dwqs/blog/issues/46)

[webpack各种loader](https://doc.webpack-china.org/loaders/)

持续更新中。。。。。。

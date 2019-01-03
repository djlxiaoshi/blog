---
title: Webpack优化
date: 2018-09-02 19:11:08
tags: 
    - Webpack
---

webpack 优化主要分为两部分，一是优化构建速度，二是优化输出质量。所谓优化构建速度，那就是要打包快，优化输出质量就是要打的包尽可能的小。
<!--more-->

### 缩小文件匹配范围(include/exclude)
顾明思议，exclude就是用来告诉loader哪些目录下的资源是不用构建，而include就是告诉loader哪些目录下的文件是需要被构建的。
```
module: {
    rules: [
        {
            test: /\.js$/,
            use: 'babel-loader',
            exclude: /node_modules/, // 排除不处理的目录
            include: path.resolve(__dirname, 'src') // 精确指定要处理的目录
        }
    ]
}
```

### 缓存loader的执行结果(cacheDirectory)
> cacheDirectory：默认值为 false。当有设置时，指定的目录将用来缓存 loader 的执行结果。之后的 webpack 构建，将会尝试读取缓存，来避免在每次执行时，可能产生的、高性能消耗的 Babel 重新编译过程(recompilation process)。如果设置了一个空值 (loader: 'babel-loader?cacheDirectory') 或者 true (loader: babel-loader?cacheDirectory=true)，loader 将使用默认的缓存目录 node_modules/.cache/babel-loader，如果在任何根目录下都没有找到 node_modules 目录，将会降级回退到操作系统默认的临时文件目录。

> 以上来自[这里](https://webpack.docschina.org/loaders/babel-loader/)


```
module: {
    rules: [
        {
            test: /\.js$/,
            use: 'babel-loader?cacheDirectory', // 缓存loader执行结果
            exclude: /node_modules/,
            include: path.resolve(__dirname, 'src')
        }
    ]
}
```

### 优化 resolve modules 配置
resolve modules：它用于配置 Webpack 去哪些目录下寻找第方模块。默认值是['node_modules']，这代表的含义先去当前目录的./node_modules目录下去找我们想找的模块，如果没找到，就去上一级目录的../node_modules中找，如果还没有找到去../../node_modules中去找，以此类推，这和Node.js的模块寻找机制是相似的(向上递归搜索的方式)，在现在的应用中我们的第三方模块都放在项目的根目录的node_modules目录下，所以就没有必要用默认向上的方式一层层去找，在这一层如果没有找到，就直接报错就行。配置如下：

```
const path = require('path');

function resolve(dir) { // 转换为绝对路径
   return path.join(__dirname, dir);
}

resolve: {
    modules: [ // 优化模块查找路径
        path.resolve('src'),
        path.resolve('node_modules') // 指定node_modules所在位置 当你import 第三方模块时 直接从这个路径下搜索寻找
    ]
}
```

### 优化 resolve.alias 配置

resolve.alias 配置项通过别名来将原导入路径映射成一个新的导入路径。
在实 战项目中经常会依赖一些庞大的第三方模块，以 React 库为例，安装到 node_module 目录下的 React 库的目录结构如下：

![](http://images.djl.pub/19-1-3/32679138.jpg)

可以看到在发布出去的 React 库中包含两套代码:
- 一套是采用 CornmonJS 规范的模块化代码，这些文件都放在 ib 录下，package.json 中指定的入口文件 react.js 为模块的入口。
- 一套是将 React 的所有相关代码打包好的完整代码放到 个单独的文件中， 这些代码没有采用模块化，可以直接执行。其中 dist/react.j s用于开发环境，里包含检查和警告的代码。 dist/react.min.j 用于线上环境，被最小化了。

在默认情况下， Webpack 会从入口文件.／ node_modules/react/react.js 开始递
归解析和处理依赖的几十个文件，这会是一个很耗时的操作。通过配置 resolve. alias, 
可以让 Webpack 在处理 React 库时，直接使用单独、完整的 react.min.js 文件 ，从而跳
过耗时的递归解析操作。

```
module.exports = {
	resolve: {
	// 使用 alias 将导入 react的语句换成直接使用单独、 完整的 react.min.js 文件，
	// 减少耗时的递归解析操作
		alias: {
			'react': path.resolve(__dirname, './ node_modules/react/dist/react.min.js')
		}
	}
};
```
对某些库使用本优化方法后，会影响到后面要讲的使用 Tree-Sharking 去除无效
代码的优化，因为打包好的完整文件中有部分代码在我们的项目中可能永远用不上。一般对整体性比较强的库采用本方法优化，因为完整文件中的代码是个整体，每行都是不可或缺的但是对于些工具类的库如 odash ( https:/ github.com/lodash/lodash），我们的项目中可能只用到了其中几个工具函数，就不能使用本方法去优化了，因为这会导致在我们 的输出代码中包含很多永远不会被执行的代码。

### 优化 resolve.extensions 配置

resolve.extensions 用于配置在尝试过程中用到的后缀列表，默认是：`extensions:['.js', '.json']`

也就是说，当遇到 require ( '. /data ’） 这样的导入语句 webpack 会先去寻找`./data .js` 文件，如果该文件不存在，就去寻找 `data.json` 文件，如果还是找不到就报错。如果这个列表越长，或者正确的后缀越往后，就会造成尝试的次数越多，所以
resolve .extensions 的配置也会影响到构建的性能 在配置 resolve.extensions时需要遵守 以下几点，以做到尽可能地优化构建性能。
- 后缀尝试列表要尽可能小，不要将项目中不可能存在的情况写到后缀尝试列表中。
- 频率出现最高的文件后缀要优先放在最前面，以做到尽快退出寻找过程。
- 在源码中写导入语句时，要尽可能带上后缀从而可以避免寻找过程。例如在确定
的情况下将 `require('./data')`写成 `require('. data.json')`。

### 优化 module. noParse 配置

module.noParse 配置项可以让 Webpack 忽略对部分没采用模块化的文件的递归解析处理，这样做的好处是能提高构建性能。原因是一些库如 jQuery、ChartJS 庞大又没有采用模块化标准，让 Webpack 解析这些文件既耗时又没有意义。

```
module.exports = { 
	module: { 
		noParse: [/react\.min\.js$/],  //单独、完整的 react.min.js 文件没有采用模块归解析处理
	}
};
```
> 注意，被忽略掉的文件里不应该包含 import require define 等模块化语句，不
然会导致在构建出的代码中包含无法在浏览器环境下执行的模块化语句。

### 使用 DllPlugin

dll（动态链接库），在一个动态链接库中可以包含为其他模块调用的函数和数据。
为什么需要动态链接库：
在通常情况下我们使用一个第三方包，我们希望最后加载的是min.js，现在基本都会去踢动这样一个打包后的min.js。但是我们同在使用第三方包是通过`import React from 'react'`来进行引用，但是这样引用Webpack就对React做了一次构建，但其实React官方已经提供了构建好的react.min.js，我们其实不必要去再构建一遍，于是我们可能这样去配置Webpack：
```
module.exports = {
    externals: {
        'react': 'window.React'
    }
    //其它配置忽略...... 
};
```
这样Webpack在js文件中发现了`import React from 'react'`就不会再去构建一遍（当然要在html文件中用script标签引入react.min.js），但是这么做还有两个问题，第一如果另外一个第三方内部引用了react，那么在webpack构建这个第三方库时，又会把react构建一遍，第二个问题未必是所有的库都提前已经给你了一个构建好的版本（也就是没有提供min.js），所以我们在引用后，每次build时都会去构建一遍这个第三方库，这样就会造成构建变慢。所以在使用了这种方式后我们只需要在我们所依赖的第三方库发生变化的时候，去执行一遍`npm run dll`。

所以这个时候我们就可以使用动态链接库。具体配置请查看https://webpack.docschina.org/plugins/dll-plugin/

### 使用[HappyPack](https://github.com/amireh/happypack)

由于有大量文件需要解析和处理，所以构建是文件读写和计算密集型的操作， 特别是当文件数量变多后， Webpack 构建慢的问题会显得更为严重。运行在 Node. 之上的 Webpack 是单线程模型的，也就是说 Webpack 需要一个一个地处理任务，不能同时处理多个任务。
Happy Pack 将任务分解给多个子进程去并发执行，子进程处理完后再将结果发送给主进程。由于 JavaScript 是单线程模型，所以要想发挥多核 CPU 的功能，就只能通过多进程实现，而无法通过多线程实现。

```
const HappyPack =require('happypack');
```

```
module: {
        rules: [
            {
                test: /\.js$/,
                exclude: path.resolve(__dirname, '../node_modules'),
                use: ['happypack/loader?id=babel']
            }
         ]
},
plugins: [
        new HappyPack({
            id: 'babel',
            loaders: [ 'babel-loader?cacheDirectory' ]
        })
]
```
            

### 使用 ParallelUglifyPlugin

在使用 Webpack 构建出用于发布到线上的代码时，都会有压缩代码这 流程 。最常见
JavaScript 代码压缩工具是 [UglifyJS](https://github.com/mishoo/UglifyJS2)，并且 Webpack也内置了它若用过 UglifyJS ，则我们 定会发现能很快通过它构建用于开发环境的代码，但在构建用于线上的代码时会卡在一个时间点迟迟没有反应，其实在这个卡住的时间 点正在进行的就是代码压缩。

由于压缩 JavaScript 代码时，需要先将代码解析成用 Object 抽象表示的 AST 语法树，
再去应用各种规则分析和处理 ST ，所以导致这个过程的计算量巨大 耗时非常多。

当Webpack 有多个 JavaScript 文件需要输出和压缩时 原本会使用 UglifyJS 去一个一个压缩再输出，但是 Paralle!Uglify Plugin 会开启多个子进程，将对多个文件的压缩工作分配给多个子进程去完成，每个子进程其实还是通过 UglifyJS 去压缩代码，但是变成了并行执行。所以Paralle!Uglify Plugin 能更快地完成对多个文件的压缩工作。

当然也可以使用[UglifyjsWebpackPlugin](https://webpack.docschina.org/plugins/uglifyjs-webpack-plugin/)

### 使用Tree Shaking
Tree Shaking 可以用来剔除 JavaScript 中用 不上的死代码。它依赖静态的 ES6 模块化
语法，例如通过 import和 export 导入、导出。首先在使用Tree Shaking 的时候，需要将ES6 模块化的代码提交给 Webpack，而不是babel（因为Tree Shaking依赖于ES6的模块化语法）配置如下：

```json
{
	"presets": [
		[
			"env",
			{
				"modules": false
			}
		]
		
	]
}
```

其中，"modules"： false 的含义是关闭 babel 的模块转换功能，保留原本的 ES6模块化语法。然后接入UglifyJS。`webpack -- display-used-exports --optimize-minimize`

> 在项目中使用大量的第 方库时，我们会发现 Tree Shaking 似乎不生效了，原因是大部分 Npm 中的代码都采用了 CommonJS 语法，这导致 Tree Shaking 无法正常工作而降级处理。

但幸运的是，有些库考虑到了这一点，这些库在发布到 Npm 上时会同时提供两份代码，一份采用CommonJs 模块化语法，一份采用 ES6 模块化语法。并且在 package.json 文件中分别指出这两份代码的入口。以Redux 库为例，其发布到 Npm 上的目录结构为

![](http://images.djl.pub/19-1-3/32677299.jpg)

然后我们可以通过 mainFields 字段配置优先使用哪个作为入口文件
```
module.exports = { 
	resolve: { 
		// 针对 Npm 中的第 方模块优先采用 snext main 中指向的 ES6 模块化语法的文件
		mainFields: ['jsnext:main', 'browser', 'main'] 
	}
};
```
以上配置的含义是优先使用 jsnext:main 作为入口，如果不存在， jsnext:main 就会采用browser或者main作为入口文件。

### 提取公共代码
在一个多页应用中，可能使用过一些相同的代码，最常见的就是utils.js。如果每个页面都去包含这一段代码，必定会增加资源大小，我们可以将这一部分公共代码提取出来，成为一个单独的bundle，而且由于用户在访问一个页面后很可能会访问另外一个页面，那么在访问前一个页面的时候就会将那个公共的bundle进行缓存，那么在访问下一个页面直接从缓存里面读取就可以。

一般我们的应用中js代码分为三个部分，业务代码、业务公共代码、第三方库（为了长期缓存）。由于一般情况我们采用的第三库是不会改变的那么打的bundle的hash也只也不会改变，这样就可以将这个bundle长期保存。

#### Webpack3提取公共代码
```
const ComrnonsChunkPlugin = 
require ('webpack/lib/optimize/CommonsChunkPlugin');

new CommonsChunkPlugin({ 
// 从哪些 Chunk 中提取
chunks : [ 'a', 'b'], 
// 提取出的公共部分形成 个新的 Chunk
name: 'common'
});
```

#### Webpack4提取公共代码

```javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: 'async',
      minSize: 30000,
      maxSize: 0,
      minChunks: 1,
      maxAsyncRequests: 5,
      maxInitialRequests: 3,
      automaticNameDelimiter: '~',
      name: true,
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

#### 区别 
它们的区别就在于，CommonChunksPlugin 会找到多数模块中都共有的东西，并且把它提取出来（common.js），也就意味着如果你加载了 common.js，那么里面可能会存在一些当前模块不需要的东西。

而 SplitChunksPlugin 采用了完全不同的方法，它会根据模块之间的依赖关系，自动打包出很多很多（而不是单个）通用模块，可以保证加载进来的代码一定是会被依赖到的。


### 按需加载
在为单页应用做按需加载优化时， 一般采用以下原则。
- 将整个网站划分成 个个小功能，再按照每个功能的相关程度将它们分成几类
- 将每 类合并为一个 Chunk ，按需加载对应的 Chunk
- 不要按需加载用户首次打开网站时需要看到的画面所对应的功能，将其放到执行入口所在的 Chunk 中，以减少用户能感知的网页加载时间。
- 对于不依赖大量代码的功能点，例如依赖 Chart.js 去画图表、依赖 flv.js 去播放视
频的功能点，可再对其进行按需加载。

被分割出去的代码的加载需要一定的时机去触发，即当用户操作到了或者即将操作到对应的功能时再去加载对应的代码。被分割出去的代码的加载时机需要开发者根据网页的需求去衡量和确定。

如果要实现点击某一个按钮之后，去加载一段js，webpack可以这样配置
```
window.document.getElementByid('btn')
.addEventListener ('click', function () {
	// 在按钮被单击后才去加载 show js 文件，文件加载成功后执行文件导出的函数
	import(/* webpackChunkName : "show" */'./show').then ((show) => { 
		show ();
	})
});
```

```javascript
module.exports = {
// JavaScript 执行入口文件
	entry: {
		main:'./main.js', 
	},
	output: {
		// 为从 entry 中配直生成的 Chunk 配置输出文件的名称
		filename:'[name].js', 
		// 为动态加载 Chunk 配置输出文件的名称
		chunkFilename: '[name].js'
	}
};
```
其中最关键的一句是：`import(/* webpackChunkName : ""show" */'./ show')`
Webpack 内置了对 import (*)语句的支持，当 Webpack遇到了类似的语句时会这样处理：
- 以./show.js 为入口重新生成一个 Chunk;
- 当代码执行到 import 所在的语句时才去加载由 Chunk 对应生成的文件；
- import 返回一个 Promise ，当文件加载成功时可以在 Promise then 方法中获取
show.js 导出的内容。

`/* webpackChunkName: "show" */`的含义是为动态生成的 Chunk 赋予一个名称，
以方便我们追踪和调试代码 。如果不指定动态生成的 Chunk 的名称，则其默认的名称将会是[id] .js。`/* webpackChunkName: "show" */`，是在 Webpack 中引入的新特性，在Webpack3 之前是无法为动态生成的 Chunk 赋予名称的。

### 参考文章

[深入浅出Webpack](http://webpack.wuhaolin.cn/)

https://jeffjade.com/2017/08/12/125-webpack-package-optimization-for-speed/

http://www.cnblogs.com/imwtr/p/7801973.html

https://zhuanlan.zhihu.com/p/37148975?utm_source=wechat_session&utm_medium=social&utm_oi=32383348768768&from=timeline&isappinstalled=0

### 最后（欢迎大家关注我）
[DJL箫氏个人博客](http://djl.pub/)
[博客GitHub地址](https://github.com/djlxiaoshi/blog/issues)
[简书](https://www.jianshu.com/u/d8657fcf1678)
[掘金](https://juejin.im/user/57183fcac4c9710054bc2fcf)
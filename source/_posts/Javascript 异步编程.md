---
title: Javascript 异步编程
date: 2017-12-13 18:06:25
tags: 
    - 异步编程
    - promise
    - generator
    - async/await
---

所谓"异步"，简单说就是一个任务分成两段，先执行第一段，然后转而执行其他任务，当第一段有了执行结果之后，再回过头执行第二段。JavaScript采用异步编程原因有两点，一是JavaScript是单线程，二是为了提高CPU的利用率。在提高CPU的利用率的同时也提高了开发难度，尤其是在代码的可读性上。
<!--more-->

```javascript
console.log(1);

setTimeout(function () {
  console.log(2);
});

console.log(3);
```

![JavaScript异步执行示意图](http://images.djl.pub/17-12-14/94900806.jpg)


## callback
最开始我们在处理异步的时候，采用的是callback回调函数的方式
```javascript
asyncFunction(function(value){
    // todo
})
```
在一般简单的情况下，这种方式是完全够用的，但是如果碰到稍微复杂的场景，就有些力不从心，例如当异步嵌套过多的时候。

### 回调金字塔
但是当我们的异步操作比较多，而且都依赖于上一步的异步的执行结果，那么我们就会产生回调金字塔，难于阅读
```javascript
step1(function (value1) {
    step2(function(value2) {
        step3(function(value3) {
            step4(function(value4) {
                // Do something with value4
            });
        });
    });
});
```
当然为了改进这种层层嵌套的写法，我们有几种方式
1 命名函数

```javascript
function fun1 (params) {
  // todo
  asyncFunction(fun2);
}

function fun2 (params) {
  // todo
  asyncFunction(fun3)
}

function fun3 (params) {
  // todo
  asyncFunction(fun4)
}

function fun4 (params) {
  // todo
}

asyncFunction(fun1)
```

2 基于事件消息机制的写法
```javascript
eventbus.on("init", function(){
    operationA(function(err,result){
        eventbus.dispatch("ACompleted");
    });
});
 
eventbus.on("ACompleted", function(){
    operationB(function(err,result){
        eventbus.dispatch("BCompleted");
    });
});
 
eventbus.on("BCompleted", function(){
    operationC(function(err,result){
        eventbus.dispatch("CCompleted");
    });
});
 
eventbus.on("CCompleted", function(){
    // do something when all operation completed
});
```

当然也可以利用模块化来处理，使得代码易于阅读。以上这三种方式都只是在代码的可读性上面做了改进，但是并没有解决另外一个问题就是异常捕获。

### 错误栈

```javascript
function a () {
    b();
}

function b () {
    c();
}

function c () {
    d();
}

function d () {
    throw new Error('出错啦');
}

a();
```

![Node错误打印](http://images.djl.pub/17-12-14/34642187.jpg)

从上面的图我们可以看到有一个比较清晰的错误栈信息，a调用b - b调用c - c调用d ，在d中抛出了一个异常。也就是说在JavaScript中在执行一个函数的时候首先会压入执行栈中，执行完毕后会移除执行栈，FILO的结构。我们可以很方便的从错误信息中定位到出错的地方。

```javascript
function a() {
    b();
}

function b() {
    c(cb);
}

function c(callback) {
    setTimeout(callback, 0)
}

function cb() {
    throw new Error('出错啦');
}

a();
```

![包含异步的错误栈](http://images.djl.pub/17-12-14/24038039.jpg)

从上图我们可以看到只打印出了是在一个setTimeout中的回调函数中出现了异常，执行顺序是跟踪不到的。

### 异常捕获
回调函数中的异常是不能够捕捉到的，因为是异步的，我们只能在回调函数中使用try catch捕获，也就是我注释的部分。
```javascript
function a() {
    setTimeout(function () {
        // try{
            throw new Error('出错啦');
        // } catch (e) {
        
        // }
        
    }, 0);
}

try {
    a();
} catch (e) {
    console.log('捕捉到异常啦，好高兴哦');
}
```
但是try catch只能捕捉到同步的错误，不过在回调中也有一些比较好的错误处理模式，例如error-first的代码风格约定，这种风格在node.js中广泛被使用 。

```javascript
function foo(cb) {
  setTimeout(() => {
    try {
      func();
      cb(null, params);
    } catch (error) {
      cb(error);
    }
    
  }, 0);
}

foo(function(error, value){
    if(error){
        // todo
    }
    // todo
});
```
但是这么做也很容易陷入恶魔金字塔中。

## Promise

### [规范简述](http://www.ituring.com.cn/article/66566) 

- promise 是一个拥有 then 方法的对象或函数。
- 一个promise有三种状态 pending, rejected, resolved 状态一旦确定就不能改变，且只能够由pending状态变成rejected或者resolved状态，reject和resolved状态不能相互转换。
- 当promise执行成功时，调用then方法的第一个回调函数，失败时调用第二个回调函数。
- promise实例会有一个then方法，这个then方法必须返回一个新的promise。

### 基本用法
```javascript
// 异步操作放在Promise构造器中
const promise1 = new Promise((resolve) => {
    setTimeout(() => {
        resolve('hello');
    }, 1000);
});

// 得到异步结果之后的操作
promise1.then(value => {
  console.log(value, 'world');
}, error =>{
  console.log(error, 'unhappy')
});
```

### 异步代码，同步写法
```javascript
asyncFun()
    .then(cb)
    .then(cb)
    .then(cb)
```
promise以这种链式写法，解决了回调函数处理多重异步嵌套带来的回调地狱问题，使代码更加利于阅读，当然本质还是使用回调函数。

### 异常捕获
前面说过如果在异步的callback函数中也有一个异常，那么是捕获不到的，原因就是回调函数是异步执行的。我们看看promise是怎么解决这个问题的。
```javascript
asyncFun(1).then(function (value) {
    throw new Error('出错啦');
}, function (value) {
    console.error(value);
}).then(function (value) {

}, function (result) {
  console.log('有错误', result);
});
```
其实是promise的then方法中，已经自动帮我们try catch了这个回调函数，实现大致如下。
```javascript
Promise.prototype.then = function(cb) {
    try {
        cb()
    } catch (e) {
       // todo
       reject(e)
    }
}
```
then方法中抛出的异常会被下一个级联的then方法的第二个参数捕获到（前提是有），那么如果最后一个then中也有异常怎么办。
```javascript
Promise.prototype.done = function (resolve, reject) {
    this.then(resolve, reject).catch(function (reason) {
        setTimeout(() => {
           throw reason;
        }, 0);
    });
};
```
```javascript
 asyncFun(1).then(function (value) {
     throw new Error('then resolve回调出错啦');
 }).catch(function (error) {
     console.error(error);
     throw new Error('catch回调出错啦');
 }).done((reslove, reject) => {});
```
我们可以加一个done方法，这个方法并不会返回promise对象，所以在此之后并不能级联，done方法最后会把异常抛到全局，这样就可以被全局的异常处理函数捕获或者中断线程。这也是promise的一种最佳实践策略，当然这个done方法并没有被ES6实现，所以我们在不适用第三方Promise开源库的情况下就只能自己来实现了。为什么需要这个done方法。
```javascript
const asyncFun = function (value) {
  return new Promise(function (resolve, reject) {
    setTimeout(function () {
      resolve(value);
    }, 0);
  })
};


asyncFun(1).then(function (value) {
  throw new Error('then resolve回调出错啦');
});
```
`(node:6312) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): Error: then resolve回调出错啦`
`(node:6312) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code`
我们可以看到JavaScript线程只是报了一个警告，并没有中止线程，如果是一个严重错误如果不及时中止线程，可能会造成损失。

### 局限
promise有一个局限就是不能够中止promise链，例如当promise链中某一个环节出现错误之后，已经没有了继续往下执行的必要性，但是promise并没有提供原生的取消的方式，我们可以看到即使在前面已经抛出异常，但是promise链并不会停止。虽然我们可以利用返回一个处于pending状态的promise来中止promise链。
```javascript
const promise1 = new Promise((resolve) => {
    setTimeout(() => {
        resolve('hello');
    }, 1000);
});

promise1.then((value) => {
    throw new Error('出错啦!');
}).then(value => {
    console.log(value);
}, error=> {
    console.log(error.message);
    return result;
}).then(function () {
    console.log('DJL箫氏');
});
```


### 特殊场景
- 当我们的一个任务依赖于多个异步任务，那么我们可以使用Promise.all
- 当我们的任务依赖于多个异步任务中的任意一个，至于是谁无所谓，Promise.race

上面所说的都是ES6的promise实现，实际上功能是比较少，而且还有一些不足的，所以还有很多开源promise的实现库，像q.js等等，它们提供了更多的语法糖，也有了更多的适应场景。

###  核心代码

```javascript
var defer = function () {
    var pending = [], value;
    return {
        resolve: function (_value) {
            value = _value;
            for (var i = 0, ii = pending.length; i < ii; i++) {
                var callback = pending[i];
                callback(value);
            }
            pending = undefined;
        },
        then: function (callback) {
            if (pending) {
                pending.push(callback);
            } else {
                callback(value);
            }
        }
    }
};
```
当调用then的时候，把所有的回调函数存在一个队列中，当调用resolve方法后，依次将队列中的回调函数取出来执行

```javascript
var ref = function (value) {
    if (value && typeof value.then === "function")
        return value;
    return {
        then: function (callback) {
            return ref(callback(value));
        }
    };
};
```
这一段代码实现的级联的功能，采用了递归。如果传递的是一个promise那么就会直接返回这个promise，但是如果传递的是一个值，那么会将这个值包装成一个promise。

## generator

### 基本用法
```javascript
function * gen (x) {
    const y = yield x + 2;
    // console.log(y);  // 猜猜会打印出什么值
}

const g = gen(1);
console.log('first', g.next());  //first { value: 3, done: false }
console.log('second', g.next()); // second { value: undefined, done: true }
```


通俗的理解一下就是yield关键字会交出函数的执行权，next方法会交回执行权，yield会把generator中yield后面的执行结果，带到函数外面，而next方法会把外面的数据返回给generator中yield左边的变量。这样就实现了数据的双向流动。

### generator实现异步编程
我们来看generator如何是如何来实现一个异步编程（*）
```javascript
const fs = require('fs');

function * gen() {
    try {
        const file = yield fs.readFile;
        console.log(file.toString());
    } catch(e) {
        console.log('捕获到异常', e);
    }
}

// 执行器
const g = gen();

g.next().value('./config1.json', function (error, value) {
  if (error) {
    g.throw('文件不存在');
  }
  g.next(value);
});
```
那么我们next中的参数就会是上一个yield函数的返回结果，可以看到在generator函数中的代码感觉是同步的，但是要想执行这个看似同步的代码，过程却很复杂，也就是流程管理很复杂。那么我们可以借用TJ大神写的co。
### generator 配合 co
下面来看看如何使用：
```javascript
const fs = require('fs');
const utils = require('util');
const readFile = utils.promisify(fs.readFile);
const co = require('co');

function * gen(path) {
    try {
        const file = yield readFile('./basic.use1.js');
        console.log(file.toString());
    } catch(e) {
        console.log('出错啦');
    }
}

co(gen());
```
我们看到使用co这个执行器配合generator和promise会非常方便，非常类似同步写法，而且异步中的错误也能很容易被try catch到。这里之所以要使用utils.promisify这个工具函数将普通的异步函数转换成一个promise，是因为co may only yield a chunk, promise, generator, array, or object。使用co 配合generator最大的一个好处就是错误可以try catch 到。

## async/await
先来看一段async/await的异步写法
```javascript
const fs = require('fs');
const utils = require('util');
const readFile = utils.promisify(fs.readFile);
async function readJsonFile() {
    try {
        const file = await readFile('../generator/config.json');
        console.log(file.toString());
    } catch (e) {
        console.log('出错啦');
    }

}

readJsonFile();
```
我们可以看到async/await的写法十分类似于generator，实际上async/await就是generator的一个语法糖，只不过内置了一个执行器。并且当在执行过程中出现异常，就会停止继续执行。当然await后面必须接一个promise，而且node版本必须要`>=7.6.0`才可以使用，当然低版本也可以采用babel。

## 补充
在开发过程中我们常常手头会同时有几个项目，那么node的版本要求很有可能是不同的，那么我们就需要安装不同版本的node，并且管理这些不同的版本，这里推荐使用[nvm](https://github.com/coreybutler/nvm-windows)，[下载](https://github.com/coreybutler/nvm-windows/releases)好nvm，安装，使用nvm list 查看node版本列表。使用nvm use 版本号 进行版本切换。

在Node.js中捕获漏网之鱼
```javascript
process.on('uncaughtException', (error: any) => {
    logger.error('uncaughtException', error)
})
```
在浏览器环境中捕获漏网之鱼
```javascript
window.addEventListener('onrejectionhandled', (event: any) => {
    console.error('onrejectionhandled', event)
})
```

## 参考文章
[Promise中文迷你书](http://liubin.org/promises-book/)
[剖析Promise内部结构，一步一步实现一个完整的、能通过所有Test case的Promise类](https://github.com/xieranmaya/blog/issues/3)
[深入理解Promise实现细节](http://acgtofe.com/posts/2015/03/promise-from-zero)
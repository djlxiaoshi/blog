---
title: This解读
date: 2018-03-13 09:43:08
tags: 
    - JavaScript
    - This
---

在很长的一段时间之内，我一直以为作用域就是上下文，这也就对JavaScript中的this理解增加了很多麻烦，所以这篇文章开篇第一个要陈诉的概念就是作用域和上下文不是一个概念。作用域(scope) 是指变量的可访问性，上下文是来决定this。（注意执行期上下文指的是作用域，这是JavaScipt规范，所以得遵守）
<!--more-->
在JavaScript中只有两种作用域，一种是全局作用域，另一个就是函数作用域。上下文则会this息息相关，而this是在运行的时候进行绑定的，它的上下文取决于函数在哪里被调用，this的绑定和函数声明的位置没有任何关系。

当一个函数被调用时，会创建一个活动记录（即执行上下文）。这个活动会包含函数在哪里被调用，函数的调用方法，传入的参数信息等信息。this就是记录的其中一个属性，会在函数的执行过程中用到。

当然这句话出自《你不知道的JavaScript（上卷）》，在这里强烈推荐这本书，字字珠玑。


> **再次强调**：this实际上是在函数被调用的时候发生绑定，它指向什么完全取决于函数在哪里调用。

## 调用位置

接下来我们看看函数调用包括哪几种情况，只有正确的知道函数调用的位置，才能正确的明白this的指向问题。

### 默认绑定(全局调用)

```javascript
var a = 2;
function foo() {
    console.log(this.a)
}
foo();
```
以上就是默认绑定，foo函数是直接调用的。

### 隐式绑定

```javascript
b = 2;
var obj = {
    b: 3,
    foo: foo
}

function foo () {
    console.log(this.b);
}
obj.foo(); // 3
```
这里为什么叫做隐式绑定，因为这个foo函数无论是在obj里面声明还是在obj外面声明，他实际上都是不属于obj这个对象的（obj只是记录了foo这个属性的引用值），但是最后在执行的时候this却被绑定到了obj这个对象上下文中。当然如果有多个对象链式调用，this只会绑定到最后一层。`obj2.obj1.foo()`，this是绑定到obj1这个对象上下文中。

当然这里有一个注意点
```javascript
var obj = {
    b: 3,
    foo: foo
}
function foo () {
    console.log(this.b);
}
var bar = obj.foo;

bar(); // 2 
```
这里实际上bar直接是foo的引用，就相当于`var bar = obj.foo = foo`，我们打印一下可以发现
```javascript
console.log(bar === foo && foo === obj.foo && bar === obj.foo) // true
```
所以此时就和第一种默认绑定一样，bar函数是直接在全局上下文中被调用的，所以this会指向全局。

还有一种就是嵌套函数了
```javascript
b = 2;
var obj = {
    b: 3,
    foo: foo
}
function foo () {
    console.log('foo', this.b);// 3
    foo2();
}
function foo2() {
    console.log('foo2', this.b); // 2
}
obj.foo();
    
```
实际上foo2也是直接被（window）调用了。

### 显示绑定call，apply，bind

通过call，apply，bind函数可以强制某个函数在哪个对象（或者上下文）中被调用
```
b = 2;

var obj = {
    b: 3,
    foo: foo
}

function foo () {
    console.log('foo', this.b);
}

foo.call(obj); // 3
```

当然如果你传入的是一个基本类型的值，那么JavaScript会把它转换成它的对象形式。

### new绑定

说到new操作符，就不得不说它的内部工作原理了，我们在执行new操作的时候究竟执行了什么。
> 1 创建一个全新的对象 var obj = {}
> 2 这个新对象的原型会被执行[[原型]]连接 obj[[prototype]] = Fun.prototye
> 3 这个新对象会绑定到函数调用的this Fun.bind(obj)
> 4 如果函数没有返回其他对象，那么会返回这个新创建的对象 return obj;

所以new绑定实质还是显式绑定。

总结一下我们可以按照下面的顺序进行判断
> 1 函数是否在new中调用（new 绑定），如果是this绑定的就是返回的新对象
> 2 函数是否通过call、apply（显式绑定）如果是this绑定的是那个指定的对象
> 3 函数是否在某个上下文对象中调用（隐式绑定），如果是，this绑定的是那个上下文无关文法对象
> 4 如果都不是那么就是默认绑定，this绑定的就是全局对象或者undefined（严格模式）

### 例外

凡事总有例外，如果你把null、undefined作为this的绑定对象传入call、apply或者bind那么实际上，这些值在执行的时候会被忽略，实际使用的是默认绑定。那么什么情况下我们会去绑定一个null或者undefined的呢？一种就是用apply来展开一个数组，当然这种方法的确很实用(不过在ES6中出现了...操作符来展开数组)。
```javascript
function foo(a, b) {return a + b}
foo.apply(null, [2, 3]);
```
箭头函数，箭头函数中的this是根据外层作用域来决定this的，也就是说箭头函数中的this就和箭头函数在哪里声明有关系了。
```javascript
a = 2;

var obj = {
    a: 3,
    foo: foo
}

function foo () {
    return () => {
        console.log(this.a);
    };  
}


var fun = foo.call(obj);

fun(); // 3 此时箭头函数的外层作用域为foo，foo函数的this被绑定在了obj对象上
```

```
a = 2;

var obj = {
    a: 3,
    foo: foo
}

var arrowFun = () => {
    console.log(this.a);
}

function foo () {
    return arrowFun;    
}


var fun = foo.call(obj);

fun(); //2 箭头函数的外层作用域为全局作用域，全局作用域中的this指向全局上下文
```


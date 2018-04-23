---
title: JavaScript 继承
date: 2018-04-23 20:24:08
tags: 
    - JavaScript
    - 继承
---

继承是JS中非常内容，原因就是JS没有地道的继承方式，我们只能通过各种方式来模拟面向对象中的继承。下面介绍几种常见的继承方式及其不足。
<!--more-->

### 构造函数继承
```javascript
function Parent1 (){
	this.name = 'parent'
}

function Child1() {
	Parent1.call(this);
	this.type = 'child1';
}
```
缺点：Parent1 原型链上的东西并不会继承，这种方式，所以只实现了部分继承，如果父类的属性都在构造函数中，没问题，但是如果有一部分在原型链上，那么就继承不了，为了解决这个不足我们就要使用原型链来进行原型继承。

### 原型继承
```javascript
function Parent2() {
this.name = 'Parent2';
this.res = [1, 2, 3]
}

function Child2 () {
this.type = 'Child2'
}

Child2.prototype = new Parent2()

var child2 = new Child2();
var child3 = new Child3();

child2.res.push(4);
console.log(child3.res) // 1,2,3,4
```
缺点  : 这种方式缺点也很明显，实例的两个对象，如果一个对象改变res的值，那么另一个对象 的res属性也会被改变（这两个对象共享一个原型），这违背了独立性。

## 组合
```javascript
function Parent3() {
	this.name = 'parent3';
	this.res = [1, 2, 3];
}

function Child3() {
	Parent3.call(this);
	this.type = 'child3';
}

Child3.prototype = Parent3.prototype;
var child = new Child3();
```
缺点：此时`child.constructor`并不是`Child3`；而是`Parent3`，这是因为当我们想获取`child.constructor`实际上是访问`Child3.prototype.constructor`（也就是说constructor这个属性是存在于原型上，并不是直接在child这个对象上），而`Child3.prototype`此时等于`Parent3.prototype`,所以最后`constructor`的属性值为`Parent3`。

### 组合优化
```javascript
function Parent4() {
this.name = 'parent4';
this.res = [1, 2, 3];
}

function Child4() {
 Parent4.call(this);
 this.type = 'child4';
}

Child4.prototype = Object.create(Parent4.prototype);
// Child4.prototype = Parent4.prototype;
Child4.prototype.constructor = Child4;	
```
在这里我们加上这句`Child4.prototype = Object.create(Parent4.prototype);`的目的是为了隔离子类原型和父类原型，现在就是`Child4.prototype.__proto__ === Parent4.prototype`，如果我们不加，直接修正子类的构造函数（`Child4.prototype.constructor = Child4;`）那么也会把父类的`Parent4.prototype.constructor`更改成`Child4`，因为此时`Child4.prototype`和`Parent4.prototype`指向同一地址。

> 注: 如果这里不支持Object.create,我们可以采用下面的plolly
> `function F(){}
> F.prototype = Parent4.prototype
> Child4.prototype = new F()`

缺点：貌似还没有实现静态属性的继承

### 实现静态属性的继承
```javascript
function Parent() {
  this.age = 20
}

Parent.sex = 'male';
Parent.habby = 'badminton';

function Child() {
  Parent.call(this);
  this.type = 'Child';

 if (Object.setPrototypeOf) {
   Object.setPrototypeOf(Child, Parent)
 } else if (Child.__proto__) {
   Child.__proto__ = Parent
 } else {
   for (var attr in Parent) {
     if (Parent.hasOwnProperty(attr) && !(attr in Child)) {
       Child[attr] = Parent[attr]
     }
   }
 }   
}

Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;

var child = new Child();
var parent = new Parent();
console.log(child)
for (var key in Child) {
  console.log(key)
}

console.log(child.constructor)
console.log(parent.constructor)
console.log(child instanceof Child)
console.log(child instanceof Parent)

```

看似完美了，但是还有一个问题，就是如果后续父类继续添加一些静态的方法，是不会自动同步到子的静态方法上面去的。

#### 最后（欢迎大家关注我）
[DJL箫氏个人博客](http://djl.pub/)
[博客GitHub地址](https://github.com/djlxiaoshi/blog/issues)
[简书](https://www.jianshu.com/u/d8657fcf1678)
[掘金](https://juejin.im/user/57183fcac4c9710054bc2fcf)
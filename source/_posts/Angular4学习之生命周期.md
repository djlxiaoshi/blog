---
title: Angular4学习之生命周期
date: 2017-08-17 10:47:46
tags: 
    - Angular4 
    - 生命周期 
---

组件生命周期就是组件从创建到销毁过程中的一些关键点，Angular为我们提供了一些钩子函数，允许我们在组件的相应生命周期到来时进行必要操作。首先我们来看看Angular的生命周期示意图：
<!-- more -->
![Angular生命周期示意图](http://ok3x4ia9b.bkt.clouddn.com/17-8-5/94857391.jpg)

从上图我们可以发现组件在构建过程中最先执行的是`constructor`构造函数，但是在构造函数中我们不应该来进行一些比较复杂的初始化操作（一般进行变量初始化操作）。下面我们来一一解释一下每个钩子函数所代表的意义和作用。

- ngOnChanges: 该方法在ngOnInit方法之前调用，或者由@input装饰器装饰的输入数据变化时调用。但是注意如果这里输入数据是一个引用类型，那么它检测的是这个对象的引用。也就是说你改变对象的某一个属性值并不会触发这个钩子函数的执行，同样你调用数组的splice等等方法也是不会改变数组的引用（这里和Vue有所不同），至于如何达到我们所想的结果，我们下面说。ngOnChanges(changes)会有一个参数格式如下：
```javascript
{
	changeData1: {
		currentValue: XXX,
		firstChange: false/true,
		previousValue: XXX
	},
	changeData2: {
		currentValue: XXX,
		firstChange: false/true,
		previousValue: XXX
	}
}
```


- ngOnInit:  用于数据绑定输入属性之后初始化组件。在第一次ngOnChanges调用之后调用，一般在这里进行一些组件较复杂的初始化操作。
- ngDoCheck: 在每次变化监测之后调用。在一个变化监测周期内，不论数据是否变化都会执行。也就是说Angular监测的并不是数据的变动，而是可能引起数据发送变化的行为（例如：用户的各种操作，ajax请求，定时器等等）。相比ngOnChanges而言，ngDoCheck监测粒度更加小，所以在前面讲到的对象单个属性值变化时不会触发ngOnChanges钩子执行，那么我们可以通过ngDoCheck来进行监听，因为数据的变动一定会伴随相应的行为（且都是一些异步操作），但是我们也要注意像mousemove这些事件可能会频发触发调用ngDoCheck。而且在一个组件中我们一般不会同时实现ngOnchanges和ngDoCheck这两个接口。

- ngAfterContentInit: 在组件中使用`<ng-content>`将外部内容嵌入到组件视图后调用，它在第一次ngDoCheck执行后调用，且只调用一次。
- ngAfterContentChecked: 在组件使用`<ng-content>`引入外部元素的情况下，Angular在这些外部内容嵌入到组件视图之后调用，或者每次变化监测时候调用。
- ngAfterViewInit: 当Angular创建了视图及其子视图之后调用
- ngAfterViewChecked: 在创建了组件视图及其子视图之后被调用一次，并且在每次子组件变化监测时会被调用
- ngOnDestroy: 在销毁指令/组件之前调用，用来销毁一些不会被垃圾回收器回收的资源（已订阅的观察者事件、绑定的DOM事件，定时器等等）。

在前面说到通过@input绑定的输入值如果是一个对象，那么单纯的改变对象的某个属性值是不会触发ngOnChanges钩子函数的，那么解决办法有如下几种：
- 实现ngDoCheck钩子函数 
- 可以改变对象的引用例如： `object = Object.assign({}, object)`、`arr = [].concat(arr)`
- 使用Immutable对象来传值 （具体使用看下面的示例代码）

在通常情况我们单纯改变对象的某个属性值或者对象结构，对象的引用是不会改变的，但是Immutable能够保证只要当对象的值或者结构改变时，对象的引用必定发生改变。

下面我们通过一段示例代码来验证一下Angular的生命周期钩子函数的运行情况
```
<!--child.html--->
<ng-container>
  <h1>lifecircle test</h1>
  <h3>姓名：{{obj.get('name')}}</h3>
  <h3>年龄：{{obj.age}}</h3>
  <ng-content></ng-content>
</ng-container>
```

```javascript
// child.component.ts
import {
  Component, OnInit, OnChanges, DoCheck, AfterViewInit, AfterViewChecked, AfterContentInit,
  AfterContentChecked, OnDestroy, SimpleChanges, Input
} from '@angular/core';

@Component({
  selector: 'app-lifecircle',
  templateUrl: './lifecircle.component.html',
  styleUrls: ['./lifecircle.component.scss']
})
export class LifecircleComponent implements OnChanges,OnInit, DoCheck, AfterContentInit, AfterContentChecked ,AfterViewInit, AfterViewChecked, OnDestroy {
  @Input() obj;
  constructor() { }

  ngOnChanges(changes: SimpleChanges): void {
    console.log('ngOnChanges', changes)
  }

  ngOnInit() {
    console.log('ngOnInit')
  }

  ngDoCheck(): void {
    console.log('ngDoCheck')
  }

  ngAfterContentInit(): void {
    console.log('ngAfterContentInit')
  }

  ngAfterContentChecked(): void {
    console.log('ngAfterContentChecked')
  }

  ngAfterViewInit(): void {
    console.log('ngAfterViewInit')
  }

  ngAfterViewChecked(): void {
    console.log('ngAfterViewChecked')
  }

  ngOnDestroy(): void {
    console.log('ngOnDestroy')
  }

}
```

```
<!--parent.html-->
<ng-container>
  <h1>生命周期示例</h1>
  <app-lifecircle [obj]="obj">
    <h4>这是插入的内容</h4>
  </app-lifecircle>

  <button pButton type="button" (click)="changeObj()" label="ngAfterViewChecked"></button>
</ng-container>
```

```javascript
// parent.component.ts
import {Map} from 'immutable'

import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-blog',
  templateUrl: './blog.component.html',
  styleUrls: ['./blog.component.scss']
})
export class BlogComponent implements OnInit {
   obj

  constructor() { }
  ngOnInit() {
    this.obj = Map({name: 'djlxs', habits: {item: '羽毛球'}})
  }

  changeObj() {
    this.obj = this.obj.set('name', 'djl');
  };
}
```


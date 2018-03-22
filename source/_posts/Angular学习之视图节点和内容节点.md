---
title: Angular4学习之视图节点和内容节点
date: 2017-08-21 20:38:40
tags:
    - Angular4
---

通常我们在自定义一个组件后，我们会按照如下方式使用

```
<my-component></my-component>
```
但是有时候我们组件或许有些差异，我们可以通过输入变量来控制组件的DOM结构，但是这种方式不够灵活，你可能需要列举出组件的所有差异化的表现，从而会有很多的配置选项，我们需要一种更加灵活的实现方式，那就是`ng-content`。你需要什么特定的结构，你自己决定，而不是我们写好各种组合，由你来选择。这个有点类似插件。
<!--more-->

## 使用

```
hello world
<ng-content></ng-content>
hello djlxs
```
父组件中引用：
```
<my-component>
hello hello hello
</my-component>
```
那么就会用我们在`my-component`中的html代码替换掉`ng-content`,但是有时候我们可能会有多处例如：
```
<div>
  <ng-content selector="app-panel-title"></ng-content>
  <ng-content selector="app-panel-content"></ng-content>
</div>
```
那么我们此时就需要使用selector来进行标识：
```
<div>
 <app-panel>
   <app-panel-title></app-panel-title>
   this is boundary
   <app-panel-content></app-panel-content>
 </app-panel>
</div>
```
seletor对应的值会匹配同名的标签。这样看来使用是非常简单的。

## ViewChild 和 ContentChild
在使用这两个装饰器的时候我们首先需要明白两个概念：视图子节点和内容子节点。
- 视图子节点：组件模板里面用到的子标签 (如下面的h1和`my-component`)
```javascript
component({
    template: `
        <h1>Hello world</h1>
        <my-component></my-component>
    `
})
```
- 内容子节点：内嵌在组件标签里面的子标签(如下面的`my-title`)
```
<my-component>
    <my-title></my-title>
</my-component>
```
好了明白了这两个概念我们就可以使用`ViewChild`和`ContentChild`这两个选择器了，`ViewChild`获取视图子节点，`ContentChild`获取内容子节点
```javascript
@ViewChild(PanelComponent) panel:PanelComponent
ngAfterViewInit(): void {
    console.log('ngAfterViewInit', this.panel)	
}
```

```javascript
@ContentChild(PanelTitleComponent) panelTitle:PanelTitleComponent
ngAfterContentInit(): void {
    console.log('ngAfterViewInit','panelTitle', this.panel)
}
```

当然有时候我们可能想获取普通的H5标签，Angular为我们提供了一种简便的方式
```
<h1 #test></h1>
```

```javascript
export class TestComponent() {
    @ViewChild('test') h1Dom
}
```
这样我们就可以取到非自定义的标签元素了，当然这种方法可以用到自定义的标签上面。同样还有两个于此类似的装饰器`ViewChildren`和`ContentChildren`只不过会取得多个子节点。

## 生命周期
关于生命周期，参见另一篇博客[Angular4学习之生命周期](http://djl.pub/2017/08/17/Angular4%E5%AD%A6%E4%B9%A0%E4%B9%8B%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F/)
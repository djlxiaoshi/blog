---
title: Angular4学习之依赖注入
date: 2017-12-26 18:06:40
tags:
    - Angular4
    - 依赖注入
---

在一个项目中，组件和服务之间存在错综复杂的关系，为了最小程度的耦合，我们需要来管理组织这种关系，依赖注入就是管理这种关系的一种方式。
<!--more-->

## 为什么要使用依赖注入
在学习一个概念之前，我们必须要知道我们为什么要学习这个东西，这个东西究竟解决了什么问题。就好比这里讲到的，依赖注入究竟解决了什么问题。要解决这个问题，我们先来看看示例代码：

```javascript
export class Car {

  public engine: Engine;
  public tires: Tires;
  public description = 'No DI';

  constructor() {
    this.engine = new Engine();
    this.tires = new Tires();
  }

  // Method using the engine and tires
  drive() {
    return `${this.description} car with ` +
      `${this.engine.cylinders} cylinders and ${this.tires.make} tires.`;
  }
}
```

以上是来自angular官网的一段代码，我们可以看到一个`Car`类依赖于`Engine`和`Tires`这两个类，我们在`Car`的构造函数中去实例这两个依赖类。这有什么问题？如果有一天我们的`Tires`构造函数需要一个参数，那么我们必须要在`Car`的构造函数中去更改代码。

```javascript
// ...
constructor() {
   this.engine = new Engine();
   this.tires = new Tires(params);
 }
]
// ...
```

这种代码是非常不灵活的。虽然我们可以进行如下结构调整

```javascript
export class Car {

  public engine: Engine;
  public tires: Tires;
  public description = 'No DI';

  constructor(engine, tires) {
    this.engine = engine;
    this.tires = tires;
  }

  // Method using the engine and tires
  drive() {
    return `${this.description} car with ` +
      `${this.engine.cylinders} cylinders and ${this.tires.make} tires.`;
  }
}

const car = new Car(new Engine(), new Tires())
```

这样似乎解决了不灵活的问题，但是如果依赖项很多的话，我们都要去手动创建这些实例，也不太方便。其实创建依赖实例的过程完全可以交给一个专门的'工厂'来做，这就是angular里面的Injector。

## 基本使用

- 在组件中使用

```javascript
@Component({
  selector: 'app-heroes',
  providers: [Engine, Tires],
  template: `
    <h2>Heroes</h2>
    <app-hero-list></app-hero-list>
  `
})
export class HeroesComponent {
  construtor(private engine: Engine) {
    this.engine.start();
  }
}
```

在Angular中，一般我们将这些公共的依赖都会一些一个服务里面。在上面的用法我们可以看到多了一个providers，另外就是在类的构造函数中增加了`private engine: Engine`我们就可以去使用engine这个实例了，在这个过程中，我们并没有去手动去创建依赖项的实例。这是因为angular的Injector帮我们自动创建了。在这里有一个比较形象的比喻就是，一个厨子（Injector）根据菜谱（providers）去做菜（依赖的实例），但是究竟做哪些菜呢，客人说了算（`private engine: Engine`也就是构造函数中的）

- 在服务中使用

```javascript
import { Injectable } from '@angular/core';

@Injectable()
export class HeroService {
  constructor(private engine: Engine) { }
}
```

如果我们的一个服务本身就依赖于其他依赖项，那么我们使用`@Injectable()`装饰器（即使一个服务并没有依赖于其他服务，我们也推荐加上@Injectable()装饰器），我们依然要提供providers。这里由于服务通常跟视图是没有具体的关系，所以这里我们不会引入`@component`装饰器，那么我们在哪里确定这个providers呢?我们可以在一个`module`中的providers属性中去定义，那么这个`module`中的所有组件都会去共用这一个实例，但是我们有时候我们不希望共用一个实例，而是一个新的实例，那么我们可以在这个组件中的providers中重新定义，这样我们就会得到一个新的实例。实际上这就是层级注入。利用层级注入我们既可以共用实例，也可以不共用实例非常方便。一般全局使用的服务，我们会注册在app.module模块之下，这样在整个应用中都可以使用。

在上面我们说过通过依赖注入创建的实例是可以实现共享的，我们证明一下。

```javascript
import { Component, OnInit, ReflectiveInjector } from '@angular/core';
import {DependenceComponent} from './dependence.component';

@Component({
  selector: 'app-service',
  templateUrl: './service.component.html',
  styleUrls: ['./service.component.scss'],
})


@Injectable()
export class ServiceComponent implements OnInit {
  
  constructor() {
    let injector = ReflectiveInjector.resolveAndCreate([Dependence]);
    let dependence1 = injector.get(Dependence);
    let dependence2 = injector.get(Dependence);
    console.log('dependence1 === dependence2', dependence1 === dependence2); // true
  }
  
  ngOnInit() {}
}
```

在这里我们可以看见打印出来的是`true`，这里我们采用的是手动创建实例，所以我们并不需要在providers中提供“菜谱”，实际上`resolveAndCreate`的参数就是一个`providers`

## Providers

我们有四种配置注入过程，即使用类、使用工厂、使用值、使用别名

- 使用类

```javascript
{provide: MyService, useClass: MyService}
```

这是我们最常见的情形在angular中，通常如果provide的值和useclass的值一样，我们可以简化为`[MyService]`。

- 使用值
显然并不是每种情况，我们都需要注入一个类，有时候可以仅仅是一个值

```javascript
{provide: MyValue, useValue: 12345}
```

- 使用别名

```javascript
{provide: OldService, useClass: NewService}
```

如果我们有两个服务`OldService`和`NewService`接口都一致，出于某种原因，我们不得不使用`OldService`作为Token，但是我们又想使用`NewService`中的接口，那么我们就可以使用别名。

- 使用存在的值

```javascript
[ NewLogger,
  // Not aliased! Creates two instances of `NewLogger`
  { provide: OldLogger, useClass: NewLogger}]
```

这种情况下会创建两个NewLogger的实例，这显然不是我们想要的结果，这时我们就可以使用存在的

```javascript
[ NewLogger,
  // Alias OldLogger w/ reference to NewLogger
  { provide: OldLogger, useExisting: NewLogger}]
```
- 使用工厂
如果我们的服务需要根据不同的输入值，做出不同的响应，那么就必须要接受一个参数，那么我们就必须使用工厂

```javascript
{provide: MyService, useFactory: (user: User) => {
    user.isAdmin ? new adminService : customService,
    deps: [User]
}}
```

当使用工厂时，我们可以通过变量的不同值，去实例不同的类。也就是说我们需要根据不同的值返回不同的依赖实例的时候，那么我们就需要使用工厂。

## @Options 、@Host

目前为止我们的依赖都是存在的，但是实际情况并不是总是这样。那么我们可以通过@Optional装饰器来解决这个问题。

```javascript
import { Optional } from '@angular/core';
// ....
constructor(
    @Optional() private dependenceService: DependenceService
) {}
```

> 但是这里DependenceService这个服务类的定义还是存在的，只是没有准备好，例如没有在providers中使用

依赖查找的规则是按照注入器从当前组件向父级组件查找，直到找到这个依赖为止，但是如果限定查找路径截止在宿主组件，那么如果宿主组件中没有就会报错，我们可以通过@Host修饰器达到这一功能。

> 如果一个组件注入了依赖项，那么这个组件就是这个依赖项的宿主组件，但是如果这个组件通过`ng-content`被嵌入到宿主组件，那么这个宿主组件就是该依赖项的宿主组件。

## Token

当我们在构造函数中使用`private dependenceService: DependenceService`,injector就可以正确的知道我们要实例哪一个类，这是因为在这里`DependenceService`充当了Token的角色（也就是说类名是可以充当Token的），我们只需要在providers中去寻找具有相同Token的值就行，但是往往我们注入不是一个类，而是一个字符串，function或者对象。而这里string、方法名和对象是不能够充当Token的，那么这时我们就需要来手动创建一个Token:

```javascript
import { InjectionToken } from '@angular/core';

export const APP_CONFIG = new InjectionToken<AppConfig>('app.config');

providers: [{ provide: APP_CONFIG, useValue: HERO_DI_CONFIG }]
```

```javascript
constructor(@Inject(APP_CONFIG) config: AppConfig) {
  this.title = config.title;
}
```

Inject 装饰器显示的声明所依赖对象的类型

```javascript
@Injectable()
class A {
    constructor(private buffer: Buffer) {}
}
```

等同于

```javascript
class A {
    constructor(@Inject(Buffer) private buffer: Buffer) {}
}
```


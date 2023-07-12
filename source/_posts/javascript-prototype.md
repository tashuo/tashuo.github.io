---
title: 浅谈原型链
date: 2023-04-23 18:00:58
categories:
- typescript
- javascript
tags:
- prototype
- 原型链
---
### 上题
先上前菜看段代码，思考以下几个问题

```typescript
# test.ts
class Thing {
  born = () => {
    console.log("time to born");
  };

  die() {
    console.log("time to die");
  }
}

class Animal extends Thing {
  name: string;
  constructor(name = "dog") {
    super();
    this.name = name;
  }

  eat = () => {
    console.log("eat");
  };

  say() {
    console.log(`Hi, i'm ${this.name}`);
  }
}

const cat = new Animal("cat");
const dog = new Thing();


```

以下几个的值分别是什么

1. `Object.getOwnPropertyNames(cat)`
2. `Object.getOwnPropertyNames(dog)`
3. `Object.getOwnPropertyNames(Object.getPrototypeOf(cat))`
4. `Object.getOwnPropertyNames(Object.getPrototypeOf(dog))`
5. `Object.getOwnPropertyNames(Animal)`
6. `Object.getOwnPropertyNames(Thing)`
<!-- more -->
### 解题
#### 1. `Object.getOwnPropertyNames(cat)`
`cat`对象由`Animal`类实例化而来，继承Animal类(及其父类及父类的父类...)的所有属性，即**`[ 'born', 'eat', 'name' ]`**

#### 2. `Object.getOwnPropertyNames(dog)`
同理1，dog对象继承Thing，拥有的属性是**`[ 'born' ]`**

#### 3. `Object.getOwnPropertyNames(Object.getPrototypeOf(cat))`
`cat`对象的原型对象即`Animal.prototype`，**原型对象默认有一个constructor属性指向构造函数本身**，即`Animal.prototype.constructor = Animal`，再加上`Animal`类本身定义的say方法，所以结果是**`[ 'constructor', 'say' ]`**

#### 4. `Object.getOwnPropertyNames(Object.getPrototypeOf(dog))`
同理3，结果是**`[ 'constructor', 'die' ]`**

#### 5. `Object.getOwnPropertyNames(Animal)`
这个有点绕，在一切皆对象的`JavaScript`中，`Animal`类也是一个函数对象，实例化内置的`Function`对象而来，底层的定义类似于`let Animal = new Function('name', { ...Animal构造函数的方法体... })`，所以继承了Function的所有属性，即**`[ 'length', 'name', 'arguments', 'caller', 'prototype' ]`**(ES5, 新标准中Function的属性有所出入，此处暂不考虑)

#### 6. `Object.getOwnPropertyNames(Thing)`
同理5，结果一样是**`[ 'length', 'name', 'arguments', 'caller', 'prototype' ]`**


### 深入
上面出现了三类数据

1. 类/class，即`Animal`、`Thing`
2. 类的实例，即`cat`、`dog`
3. 原型对象，即`Object.getPrototypeOf(cat)`、`Object.getPrototypeOf(dog)`

这三者之间的关联说简单也简单，说难也难，对于从其他语言转过来的筒子来说概念上很容易绕晕，首先看第3个本文重点关注的原型对象

#### 原型对象
原型对象相关的概念有4个：`__proto__`、`prototype`、`Object.setPrototypeOf`和`Object.getPrototypeOf`，我们看下它们的作用及历史渊源：
> * 构造函数的 "prototype" 属性自古以来就起作用。这是使用给定原型创建对象的最古老的方式。
> 
> * 之后，在 2012 年，Object.create 出现在标准中。它提供了使用给定原型创建对象的能力。但没有提供 get/set 它的能力。一些浏览器实现了非标准的 \_\_proto__ 访问器，以为开发者提供更多的灵活性。
> 
> * 之后，在 2015 年，Object.setPrototypeOf 和 Object.getPrototypeOf 被加入到标准中，执行与 \_\_proto__ 相同的功能。由于 \_\_proto__ 实际上已经在所有地方都得到了实现，但它已过时，所以被加入到该标准的附件 B 中，即：在非浏览器环境下，它的支持是可选的。
> 
> * 之后，在 2022 年，官方允许在对象字面量 {...} 中使用 __proto__（从附录 B 中移出来了），但不能用作 getter/setter obj.\_\_proto__（仍在附录 B 中）。

*引用自[https://zh.javascript.info/prototype-methods](https://zh.javascript.info/prototype-methods)*

简单来讲，目前的标准有两种获取原型对象的方式，`prototype`和`Object.getPrototypeOf`，而且有使用限制：

* **`prototype`只能用于构造函数，如本文中的`Animal`、`Thing`**

这句话除了含有**非构造函数只能用`Object.getPrototypeOf`获取原型对象**的意义外，还隐藏着另外一个原则，即**构造函数也可以用`Object.getPrototypeOf`获取原型对象**，这含义真的是九曲十八弯，脑袋冒烟不打紧，我们继续看，根据`JavaScript`原型链的定义，可得出以下结论：

1. `Object.getPrototypeOf(cat)` = `Animal.prototype`
2. `Object.getPrototypeOf(dog)` = `Thing.prototype`
3. `Object.getPrototypeOf(Animal.prototype)` = `Thing.prototype`
4. `Object.getPrototypeOf(Thing.prototype)` = `Object.prototype`
5. `Object.getPrototypeOf(Object.prototype)` = `null`

这个就是经典的原型链，这也是`JavaScript` *一切都从对象继承而来* 的原因，上面刚说了*构造函数也可以用`Object.getPrototypeOf`获取原型对象*，那`Animal`和`Thing`的原型对象分别是啥？`Object.getPrototypeOf(Thing) = ?`

#### 类
`class`关键字非严谨层面可以看作一个语法糖，通过编译后的js关键代码可见一斑：

```javascript
# test.js
var Thing = /** @class */ (function () {
    function Thing() {
        this.born = function () {
            console.log("time to born");
        };
    }
    Thing.prototype.die = function () {
        console.log("time to die");
    };
    return Thing;
}());

```

class中定义的属性及`constructor`方法逻辑构成了同名的函数(即构造函数)，定义的方法被加在构造函数的原型链上(`prototype`)；

而实例化对象的关键字`new`底层的逻辑大致是以下三步：

1. 创建一个新对象
2. 执行构造函数，并将this指向当前的新对象
3. 将新对象的原型对象设置为构造函数的原型对象

第2步使实例化的对象继承了类的所有属性，可以解释`Object.getOwnPropertyNames(cat)`和`Object.getOwnPropertyNames(dog)`的值；

第3步原型对象的设置，使实例化的对象继承了原型链上的所有方法，可以解释`Object.getOwnPropertyNames(Object.getPrototypeOf(cat))`和`Object.getOwnPropertyNames(Object.getPrototypeOf(dog))`的值；

那上小节提及的`Animal`构造函数对象本身的原型链(`Object.getPrototypeOf(Animal)`)和`Thing`构造函数对象本身的原型链(`Object.getPrototypeOf(Thing)`)的值又是什么呢？

其实上面稍为提到过，构造函数对象本身是由内建的类型`Function`实例化而来，以此可推论出`Object.getPrototypeOf(Thing)` = `Function.prototype`，简单来讲这里的`Thing`和`Function`类比于前面的`dog`和`Thing`，这里的绕在于`JavaScript`的类型系统，`class Thing`表面是一个类，实际上是一个构造函数，而底层又是`Function`类的实例，`Thing`是别人(Function)的实例，它又可以再实例出别的实例(dog)，实例实例无穷尽也~ 其实站在`JavaScript`一切皆对象的角度看，没有那些眼花缭乱的类呀、实例呀、函数呀、字符串呀、整数呀、布尔值呀等等，对v8来讲都是`object`，等等，是不是还有漏了个`Animal`，`Object.getPrototypeOf(Animal)` = ？

这里又不得不提`extends`关键字了，和`class`类似，也是一个语法糖，咱直接看编译后的js关键代码：

```javascript
# test.js
var __extends = (this && this.__extends) || (function () {
    var extendStatics = function (d, b) {
        extendStatics = Object.setPrototypeOf ||
            ({ __proto__: [] } instanceof Array && function (d, b) { d.__proto__ = b; }) ||
            function (d, b) { for (var p in b) if (Object.prototype.hasOwnProperty.call(b, p)) d[p] = b[p]; };
        return extendStatics(d, b);
    };
    return function (d, b) {
        if (typeof b !== "function" && b !== null)
            throw new TypeError("Class extends value " + String(b) + " is not a constructor or null");
        extendStatics(d, b);
        function __() { this.constructor = d; }
        d.prototype = b === null ? Object.create(b) : (__.prototype = b.prototype, new __());
    };
})();

var Animal = /** @class */ (function (_super) {
    __extends(Animal, _super);
    function Animal(type) {
        if (type === void 0) { type = "dog"; }
        var _this = _super.call(this) || this;
        _this.eat = function () {
            console.log("eat");
        };
        _this.type = type;
        return _this;
    }
    Animal.prototype.say = function () {
        console.log("Hi, i'm ".concat(this.type));
    };
    return Animal;
}(Thing));

```
有两个关键点：

1. `extendStatics(d, b)`其实执行了`Object.setPrototypeOf(d, b)`，在本文中即`Object.setPrototypeOf(Animal, Thing)`，这一下子不就找到了答案？`Object.getPrototypeOf(Animal)` = `Thing`；

2. `d.prototype = b === null ? Object.create(b) : (__.prototype = b.prototype, new __());`
   这行代码大致可以拆分为以下几个流程：
   
   1. 覆盖\_\_的原型链: \_\_.prototype = b.prototype
    2. 使用__构造函数创建对象abc: let abc = { constructor: d; }
    3. 设置abc的原型对象: abc.\_\_proto__ = b.prototype;
    4. 设置d的原型对象: d.prototype = abc
    
    这个流程中，第一次看到整篇都在谈却一直没见踪影的的原型对象本尊，上面通过`prototype`和`Object.getPrototypeOf`获取到的就是abc这样的原型对象，结合我们的实例可得出：`Object.getPrototypeOf(cat)` = `Animal.prototype` = `abc`，`Object.getPrototypeOf(Animal.prototype)` = `abc.__proto__` = `Thing.prototype`，跟我们上面谈到的也对上了号

*最后祭出这张神图！！*
![VgXXkZ.png](https://i.imgloc.com/2023/07/12/VgXXkZ.png)
    
### 结尾
完结撒花🎉，原型链在日常开发中直接碰到的不多，但每次遇到相关的问题都会很难绕出来，所以大致做了一下总结，有纰漏及错误之处还请勇猛指出

本文完整示例代码：[https://github.com/tashuo/note/tree/master/typescript/prototype](https://github.com/tashuo/note/tree/master/typescript/prototype)

### 参考
[https://zh.javascript.info/prototypes](https://zh.javascript.info/prototypes)

[https://juejin.cn/post/6844903989088092174](https://juejin.cn/post/6844903989088092174)
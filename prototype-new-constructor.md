# 追溯原型
---------

我个人认为 `JavaScript` 首先是一门函数式语言,
函数天然就是一等公民来可以拿来随意传递而丝毫无须在意什么是 `Lambda`,
闭包的创建和捕获轻而易举(需要避免一些小陷阱).

其次它完全谈不上面向对象(Object-Oriented-Programming).
虽然万年保留的 `class` 关键词终于在 `ES2015` 中派上了用场,
却也只是 `prototype` 的语法糖.
依赖原型来实例化新对象,
所以 `JavaScript` 又是一门基于对象的语言(Object-Based-Programming).

接下来就来一探 `JavaScript` 的原型,顺着原型链追本溯源.

# 词汇表
- 原型(prototype)
- 原型链(prototype chain)
- 实例化(new)
- 构造器(constructor)

## 什么是原型?
在四人组 GOF 的著作《设计模式 —— 可复用面向对象软件的基础》中, 有一种对象创建型模式叫 "原型模式", 其概念十分简单.

在传统的 OOP 开发中, 最简单的创建对象的方式莫过于直接通过 `new` 一个 `class` 来获得类的实例.

考虑下面这个例子, 有一个 `TodoItem` 类:
```cpp
class TodoItem {
    constructor() {
        this.title = null;
        this.date = null;
    }
}

// 为了得到若干个 Todo 条目, 需要执行如下操作:
var item1 = new TodoItem();
item1.title = 'item';
item1.date = new Date();

var item2 = new TodoItem();
item2.title = 'item';
item2.date = new Date();
// ...
```
如果对象的字段更多, 赋值更加复杂, 那么直接通过类来实例化对象将变得更加繁杂.

接下来通过原型模式改造 `TodoItem` 类:
```cpp
class TodoItem {
    constructor() {
        this.title = null;
        this.date = null;
    }
    clone() {
        var _instance = new TodoItem();
        _instance.title = this.title;
        _instance.date = this.date;
        return _instance;
    }
}

// 仅需要创建一个原型, 便可通过原型复制得到相同的对象:
var proto = new Item();
proto.title = 'item';
proto.date = new Date();

var item1 = proto.clone();
var item2 = proto.clone();
// ...
```
原型模式就好比是抄作业, 学霸写完作业之后, 全班人以学霸的作业为原型, 原模原样地获得复制品.

`JavaScript` 中不存在类的概念, 于是在 `JavaScript` 中实例化对象，
`JavaScript` 引擎都是通过复制已有的原型从而得到新的对象.

## 什么是原型链?
*注:* 以下 `JavaScript` 代码均为模拟代码, 并不能被执行, 仅用来模拟原理.
```javascript
// Part I, 模拟原生 JavaScript 的对象创建过程
// =======================================
// 万物始于 Object，而 Object 基于它的原型所创建
// 所以最先创建的不是 Object, 而是它的原型
Object.prototype = {
  __proto__: null
};


// 接下来以 Object 的原型为基础，创建 Function 的原型
Function.prototype = {
  __proto__: Object.prototype
};


// 以 Function 的原型为基础，创建 Function 构造器
function Function() { }
Function.__proto__ = Function.prototype;
// Function 的原型通过 constructor 指针指回构造器
Function.prototype.constructor = Function;


// 以 Function 的原型为基础，创建 Object 构造器
function Object() { }
Object.__proto__ = Function.prototype;
// Object 的原型通过 constructor 指针指回构造器
Object.prototype.constructor = Object;


// Part II, 利用上述的构造器在实际中的应用
// ===================================
// 以 Function 的原型为基础，创建自定义的 Foo 构造器
function Foo() { }
Foo.__proto__ = Function.prototype;
// 自定义构造器 Foo 被创建时，它的原型也同时被创建
// 以 Object 的原型为基础，创建 Foo 的原型
// Foo 的原型通过 constructor 指针指回构造器
Foo.prototype = {
  constructor: Foo,
  __proto__: Object.prototype
};

// 以 Foo 的原型为基础，通过 new 操作符实例化新对象
var bar = new Foo();
bar.__proto__ = Foo.prototype;

// 以 Object 的原型为基础，通过 new 操作符实例化新对象
var baz = new Object();
baz.__proto__ = Object.prototype;
```

## 当 `new` 一个构造器时背后发生了什么?

## 如何利用原型模拟面向对象编程?

[未完待续]
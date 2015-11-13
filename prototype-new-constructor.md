# 追溯原型
---------

`JavaScript` 不具备面向对象（Object-Oriented-Programming）编程的能力。
虽然万年保留的 `class` 关键词终于在 `ES2015` 中派上了用场，却也只是 `prototype` 的语法糖。

在面向对象语言中，通常有一个名为 `Object` 的根类，并且管你愿意还是不愿意，你定义的所有类都会自动继承此类。
基类与派生类之间通过继承建立纽带，这个纽带从你定义的类开始一层层往上追溯，最终回归到 `Object` 身上。

然而在 `JavaScript` 的世界观中，确是另一番景象。
内存管理对于浏览器来说可谓是寸土寸金，倘若通过类来实例化一个个独立的对象，
那么每个对象便占用一份资源，对象越多，资源占用也就越多。
由此而言，让对象互相共享一部分成员（函数成员和对象成员），可以有效地减少资源浪费。

抛开资源这个话题不议，从设计的角度来看，
`JavaScript` 从诞生之初的设计理念就是简单、容易上手、再拼抄一些别的语言。
*Brendan Eich* 当初花了十天设计出了 `JavaScript` 便敷衍 `Netscape` 了事，大神当初的心路历程估计是这样的：<br>
摆明了想借 `Java` 上位，抄完名字顺便把语法也抄了吧；<br>
搞 `C` 的那群家伙惹不起，一块儿抄了，反正跟 `Java` 语法也差不多；<br>
搞什么 `OOP`，`Functional` 才是宇宙的终极信仰。<br>
要有闭包！要有一等函数！不要副作用……额……最后这个还是算了，这群愚民肯定学不会 -_-|||<br>
搞什么类类类，吹完封装吹继承，多态够你们吹一辈子，嘿嘿这个我~不~加~<br>
`self` 的原型继承不错，抄抄抄！

十年后……

接下来就来一探 `JavaScript` 的原型，顺着原型链追本溯源。
我是勤劳的搬运工，再混杂一些个人的愚见，如有不正之处………你能拿我怎么样 o(╯□╰)o

> 我追寻着真相，找到了，但我却不喜欢这个真相，真希望我能忘了它。<br>
> —— 虐杀原型

# 词汇表
- 构造器(constructor)
- 实例化(new)
- 原型(prototype)
- 原型链(prototype chain)

## 什么是构造器？
不要诧异，构造器是一个函数，用于生成实例。
如果一个函数打算被设计为构造器，就请不要使用 `return` 语句。
```javascript
// TodoList 构造器
function TodoList() {
  // 为实例添加独有属性
  this.title = 'an awesome todo list';
  this.storage = [];

  // 为实例添加独有方法。把方法挂载到原型上更好，稍后介绍原型。
  this.addTodo = function(something) {
    this.storage.push(something);
  };
}

// TodoList 实例
var myTodo = new TodoList();
console.log(myTodo.title);
myTodo.addTodo('say javascript is awesome 100 times every day!');
```

## `new` 一个构造器的幕后发生了什么？
`new` 操作符用于生成对象的实例。这里的对象指以下两种：
1. 自定义的 `function`
2. 内置的带构造器的对象

示例
```javascript
function Foo() {
  this.prop = 'prop in Foo\'s instance';
}

var foo = new Foo();
var bar = new Date();
console.log(foo.prop); // "prop in Foo's instance"
console.log(bar.getFullYear()); // current year
```
当我们执行 `new` 操作时，共完成了如下几件事情。
1. 创建一个临时的空对象 o
2. 将 o 的原型指向构造器的原型对象
3. 以 o 为上下文执行构造器
4. 返回 o

用代码模拟以上过程
```javascript
function Foo() {
  this.prop = 'prop in Foo\'s instance';
}

// 用 createObject 函数模拟 new 的工作过程
function createObject(Ctor) {
  var o = {
     __proto__: Ctor.prototype // 此部分开篇另讲...
  };
  Ctor.apply(o);
  return o;
}

// 测试
var newObject = new Foo();
var createdObject = createObject(Foo);

console.log(newObject.prop); // "prop in Foo's instance"
console.log(createdObject.prop); // "prop in Foo's instance"
```

## 什么是原型？
在四人组 GOF 的著作《设计模式 —— 可复用面向对象软件的基础》中，有一种对象创建型模式叫 "原型模式"，其概念十分简单。

在传统的 OOP 开发中，最简单的创建对象的方式莫过于直接通过 `new` 一个 `class` 来获得类的实例。

考虑下面这个例子，有一个 `TodoItem` 类：
```javascript
class TodoItem {
    constructor() {
        this.title = null;
        this.date = null;
    }
}

// 为了得到若干个 Todo 条目，需要执行如下操作：
var item1 = new TodoItem();
item1.title = 'item';
item1.date = new Date();

var item2 = new TodoItem();
item2.title = 'item';
item2.date = new Date();
// ...
```
如果对象的字段更多，赋值更加复杂，那么直接通过类来实例化对象将变得更加繁杂。

接下来通过原型模式改造 `TodoItem` 类：
```javascript
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

// 仅需要创建一个原型，便可通过原型复制得到相同的对象：
var proto = new Item();
proto.title = 'item';
proto.date = new Date();

var item1 = proto.clone();
var item2 = proto.clone();
// ...
```
原型模式就好比是抄作业，学霸写完作业之后，全班人以学霸的作业为原型，原模原样地获得复制品。

## 什么是原型链？
*注:* 以下 `JavaScript` 代码均为模拟代码，并不能被执行，仅用来模拟原理。
```javascript
// Part I，模拟原生 JavaScript 的对象创建过程
// =======================================
// 万物始于 Object，而 Object 基于它的原型所创建
// 所以最先创建的不是 Object，而是它的原型
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


// Part II，利用上述的构造器在实际中的应用
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

## 如何利用原型模拟面向对象编程？
```javascript
// 定义基类
function BaseClass(baseProp) {
  this.baseProp = baseProp;
}
BaseClass.prototype.baseMethod = function() {
  console.log('base method being called');
};

// 定义子类
function DerivedClass(baseProp, derivedProp) {
  BaseClass.call(baseProp);                         // 为何这样？（1
  this.derivedProp = derivedProp;
}
DerivedClass.prototype = new BaseClass();           // 为何这样？（2
DerivedClass.prototype.constructor = DerivedClass;  // 为何这样？（3
DerivedClass.prototype.derivedMethod = function() {
  console.log('derived method being called');
};
// （1 需要动态作用域的知识来理解，在我的另一篇关于 `scope` 的文章中有介绍
// （2 需要了解 `JavaScript` 是如何追溯原型链的，下一节介绍
// （3 是因为派生类的原型指向 BaseClass 的一个实例，
//     因此 `DerivedClass.prototype.constructor === DerivedClass`
//     所以需要调整为 `DerivedClass` 自己，以示标准。

// 创建实例
var foo = new BaseClass('base');
var bar = new DerivedClass('base', 'derived');

console.log(foo.baseProp, bar.baseProp);
foo.baseMethod();
bar.baseMethod();
bar.derivedMethod();
```

## `JavaScript` 如何通过原型链寻找原型方法的？
以上面的代码为例：
```javascript
// 示例一：
foo.baseMethod();
// 因为 foo 对象本身并没有 baseMethod 这个属性
// 便通过 foo.__proto__ 也就是构造器 Foo 的原型上去找
// 成功定位到 foo.__proto__.baseMethod，设置 `this` 指向的上下文，执行方法


// 示例二：
bar.baseMethod();
// 因为 bar 对象本身并没有 baseMethod 这个属性
// 便通过 bar.__proto__ 也就是构造器 Bar 的原型上去找，没找到
// 继续通过 bar.__proto__.__proto__ 也就是 Foo 的原型上去找，找到了
// 成功定位到 foo.__proto__.__proto__.baseMethod，设置 `this` 指向的上下文，执行方法

// 实例三：
bar.toString();
// 这个要一直回溯到 Object.prototype.toString 为止，分析过程和上述一致
// bar -> DerivedClass.prototype -> BaseClass.prototype -> Function.prototype -> Object.prototype
```

### 参考
- [JavaScript Object Layout](www.mollypages.org/misc/js.mp)
- [JavaScript 原型链和原型的一个误区](http://www.laruence.com/2010/05/13/1462.html)

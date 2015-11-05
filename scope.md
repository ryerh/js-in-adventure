# 逃不出的作用域
----------------

一个变量的生命由始而终，
{ 始终逃不出它的**作用域** }。
`JavaScript` 的**作用域**在初学阶段显得十分棘手，
尤其对有别的编程语言经验的人甚然，
他们很容易被别的编程经验带入，
下意识地期望在 `JavaScript` 中得到与别的语言一致的**作用域**体验，
然而结果却常常出乎意料。

`JavaScript` 从它被潦草地设计出来伊始，便踏上了一条不归路。
虽然新的 `ECMAScript` 标准正在快速演进，
但放眼当下，着眼于遍布市场的各式浏览器，
它们对新标准支持参差不齐。
我们仍旧需要解决**作用域**这个历史遗留问题。

只需要掌握一些简单的规则，
便可正确地控制**作用域**在 `JavaScript` 中的行为表现。

> 仅凭人们喜欢吃汉堡并不能断定他们就想看见牛。<br>
> 当你真正喜欢一样东西，日思夜盼……不理会你的那个人就是上帝。<br>
> —— 逃出克隆岛

## 词汇表
- `作用域`
- `命名空间`
- `函数作用域`
- `闭包、词法作用域`
- `this、动态作用域`
- `全局、公共作用域`
- `局部、私有作用域`

## 1、全局作用域
**在代码顶层声明的变量都是全局变量**
```javascript
// 在顶层声明的变量
var g_value = 'hello, kitty';
// 从这里开始，任何位置都可以获取和修改 g_value
```

```javascript
// 在 if 语句中修改 g_value
if(true) {
  g_value.replace('kitty', 'world');
}
console.log(g_value); // OK!
```

```javascript
// 在 function 中修改 g_value
!function() {
  g_value = 'where is kitty?';
}();
console.log(g_value); // OK!
```

**无论何处，只要没有使用 `var` 关键词声明变量，
都将得到全局变量，你没有任何理由拒绝使用 `var` ！**
```javascript
// 在函数中没有用 var 关键字声明变量，并立即执行这个函数
!function() {
  none_var_value = 'this is dangerous!!!';
}();
console.log(none_var_value); // Buggy, but OK!
```
## 2、局部作用域
**`for` 和 `if` 等语法块无法隔离作用域**
```javascript
  // 代码开始，全局作用域开始
  for(int i = 0; i < 10; i++) {
    // 仍然处于全局作用域
    console.log(i);
  }
  console.log(i); // OK! -> 10

  if(true) {
    // 依旧处于全局作用域
    var inner_if_value = 'I\'m in `if` statement';
  }
  console.log(inner_if_value); // OK!

  {
    // 啊哈，还是全局作用域
    var i_am = 'crazy';
  }
  console.log(i_am); // OK!
  // 代码结束，全局作用域结束
```

**只有 `function` 内部可以隔离函数作用域**
```javascript
// 代码开始，全局作用域开始
!function() {
  // 局部作用域开始
  var l_value = 'I\'m from local';
  console.log(l_value); // OK!
  // 局部作用域结束
}();
console.log(l_value); // Referrence Error!
// 代码结束，全局作用域结束
```

**示例一**
```javascript
// 在 `jQuery` 项目代码中，我们常看到类似的代码：
$(function() {
  // 变量缓存
  var $app = $('#app');
  // ...

  // 逻辑开始
  // ...
});
// 便是为了利用匿名函数隔离内部所有变量的作用域
```
**示例二**
```javascript
// 我们还经常看到一些自调用匿名函数(SIAF)的写法：
!function() {
  // ...
}();

// 等价写法有很多种，比如下面这种：
(function() {
  // ...
})();
// 也是为了利用匿名函数隔离内部所有变量的作用域
```

## 3、函数作用域
**函数的定义可以嵌套，内部作用域如下：**
```javascript
// 代码开始，全局作用域开始
function AAA() {
  // 函数 AAA 作用域开始
  function BBB() {
    // 函数 BBB 作用域开始
    function CCC() {
      // 函数 CCC 作用域开始
      // ...
      // 函数 CCC 作用域结束
    }
    // 函数 BBB 作用域结束
  }
  // 函数 AAA 作用域结束
}
// 代码结束，全局作用域结束
```

## 4、词法作用域
**一个变量在他被声明的作用域链中都是可见的，其值由最近一层的定义所决定**
```javascript
var outer_value = 'outside';

!function() {
  var outer_value = 'inside';
  console.log(outer_value); // OK! -> inside

  !function() {
    var outer_value = 'inner';
    console.log(outer_value); // OK! -> inner

    outer_value = 'modified';
  }();
  console.log(outer_value); // OK! -> inside

}();
console.log(outer_value); // OK! -> outside
```

## 5、闭包
**闭包与词法作用域的关系十分紧密**
```javascript
function create_factory(device) {
  // 1. 此处声明 product 变量
  var product = 'I created a(n) ' + device;

  // 4. 返回值是一个函数，而且该函数需要依赖 product 才能执行
  return function(model) {
    // 5. 所以 product 变量的生命会延长
    // 6. 具体延长到什么时候？你猜:)
    return product + model;
  };

  // 2. 此时 create_factory 执行完毕
  // 3. 局部变量 product 理应被销毁
}

// 执行两次 create_factory 函数，各得到一个函数作为返回值
var apple_creator = create_factory('iPhone');
var sangsung_creator = create_factory('Galaxy');

// 试想如果每次执行完毕
// product 变量都被销毁了
// 那么接下来这两行代码会得到什么结果
// 应该是 Refference Error 吧？

// 那么我们来执行以下试试看：
console.log(apple_creator('4s')); // OK! -> I created a(n) iPhone4s!
console.log(sangsung_creator('S6')); // OK! -> I created a(n) GalaxyS6!
// 事实证明，身为局部变量的 product 的确没有被销毁

// 当内层函数引用了外层的变量，就被称为**闭包**，被引用的变量就叫**闭包变量**
// **闭包变量**不会被立即销毁，所以不要滥用闭包
// ==============================
```

## 6、变量提升(Hoisting)
**变量提升与词法作用域的关系也十分紧密，
只要一个变量被声明，
那么在它所处的整个词法作用域中都是可见的**

```javascript
// 直接访问一个从未声明的变量
console.log(none_defined); // Referrence Error!
```

```javascript
// 先声明后访问
var defined_value = 'hello, kitty!';
console.log(defined_value); // OK!
```

```javascript
// 先访问后声明
console.log(will_define); // undefined!
// ...
var will_define = 'hello, kitty!';
```

```javascript
// 先访问后声明变形一
!function() {
  console.log(will_define); // undefined!
}();
var will_define = 'hello, kitty!';
```

```javascript
// 在当前作用域声明 defined_value
var defined_value = 'hello, kitty!';

!function() {
  // 当前词法作用域找到了两个 defined_value
  // 一个来上一层，另一个在本层稍后面几行
  // 以就近原则为准，因此 defined_value 的值由本层级的定义决定
  // 但是 defined_value 的赋值，还要等到后面才被执行
  // 所以此时，defined_value 相当于已被声明，但没有赋值
  // 所以直接输出 defined_value 得到的是 undefined 而不是 Referrence Error!
  console.log(defined_value); // undefined!

  var defined_value = 'hello, world!';
  // 赋值完毕，可以正常访问 defined_value 的值了
  console.log(defined_value); // OK! -> hello, world!
}();

// 函数执行结束，退回到全局作用域，defined_value 的值并没有变化
console.log(defined_value); // OK! ->  hello, kitty!
```

**上面的一切是因为 `JavaScript` 在定义它们的作用域里运行，
而不是在执行它们的作用域里运行。**

## [未完待续]

### 参考
- [Everything you wanted to know about JavaScript scope](http://toddmotto.com/everything-you-wanted-to-know-about-javascript-scope/)
- 《JavaScript 函数式编程》

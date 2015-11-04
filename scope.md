# 逃不出的作用域

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
- `闭包`
- `this`
- `命名空间`
- `函数作用域`
- `词法作用域`
- `动态作用域`
- `全局、公共作用域`
- `局部、私有作用域`

## 1、全局作用域
**在代码顶层声明的变量都是全局变量。**
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
// 在 function 中修改 g_varialbe
!function() {
  g_variable = 'where is kitty?';
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
## 局部作用域
**`for` 和 `if` 等语法块无法隔离作用域**
```javascript
  // 代码开始，全局作用域开始
  for(int i = 0; i < 10; i++) {
    // 仍然处于全局作用域
    console.log(i);
  }
  console.log(i); // OK! -> 10
```

**只有 `function` 内部可以隔离函数作用域**
```javascript
// 代码开始，全局作用域开始
!function() {
  // 局部作用域开始
  var l_value = 'I\'m from local';

  // 局部作用域结束
}();
// 代码结束，全局作用域结束
```
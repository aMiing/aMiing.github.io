# let x = x; 会报错吗？为什么？-- 暂时性死区
---
theme: channing-cyan
---

## 前言
最近在重新学习ES6,很多细节让我有种从来没学过js的错觉。

> 涉及知识点：**变量提升、暂时性死区**。 如果对这两个知识点已经非常熟悉的同学可直接跳过。
## 题目分析
其实就是把x赋值给x，我们知道赋值操作是从左到右的，所以显然执行赋值之前X自身也未必赋值过。

可能有点绕，话句话理解就是x未定义，x去给另一个变量赋值。（类似于 let y = x;）


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3daae7769679445aa60f19cccef6643b~tplv-k3u1fbpfcp-watermark.image?)

执行报错了！

如果我们使用var声明并赋值呢？

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c81ff2e1e9ec41f39e1d729830534201~tplv-k3u1fbpfcp-watermark.image?)

用var声明就不会报错，是什么原因呢？

> 这里就是我们常常听说的所谓的**变量提升**，var属于es6之前的语法，存在变量提升的问题，es6推出let,const其中一个原因就是为了避免令人费解的变量提升问题。

当我们用`var`声明变量的时候，在该语句的 **作用域** 中会出现变量提升，默认声明会提升到执行上下文的最上面。


```js
() => {
    console.log(a);
    var a = 123;
}
// 相当于
() => {
    var a;
    console.log(a);
    a = 123;
}
```
所以上面提到的问题，var a = a;由于变量提升的存在，a在赋值的时候已经存在了，只是为`undefined`。


## 暂时性死区
下面来重点说道说道暂时性死区。
> ES6 明确规定，如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。
简单理解就是let不存在变量提升，声明之前的区间被锁定了，取不到在其后声明的变量。
这里还有一层含义，是被锁定。下面是一个典型的例子：

```js
var temp = 123;
(function() {
    console.log(temp); //Uncaught ReferenceError
    let temp = 456;
})();
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c23bbb4fe91404d9c1dc5aa0220ac57~tplv-k3u1fbpfcp-watermark.image?)
即使我们在上层作用域声明了temp变量，那么由于内部作用域中存在let声明，所以形成了暂时性死区，打印temp报错。

还有些“死区”比较隐蔽，不太容易发现。

```js
function bar(x = y, y = 2) {
  return [x, y];
}

bar(); // 报错
```
先把y赋值给x,此时y还没声明。

“暂时性死区”也意味着`typeof`不再是一个百分之百安全的操作。

```
typeof x; // ReferenceError
let x;
```

上面代码中，变量`x`使用`let`命令声明，所以在声明之前，都属于`x`的“死区”，只要用到该变量就会报错。因此，`typeof`运行时就会抛出一个`ReferenceError`。

作为比较，如果一个变量根本没有被声明，使用`typeof`反而不会报错。


## 总结
暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。

let 和 const 的出现在很大程度上就是为了弥补var的全局性、变量提升的问题。


[返回首页](/)
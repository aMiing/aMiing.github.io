# 你说的this是哪个this?

## 前言

我们都知道js里的this指向一直都是个令人头大的问题。在不同的环境，甚至是不同的语义、作用域里面this的指向千差万别。让我们花几分钟时间深入理解一下this的指向问题，彻底搞懂，再学最后一次。

## 全局作用域的this
全局执行上下文中的 this 是指向 **window** 对象的。这也是 this 和作用域链的唯一交点，作用域链的最底端包含了 window 对象，全局执行上下文中的 this 也是指向 window 对象。
## 函数执行上下文中的this


```
function foo(){
  console.log(this)
}
foo()
```

在 foo 函数内部打印出来 this 值，执行这段代码，打印出来的也是 window 对象，这说明在默认情况下调用一个函数，其**执行上下文中的 this 也是指向 window 对象的**。

那么如何修改this的指向呢？
### 1. `bind、apply、call`

三者第一个参数都是作为函数的新`this`传入。区别：bind返回的是函数，不会立即执行；apply 第二个参数是数组。bind和call的参数都是散列传递的，即从第二个往后都是传递给函数的参数，用逗号隔开。
例如：

```js
function origin(name, age) {
    console.log(this);
    console.log(name, age);
}
const people1 = {
    name: '冰墩墩',
    color: 'white',
}
origin();
origin.bind(people1, 'amingxiansen', 26)();
origin.call(people1, 'amingxiansen', 26);
origin.apply(people1, ['amingxiansen', 26]);
```
执行结果：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0fbf96f857c419c86092ff80813cb29~tplv-k3u1fbpfcp-watermark.image?)

### 2. 通过对象调用方法设置


```js
const people = {
    name: 'amingxian',
    getName: function(){
        console.log(this, this.name)
    }
}
people.getName();
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d937959b4464db2956c93b5899bd107~tplv-k3u1fbpfcp-watermark.image?)

如果我们把对象调用再赋值给全局变量，this又会发生怎样的变化呢？

```js
const people = {
    name: 'amingxian',
    getName: function(){
        console.log(this, this.name)
    }
}
const otherPeople = people.getName;
otherPeople();
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59fc213b5df64554a47a6a8bb8198fa7~tplv-k3u1fbpfcp-watermark.image?)

此时的this又指向了全局`window`对象。

**小结**
>所以通过以上两个例子的对比，你可以得出下面这样两个结论：
> - 在全局环境中调用一个函数，函数内部的 this 指向的是全局变量 window。
> - 通过一个对象来调用其内部的一个方法，该方法的执行上下文中的 this 指向对象本身。

### 3. 构造函数修改this指向


```js
function MyObj() {
    this.name="amingxiansen";
}
const obj = new MyObj();
```

这里的this指向新的实例本身。

## this的设计缺陷
当对象嵌套的场景下，this的指向在不同的层级会发生令人疑惑的改变。
比如：

```js
const people = {
    name: 'amingxian',
    getName: function(){
        console.log('outerThis', this, this.name)
        function test() {
            console.log('innerThis', this)
        }
        test();
    }
}
people.getName();
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39491322c32a45d0b00328560ed71e3d~tplv-k3u1fbpfcp-watermark.image?)

内层function并不能继承getName作用域的this。

解决思路有两个：
1. 其一是在test函数外部将this赋值给一个变量，在test内部使用这个变量；
2. 其二是使用ES6的箭头函数。可以继承外部作用域的this。

ES6提供了`globalThis`对象来解决获取顶层作用域的问题。
## globalThis 对象

JavaScript 语言存在一个顶层对象，它提供全局环境（即全局作用域），所有代码都是在这个环境中运行。但是，顶层对象在各种实现里面是不统一的。

-   浏览器里面，顶层对象是`window`，但 Node 和 Web Worker 没有`window`。
-   浏览器和 Web Worker 里面，`self`也指向顶层对象，但是 Node 没有`self`。
-   Node 里面，顶层对象是`global`，但其他环境都不支持。

同一段代码为了能够在各种环境，都能取到顶层对象，现在一般是使用`this`关键字，但是有局限性。

-   全局环境中，`this`会返回顶层对象。但是，Node.js 模块中`this`返回的是当前模块，ES6 模块中`this`返回的是`undefined`。
-   函数里面的`this`，如果函数不是作为对象的方法运行，而是单纯作为函数运行，`this`会指向顶层对象。但是，严格模式下，这时`this`会返回`undefined`。
-   不管是严格模式，还是普通模式，`new Function('return this')()`，总是会返回全局对象。但是，如果浏览器用了 CSP（Content Security Policy，内容安全策略），那么`eval`、`new Function`这些方法都可能无法使用。

综上所述，很难找到一种方法，可以在所有情况下，都取到顶层对象。下面是两种勉强可以使用的方法。

```
// 方法一
(typeof window !== 'undefined'
   ? window
   : (typeof process === 'object' &&
      typeof require === 'function' &&
      typeof global === 'object')
     ? global
     : this);

// 方法二
var getGlobal = function () {
  if (typeof self !== 'undefined') { return self; }
  if (typeof window !== 'undefined') { return window; }
  if (typeof global !== 'undefined') { return global; }
  throw new Error('unable to locate global object');
};
```

[ES2020](https://github.com/tc39/proposal-global) 在语言标准的层面，引入`globalThis`作为顶层对象。也就是说，任何环境下，`globalThis`都是存在的，都可以从它拿到顶层对象，指向全局环境下的`this`。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef8ac885b6554fe78cb52a42d8848b00~tplv-k3u1fbpfcp-watermark.image?)

垫片库[`global-this`](https://github.com/ungap/global-this)模拟了这个提案，可以在所有环境拿到`globalThis`。

## 总结

js设计之初存在的问题，会逐渐在后面的es标准中逐渐完善和修复，我们在享受新语法带来的快感的同时，也应该尽其所能为这门语言的发展出谋划策、贡献力量。[ECMA ts39](https://github.com/orgs/tc39/repositories)
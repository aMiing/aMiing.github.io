# 看了辣么多人讲解原型与原型链，你真的搞明白了吗？

## 前言
关于原型与原型链，可能准备过面试的同学都能够说出个一二，但有点时候还是模棱两可，总是喜欢用 **可能， 大概， 也许** 这样不确定的词，让面试官觉得你的基础不勾选扎实。

究其原因，还是因为没有真正对这块知识掌握的不够牢固，没有彻底搞明白`prototype、 __proto__、 constructor`三者之间的关系。笔者也存在这样的问题，所以想通过这样的方式来梳理一下三者的关系，这次彻底搞懂原型和原型链。

## 基础概念

`ECMAscript`将原型链作为实现继承的主要方法。基本原理是利用原型链实现一个引用数据类型继承另一个引用类型的属性和方法。
示例代码：
```js
function FOO(){};
const f = new FOO();
```
【构造函数】用来初始化新对象的函数叫构造函数。 示例中的FOO()函数就是构造函数。
【实例对象】通过 new 关键字创建的对象叫实例对象。示例中的f。
【原型对象及prototype】构造函数有一个prototype属性，它指向实例对象的原型。常使用原型对象来实现继承。
【constructor】原型对象有constructor属性，指向原型对象的构造函数。
![原型与构造函数的关系](/source/images/constructor.png)
【__proto__】__proto__ 属性像是指针，指向构造函数的原型对象。
![三角关系](/source/images/proto.png)

其实我们这里只是讲了一个通过`new`关键字生成实例对象的特例。

具体prototype的指向和生成实例的方式有关，下面就具体讲下。

> 不同浏览器对原型链的实现会有细微的差别。本篇文章均以chrome浏览器的实现为基础。
## 构造实例对象的方式决定 `__proto__` 的指向

1. 通过构造函数和`new`构建
```js
function FOO(){};
const f = new FOO();
```
由构造函数创建的对象，其`__proto__`指向其构造函数的prototype属性指向的对象。
```js
FOO.prototype === f.__proto__
// true
```

2. 对象字面量构造的对象

```js
const person1 = {
    name: 'amingxiansen',
    sex: 'male'
};
```
其`__proto__`指向Object.prototype。

形同于：
```js
const person1 = new Object({
    name: 'amingxiansen',
    sex: 'male'
});
```
结果：
```js
person.__proto__ === Object.prototype
// true
```
所以严格来讲，1 和 2 可以算作是同一种情况。

3. 通过`Object.create()`创建

```js
const person1 = {
    name: 'amingxiansen',
    sex: 'male'
}

const person2 = Object.create(person1);
```
这里person2的 `__proto__` 指向person1。

```js
person2.__proto__ === person1
// true
```

在没有Object.create函数的日子里，人们是这样做的：
```js
Object.create = function(p) {
    function f(){}
    f.prototype = p;
    return new f();
}
```
这样就能保证新构建的对象的`__proto__`指向生成它的对象了。

## 图解

之前一直搞不清`prototype`和`__proto__`的区别，其实可以简单理解为`prototype`是原型，`__proto__`是实例指向构建函数的原型的指针。

贴上一位掘友提供的图示
![prototype](/source/images/prototype-1.awebp)

## 总结
其实理解其原理之后也没有那么难！

下回不怕面试官再问原型和原型链了！

参考资源：
1. https://juejin.cn/post/6934498361475072014#heading-6
2. https://www.zhihu.com/question/34183746/answer/58068402




[返回首页](/)
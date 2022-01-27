# 立即（自）执行函数

> 就像你看到的這樣，自執行函数，顾名思义就是会自己执行，不需要调动触发。

## 正确的自执行函数应该怎么写呢？
通常情况下，我们会这样声明一个函数，然后用他的名称加上圆括号来调用它
```js
const funcA = function(){/* do somethings */}
funcA()

```
自执行函数当然是省略掉调用的过程了

```js
const funcA = function(){console.log('hello')}() //hello
```
执行成功！

> 有些情况下没有返回值，或者我们不关心返回值，那这个赋值操作就显得比较多余了，我们把它去掉试试

### 方式一：

```js
function(){console.log('hello')}() // Uncaught SyntaxError: Function statements require a function name
```
报错了，需要一个functionName!!!

### 方式二：（错的）

```js
function foo(){console.log('hello')}() // Uncaught SyntaxError: Unexpected token ')'
```

+ 这种写法也是不行的，等价于：

```js
function foo(){console.log('hello')}
()
```

被解析成不相干的两部分。
比如我们试着传参验证一下上面的说法：

```js
function foo(){console.log('hello')}(1) // 1

function foo(){console.log('hello')}
(1) // 1
```

### 方式三：

像我们平时看到的那样，我们只需要给匿名函数用圆括号包裹

```js
(function(){/* code */}());//Crockford recommends this one，括号内的表达式代表函数立即调用表达式
(function(){/* code */})();//But this one works just as well，括号内的表达式代表函数表达式
```
### 方式四：

```js
//如果你并不关心返回值，或者让你的代码尽可能的易读，你可以通过在你的函数前面带上一个一元操作符来存储字节

!function(){/* code */}();
~function(){/* code */}();
-function(){/* code */}();
+function(){/* code */}();

```

## 总结
但当括号用于调用函数表达式时，指明函数表达式将会被立即调用，并且变量将会储存函数的结果，而不是函数本身。
当这是一个非常长的函数表达式时，这可以节约比人阅读你代码的时间，不用滚到页面底部去看这个函数是否被调用。



[返回首页](/)


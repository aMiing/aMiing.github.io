# ES6 常用新特性

1. const let
2. symbol
3. 解构： `const {key1, key2} = obj;`
4. 扩展运算符 `...`:

```js
const obj2 = { ...obj1, newKey: "" };
const arr2 = [...arr1, "test"];
// 扩展运算符和解构赋值结合使用
const [first, ...rest] = [1, 2, 3, 4, 5];
// first = 1
// rest = [2,3,4,4,5]
//扩展运算符还可以将字符串转为真正的数组
[..."hello"]; //['h', 'e', 'l', 'l', 'o']
```

```js
//替换ES5中用apply方法为函数绑定参数

function f(x, y, z) {}
es5: f.apply(null, [1, 2, 3]);
es6: f(...[1, 2, 3]);
```

5. 对象属性简写： `const obj = {name, age}`
6. promise： 异步函数
7. generator, 返回迭代器对象，`yeild` 分割步伐
8. 箭头函数`()=> {};`不改变 this 指向
9. `async await` : 像编写同步代码一样编写异步逻辑
10. 函数的默认参数，给函数传递默认值，避免在函数里面写参数的判断逻辑
11. 参数剩余参数：

```js
function (arg1, ...others) {}
```

-   `others`接收剩余参数，存在类数组中。区别于`arguments`是全部参数。

12. 数组赋值

```js
const [firstName, lastName] = ["jock", "chen"];
```

13. `?.` 对象中是否包含某个属性，避免没有属性报错。返回`undefined`.
14. `??` 区别于`||`,如果前面的值为`null`或'undefined',就把后面的值赋值给前面的变量。

```js
const arg = undefined ?? "init";
```

15. 模板字符串

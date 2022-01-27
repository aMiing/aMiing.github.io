# ES6 -- 生成器函数
### 函数异步执行
```js
function * gen() {
    // yield起到分割代码块的作用
    yield console.log(111);
    yield console.log(222);
    yield console.log(333);
}
// 生成器函数返回一个迭代器，
const iterator = gen();
// 调用迭代器的next方法，执行生成器函数的第一部分
iterator.next();//111
// 执行第二部分
iterator.next();//222
```
### 给生成器函数传参

```js

function * gen(arg) {
    console.log('arg', arg);
    yield '第一部分';
    yield '第二部分';
    yield '第三部分';
}

const iterator = gen('我是gen参数');
/* arg 我是gen参数
* {value: '第一部分', done: false}
*/
iterator.next();

```

### 给next函数传参

```js

function * gen() {
    const res = yield '第一部分';
    console.log('res', res)
    yield '第二部分';
    yield '第三部分';
}

const iterator = gen();
//next的参数会作为上一次 yield 的返回值
iterator.next('第一次调用next');
iterator.next('第二次调用next'); 

// 打印结果：
// res 第二次调用next

```

### 遍历
可以使用 ```for of```遍历迭代器
```js
function * gen() {
    yield '第一部分';
    yield '第二部分';
    yield '第三部分';
}

const iterator = gen();
for(v of iterator){
    console.log(v)
}
// 也可以手动执行next方法
// next方法返回包含value和done的对象，down<Boolean>代表迭代是否结束
iterator.next();

```

[返回首页](/)
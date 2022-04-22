# 事件循环 Event Loop

## 先说结论

**微任务的执行会因为 javascript 堆栈的情况有所不同**
## 一个典型的例子
这个例子中，触发方式的不同导致主线程的占用情况不同，也导致了执行微任务的实际不同，最后的打印结果也不同。
### 如果是用户点击触发
```js
// 监听器1
btn.addEventListener("click", () => {
    Promise.resolve().then(() => console.log("microTask1"));
    console.log("Listener 1");
});
// 监听器2
btn.addEventListener("click", () => {
    Promise.resolve().then(() => console.log("microTask2"));
    console.log("Listener 2");
});
```
当用户点击btn时，会分别触发两个事件回调
1. 执行到第一个事件回调，注册微任务1到微任务队列；打印 Listener 1
2. 主线程空闲，微任务入栈，打印 microTask1
3. 接着执行第二个事件回调，同1和2步骤，打印Listener 2、 microTask2
> Listener1、 microTask1、 Listener 2、 microTask2

### 如果是 JavaScript 触发情况就不一样了

```diff
btn.addEventListener('click', () => {
    Promise.resolve().then(() => console.log('microTask1'))
    console.log('Listener 1');
})
// 监听器2
btn.addEventListener('click', () => {
    Promise.resolve().then(() => console.log('microTask2'))
    console.log('Listener 2')
})
+ btn.click();
```
与上面例子不同的是，这里是javascript触发的click事件。
1. 执行到第一个事件回调，注册微任务1到微任务队列；打印 Listener 1
2. 紧接着不是执行微任务microTask1，因为此时主线程被btn.click()占据未释放，因为btn.click()还没执行完哦！
3. 执行第二个事件回调，注册微任务2到微任务队列；打印 Listener 2
4. 此时btn.click();执行完毕，释放主线程；
5. 依次执行微任务队列的两个微任务，打印 microTask1、 microTask2
> Listener1、Listener 2、 microTask1、  microTask2

## 小结
javascript触发的事件，触发事件的脚本会占据javascript的主线程，在其（返回）被释放之前，微任务都不会被执行，所以导致了执行结果的不同。
这会带来什么隐藏问题呢？？
> 当我们在做自动化测试时，脚本模拟点击等事件时，会和实际的点击效果不完全一致。 在这种情况下应该特别注意。

## 相似的例子
当我们用promise，then中的回调是异步的。

```js
const nextClick = new Promise(resolve => {
    link.addEventListener("click", resolve, { once: true });
});

nextClick.then(event => {
    event.preventDefault();
});
```

> `event.preventDefault()`正常情况下会阻止🔗的跳转。

我们分析一下上面的代码，当用户点击link,事件回调会直接resolve, 主线程空闲，微任务`nextClick.then`的回调会被推入到主线程并执行，阻止链接跳转。

### 如果是 JavaScript 触发情况就不一样了

```diff
const nextClick = new Promise(resolve => {
    link.addEventListener("click", resolve, { once: true });
});

nextClick.then(event => {
    event.preventDefault();
});

+ link.click();
```

javascript触发点击事件，和上面的例子很像，当nextClick resolve之后，微任务并不能被推入到主线程，原因是主线程被 link.click(); 占用，还没有返回释放。
> 那么link.click();什么时候才返回呢？？ 对于link来说，只有执行了跳转才会返回。

直到 link.click(); 返回之后，才开始执行微任务。当执行到 `nextClick.then`时，跳转已经发生，此时再preventDefault已经来不及了。所以无法阻止跳转！

## 总结
本文主要结合 Jake Archibald 大佬的两个例子来谈自己对javascript执行顺序的理解，如果没有彻底搞清楚，也可以看下大佬的演讲和动画来帮助理解。链接在底部。

## 参考

[深入事件环(In The Loop)Jake Archibald@JSconf 2018](https://www.bilibili.com/video/BV1a4411F7t7/?spm_id_from=333.788.recommend_more_video.-1)

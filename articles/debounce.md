# 防抖函数

> 防抖--防止抖动，就是在事件触发的同时又有新的事件被创建或触发，设定一个时间 T，只有当事件触发 T 时间之后才会执行；在这个时间区间内，再次触发事件，只会重置这个时间 T，直到最后达到这个目标时间 T 才会真正执行。

-   常使用防抖的场景有：搜索联想、名称重复校验、resize、滚轮滚动、mousemove 等。
-   通常我们会使用 lodash 封装的 debounce 函数来实现防抖

那么如果是自己实现防抖应该怎么做呢？

首先，需要一个定时器来检验时间是否达到设定的阈值 T
`setTimeout(() => {}, t)`
当未达到预定时间再次触发事件，需要将之前的定时器清除，重新开一个定时器：

```js
function debounce() {
    let timer = null;
    clearTimeout(timer);
    timer = setTimeout(() => {
        console.log("我执行了!");
    }, 1000);
}
```

> (其实不用判断是否达到了时间阈值，因为达到了内部的函数就执行了，没达到就还没执行，每次都执行一次清除就可了)

### 初版

时间和执行函数需要从外部传递进来

```js
function debounce(func, t) {
    let timer = null;
    return () => {
        clearTimeout(timer);
        timer = setTimeout(func, t);
    };
}
```

> 存在问题： func 原本 this 的指向发生了变化
> 经过 debounce 之后，this 执行 window
> 所以我们需要手动处理一下 this 的指向问题

### 优化版

```js
function debounce(func, t) {
    let timer = null;
    return function () {
        const _this = this;
        clearTimeout(timer);
        timer = setTimeout(() => func.apply(_this), t);
    };
}
```

此时 this 的指向已经正确了。

### event 对象的处理

JavaScript 在事件处理函数中会提供事件对象 event，我们发现当前我们的封装丢掉了 event 对象。
所以，我们仍需要继续优化：

```js
function debounce(func, t) {
    let timer = null;
    return function () {
        const _this = this;
        const args = arguments;
        clearTimeout(timer);
        timer = setTimeout(() => func.apply(_this, args), t);
    };
}
```

### 实现立即执行

已经实现了延迟执行，那么可不可以控制 debounce 先立即执行一次，其后再停止触发 T 时间后在执行呢？

我们增加一个`immediate`参数,判断是否要立即执行一次

```js
function debounce(func, t, immediate = false) {
    let timer = null;
    return function () {
        const _this = this;
        const args = arguments;
        if (immediate && !timer) {
            func.apply(_this, args);
        }
        clearTimeout(timer);
        timer = setTimeout(() => func.apply(_this, args), t);
    };
}
```

### 取消

现在我希望有一个按钮，点击后，取消防抖，这样我再去触发，就可以又立刻执行啦，是不是很开心？

```js
function debounce(func, t, immediate = false) {
    let timer = null;
    const debounce = function () {
        const _this = this;
        const args = arguments;
        if (immediate && !timer) {
            func.apply(_this, args);
        }
        clearTimeout(timer);
        timer = setTimeout(() => func.apply(_this, args), t);
    };
    debounce.abort = () => {
        clearTimeout(timer);
        timer = null; //为了能立即执行
    };
    return debounce;
}
```


[返回首页](/)
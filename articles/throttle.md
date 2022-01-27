# 节流函数
- 节流简单理解就是节约流量？可能比喻成水流比较合适些。
- 在设定时间内不管触发多少次，都只会执行一次。
- 也就是说未达到设定时间就阻断触发，直到超过时间阈值才会继续执行操作。

由此我们尝试下面的程序：
## 基础实现

```js
function throttle(func, t) {
    let timer = null;
    return function () {
        if (!timer) {
            timer = setTimeout(() => {
                timer = null;
            }, t);
            func();
        }
    };
}
```

我们写个例子来测试它的效果：
```js
        let count = 1;
        const container = document.getElementById('container');
        function getUserAction() {
            container.innerHTML = count++;
        };
        container.onmousemove = throttle(getUserAction, 1000);

```
![throttle效果](/source/gifs/throttle.gif)

## 立即执行

有了防抖函数的经验，我们很容易依葫芦画瓢写出节流的立即执行：

```js
function throttle(func, t) {
    let timer = null;
    return function () {
        if (!timer) {
            + func();
            timer = setTimeout(() => {
                timer = null;
            }, t);
            - func();
        }
    };
}
```

简单通过移动func执行的位置即可。
> 其实立即执行应该更合理，我们可以默认开启立即执行，


[返回首页](/)
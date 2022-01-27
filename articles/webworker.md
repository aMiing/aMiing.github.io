# webWorker 处理异常

## 错误处理

主线程可以监听 Worker 是否发生错误。如果发生错误，Worker 会触发主线程的 error 事件。

```js
worker.onerror(function (event) {
    console.log(["ERROR: Line ", e.lineno, " in ", e.filename, ": ", e.message].join(""));
});

// 或者
worker.addEventListener("error", function (event) {
    // ...
});
```

Worker 内部也可以监听 error 事件。

## 关闭 worker

使用完毕，为了节省系统资源，必须关闭 Worker。

```js
// 主线程
worker.terminate();

// Worker 线程
self.close();
```

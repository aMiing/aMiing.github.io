
# 最简版并发任务队列

题目要求：实现一个任务队列函数，最多并发执行limit个任务

## 核心逻辑

1. 依次执行 limit 个任务
2. 任务完成递归执行下一个任务
3. 直到所有任务执行完毕

## 代码实现

```js
function execute(url) {
    return new Promise(resolve => {
        // 模拟异步请求
        setTimeout(() => {
            console.log("任务：" + url + "完成", new Date());
            resolve({ url: url });
        }, 1000);
    });
}

function taskQueue(tasks, limit) {
    // 1. 依次执行limit个任务
    for (let index = 0; index < limit; index++) {
        run(tasks.shift(), tasks);
    }
}

function run(task, tasks) {
    new Promise((resolve, reject) => {
        resolve(execute(task).then());
    }).then(() => {
        // 2. 任务完成递归执行下一个任务
        // 3. 直到所有任务执行完毕
        tasks.length && run(tasks.shift(), tasks);
    });
}
// 测试
taskQueue([1, 2, 3, 4, 5, 6, 7], 3);
```

## 总结
熟练掌握 **递归**， **遍历** 等编程技巧，轻松应对面试常考题目。

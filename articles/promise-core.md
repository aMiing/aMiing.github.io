# 手写 Promise 的核心逻辑

## 前言

> 面试官：能说下promise实现异步的原理吗？
> 
> 我：...应该是通过状态管理来实现的吧，其他我记不清了。。。
> 
> 面试官：我手机就剩下95%的电了,今天就先到这吧。

当面试官问到promise的实现原理时，仅仅回答出promise的三种状态，以及通过状态管理来实现异步是远远不够的。
面试官想听到的是你对promise原理的深入了解，说出具体实现细节。

所以要轻松应对面试官的夺命追问，就需要深入研究其原理，手写promise函数可以算得上是一条捷径了。

闲言碎语不要讲，直接上代码。

## 基础promise

```js
// 声明状态
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

class MyPromise {
    constructor(executor) {
        // executor执行器，进入会立即执行
        executor(this.resolve, this.reject)
    }
    // 初始状态
    state = PENDING;
    // 存储异步回调
    fulfilledCallBack = null;
    rejectedCallBack = null;

    // 成功之后的值
    value = null;
    // 失败的原因
    reason = null;

    // 成功回调
    resolve = (value) => {
        console.log('execute resolve function');
        if(this.state === PENDING) {
            this.state = FULFILLED;
            this.value = value;
            // 是否有回调可执行
            this.fulfilledCallBack && this.fulfilledCallBack(value);
        }
    }
    // 拒绝回调
    reject = (reason) => {
        if(this.state === PENDING) {
            this.state = REJECTED;
            this.reason = reason;
            this.rejectedCallBack && this.rejectedCallBack(reason);
        }
    }
    then(onFulfilled, onRejected) {
        if(this.state === FULFILLED) {
            onFulfilled(this.value);
        }else if(this.state === REJECTED) {
            onRejected(this.reason);
        }else if(this.state === PENDING) {
            // 存储回调
            this.fulfilledCallBack = onFulfilled;
            this.rejectedCallBack = onRejected;
        }
    }
}
```

这部分很好理解，回想下我们平时使用Promise的方式：
```js
new Promise((resolve, reject) => {resolve();...})
.then(successCB, failedCB)
```
所以`constructor`会需要接收一个函数，参数就是resolve和reject回调。

我们来测试下效果：
```js
new MyPromise((resolve, reject) => {
    console.log('in promise', new Date().getTime());
    setTimeout(() => {
        resolve('3s 之后执行结果');
    }, 3000)
}).then(res => console.log(res, new Date().getTime()));
```
结果：
![image1.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/518cb8ead1694c24aa42f5a104d37cc6~tplv-k3u1fbpfcp-watermark.image?)
> 根据打印结果，我进一步理解到 Promise 是在 异步方法（这里是settimeout模拟的） 执行完之后才调用的resolve方法，这不就是我们在ES6之前常用的回调吗？
> 
## 链式调用

我们知道Promise后面可以跟多个then方法为此，我们需要把多个then回调用 **栈** 的思想存储起来，当resolve回调触发之后，依次从栈里面取出回调并执行。
> 注意javascript里面并没有栈这种数据结构，我们可以通过数组的方法模拟实现。

```js
// 状态
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

class MyPromise {
    constructor(executor) {
        // executor执行器，进入会立即执行
        executor(this.resolve, this.reject)
    }
    // 初始状态
    state = PENDING;
    // 存储异步回调
    fulfilledCallBacks = [];
    rejectedCallBacks = [];

    // 成功之后的值
    value = null;
    // 失败的原因
    reason = null;

    // 成功回调
    resolve = (value) => {
        if(this.state === PENDING) {
            this.state = FULFILLED;
            this.value = value;
            // 是否有回调可执行
            while(this.fulfilledCallBacks.length) this.fulfilledCallBacks.shift()(value);
        }
    }
    // 拒绝回调
    reject = (reason) => {
        if(this.state === PENDING) {
            this.state = REJECTED;
            this.reason = reason;
            while(this.rejectedCallBacks.length) this.rejectedCallBacks.shift()(reason); 
        }
    }
    then(onFulfilled, onRejected) {
        if(this.state === FULFILLED) {
            onFulfilled(this.value);
        }else if(this.state === REJECTED) {
            onRejected(this.reason);
        }else if(this.state === PENDING) {
            // 存储回调
            this.fulfilledCallBacks.push(onFulfilled);
            this.rejectedCallBacks.push(onRejected);
        }
    }
}

```
测试一下：
```js
const promise = new MyPromise((resolve, reject) => {
    console.log("in promise", new Date().getTime());
    setTimeout(() => {
        resolve(2);
    }, 3000);
})
promise.then(res => {
    console.log('first', res);
})
promise.then(res => {
    console.log('second', res);
});
promise.then(res => {
    console.log('third', res);
});

```
## 链式调用

- then 方法要链式调用那么就需要返回一个 Promise 对象
- then 方法里面 return 一个返回值作为下一个 then 方法的参数，如果是 return 一个 Promise 对象，那么就需要判断它的状态

我们在原来的基础上进行修改：

```js
// 状态
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

class MyPromise {
    constructor(executor) {
        // executor执行器，进入会立即执行
        executor(this.resolve, this.reject);
    }
    // 初始状态
    state = PENDING;
    // 存储异步回调
    fulfilledCallBacks = [];
    rejectedCallBacks = [];

    // 成功之后的值
    value = null;
    // 失败的原因
    reason = null;

    // 成功回调
    resolve = value => {
        if (this.state === PENDING) {
            this.state = FULFILLED;
            this.value = value;
            // 是否有回调可执行
            while (this.fulfilledCallBacks.length) {
                this.fulfilledCallBacks.shift()();
            }
        }
    };
    // 拒绝回调
    reject = reason => {
        if (this.state === PENDING) {
            this.state = REJECTED;
            this.reason = reason;
            while (this.rejectedCallBacks.length) this.rejectedCallBacks.shift()(reason);
        }
    };
    then(onFulfilled, onRejected) {
        const promise2 = new MyPromise((resolve, reject) => {
            // 成功
            const resolveMicrotask = () => {
                // 避免循环调用
                // 这里有个问题，promise2这里其实是拿不到的，因为promise2还没有完成初始化
                // 这里需要用创建一个微任务，在微任务里面调用到的就是初始化完成的promise2。
                // 我们用 queueMicrotask 创建微任务
                queueMicrotask(() => {
                    // then执行阶段错误捕获
                    try {
                        const x = onFulfilled(this.value);
                        this.resolvePromise(x, promise2, resolve, reject);
                    } catch (err) {
                        reject(err);
                    }
                });
            };
            // 失败
            const rejectMicrotask = () => {
                queueMicrotask(() => {
                    try {
                        const x = onRejected(this.reason);
                        this.resolvePromise(x, promise2, resolve, reject);
                    } catch (err) {
                        reject(err);
                    }
                });
            };

            if (this.state === FULFILLED) resolveMicrotask();
            else if (this.state === REJECTED) rejectMicrotask();
            else if (this.state === PENDING) {
                // 存储回调
                this.fulfilledCallBacks.push(resolveMicrotask);
                this.rejectedCallBacks.push(rejectMicrotask);
            }
        });
        return promise2;
    }
    resolvePromise(x, self, resolve, reject) {
        // 不能返回自身（循环调用）
        if (x === self) {
            return reject(new TypeError("The promise and the return value are the same"));
        }
        // 如果返回一个Promise对象，调用其then方法
        if (x instanceof MyPromise) {
            x.then(resolve, reject);
        } else {
            // 直接返回X
            resolve(x);
        }
    }
}
```

测试一下：
```js

const promise = new MyPromise((resolve, reject) => {
    // 目前这里只处理同步的问题
    setTimeout(() => {
        resolve(2);
    }, 1000);
});

promise
    .then(value => {
        console.log(1);
        console.log("resolve", value);
        return value * 2;
    })
    .then(value => {
        console.log(2);
        console.log("resolve", value);
        return ++value;
    })
    .then(value => {
        console.log(3);
        console.log("resolve", value);
        // return value;
    });

```
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0969f4de269145599aeb8a28980d263e~tplv-k3u1fbpfcp-watermark.image?)

> 链式调用的核心就是每次then都返回一个promise, 将then里的回调包装成promise,，依次执行resolve方法。

## 完善代码


```js
// 状态
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

class MyPromise {
    constructor(executor) {
        // 捕获执行器的代码错误
        try{
            // executor执行器，进入会立即执行
            executor(this.resolve, this.reject)
        } catch(err) {
            this.reject(err);
        }
    }
    // 初始状态
    state = PENDING;
    // 存储异步回调
    fulfilledCallBacks = [];
    rejectedCallBacks = [];

    // 成功之后的值
    value = null;
    // 失败的原因
    reason = null;

    // 成功回调
    resolve = (value) => {
        if(this.state === PENDING) {
            this.state = FULFILLED;
            this.value = value;
            // 是否有回调可执行
            while(this.fulfilledCallBacks.length) this.fulfilledCallBacks.shift()();
        }
    }
    // 拒绝回调
    reject = (reason) => {
        if(this.state === PENDING) {
            this.state = REJECTED;
            this.reason = reason;
            while(this.rejectedCallBacks.length) this.rejectedCallBacks.shift()(); 
        }
    }
    then(onFulfilled, onRejected) {
        // 如果不传，就使用默认函数
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
        onRejected = typeof onRejected === 'function' ? onRejected : reason => {throw reason};

        const promise2 = new MyPromise((resolve, reject) => {
            // 成功
            const resolveMicrotask = () => {
                queueMicrotask(() => {
                    // then执行阶段错误捕获
                    try{
                        const x = onFulfilled(this.value);
                        this.resolvePromise(x, promise2, resolve, reject);
                    }catch(err) {
                        reject(err);
                    }
                })
            }
            // 失败
            const rejectMicrotask = () => {
                queueMicrotask(() => {
                    try{
                        const x = onRejected(this.reason);
                        this.resolvePromise(x, promise2, resolve, reject);
                    }catch(err) {
                        reject(err);
                    }
                })
            }

            if(this.state === FULFILLED) resolveMicrotask();
            else if(this.state === REJECTED) rejectMicrotask();
            else if(this.state === PENDING) {
                // 存储回调
                this.fulfilledCallBacks.push(resolveMicrotask);
                this.rejectedCallBacks.push(rejectMicrotask);
            }
        })
        return promise2;
    }
    resolvePromise(x, promise, resolve, reject) {
        if(x === promise) {
            return reject(new TypeError('The promise and the return value are the same'));
        }
        if(typeof x === 'object' || typeof x === 'function') {
            if(x === null) {
                return resolve(x);
            }
            let then;
            try{
                then = x.then;
            }catch(err) {
                return reject(err);
            }

            if(typeof then === 'function') {
                let called = false;
                try{
                    then.call(x, y => {
                        if(called) return;
                        called = true;
                        resolvePromise(y, promise, resolve, reject);
                    }, r => {
                        if(called) return;
                        called = true;
                        reject(r);
                    })
                }catch(err) {
                    if(called) return;
                    reject(err);
                }
            }else{
                resolve(x);
            }
        }
        else {
            resolve(x);
        }
    }
    // 静态resolve方法
    static resolve = (value) => {
        if(value instanceof MyPromise) {
            return value;
        }
        // 常规resolve处理
        return new MyPromise((resolve, reject) => {
            resolve(value);
        })
    }
    // 静态reject方法
    static reject = (reason) => {
        return new MyPromise((resolve, reject) => {
            reject(reason);~
        })
    }
}
```

增加`resolve\reject`静态方法是为了满足用例：

`Promise.resolve(4)` 会直接返回fulfilled状态和4.
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8655be8b7465448a88fbbb0637f00413~tplv-k3u1fbpfcp-watermark.image?)
reject同理。

## Promise A+ 规范

 Promise 有多种规范，除了 Promise A、promise A+ 还有 Promise/B，Promise/D。**目前我们使用的 Promise 是基于 Promise A+ 规范实现的**，感兴趣的移步 [Promise A+](https://promisesaplus.com/) 规范了解一下，这里不赘述。
检验一份手写 Promise 靠不靠谱，通过 Promise A+ 规范自然是基本要求，这里我们可以借助 promises-aplus-tests 来检测我们的代码是否符合规范。

1. 安装一下
`npm install promises-aplus-tests -D`

2. 手写代码中加入 deferred
```js

MyPromise {
  ......
}

MyPromise.deferred = function () {
  var result = {};
  result.promise = new MyPromise(function (resolve, reject) {
    result.resolve = resolve;
    result.reject = reject;
  });

  return result;
}
module.exports = MyPromise;

```

3. 配置启动命令
 ```json
{
  "name": "promise",
  "version": "1.0.0",
  "description": "my promise",
  "main": "MyPromise.js",
  "scripts": {
    "test": "promises-aplus-tests MyPromise"
  },
  "author": "ITEM",
  "license": "ISC",
  "devDependencies": {
    "promises-aplus-tests": "^2.1.2"
  }
}
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/039cf9c282d944bc9d33320d5467d338~tplv-k3u1fbpfcp-watermark.image?)


最终通过所有测试用例。



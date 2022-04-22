## 字节

### 一面

-   常用选择器
    <!-- class, id, tagname,  &， > , + , ~ , *-->
-   如何管理代码中的选择器的嵌套
    <!-- -   选择器的分组管理 -->
    <!-- 非必要不嵌套，只有伪类，伪元素，动态样式时建议使用选择器嵌套 -->
    <!-- 命名原则： 抽象、模糊、复用 -->
    <!-- 尽量避免使用 ul.nav tagname+class 的命名， 会降低体系结构的扩展能力和复用能力 -->
    <!-- 使用类名来描述内容是多余的，因为内容描述了它自己。 -->

    <!-- 不要不必要地嵌套选择器；不要不必要地限定选择器； 保持选择器尽可能短 -->

-   嵌套过多会有什么问题
    <!-- 降低选择器性能： 一般来说，选择器越长越慢；嵌套选择是从右向左查找的，所以查找过程越明确越快。.box > .img 优于 .box .img 。因为从.img 向上查找 会查找其直接父元素，找不到也会停止。 -->
    <!-- 增加特异性，降低可复用性 -->
-   cookie 安全隐患
    <!-- -  csrf/xsrf攻击，通过伪造目标站点请求非法操作，现代浏览器的 same-site 可以有效避免受此攻击。为了更好的兼容性，可以使用token代替cookie做状态保持  -->
    <!-- xss攻击，数据安全攻击，在表单中携带script获取cookie数据，现代浏览器的http-only可以优先避免此攻击。 -->
-   你做过的性能优化
    <!-- -代码层面：   1、RAF： 在线程空闲时执行代码逻辑；2、避免css嵌套过深； 3、耗时任务分割到web worker，减少主线程阻塞； -->
    <!-- 打包优化：1、webpack splitChunks拆分公共模块，提高缓存利用率； 2、 组件按需加载； 3、 tree shaking 缩减代码体积 -->
    <!-- 非关键js\css延迟加载： 1、defer、async、动态加载js -->
    <!-- 媒体资源加载优化： 1、图片视频懒加载；2、资源压缩，根据需要选择合适分辨率的图片资源 -->
    <!-- 页面渲染优化： 1、ssr页面首屏css内联； 2、 合理使用Layers； 3、布局抖动优化，提前定好宽高； 4、 减少重排重绘； -->
-   vuex 的理解
    <!-- vue中的数据流是单向传递的，vuex是为了解决多层组件嵌套状态传递的问题，集中式状态管理模式 -->
    <!-- vuex通过状态集中管理驱动组件的变化， 应用级的数据存储在store中，改变状态的方式是提交mutations, 一步的逻辑封装到actions中，vuex使用单一状态树，即用一个对象包含所有的状态数据。state作为构造器选项，定义了所有基本状态参数-->
-   raf 的调用机制
<!-- raf的回调会在浏览器下一次重绘之前执行 -->
-   如何让代理 focus/blur
<!-- -  不支持冒泡，可以在捕获阶段监听，el.addEventListener('blur', callback, true); IE中使用focusin\focusout达到相似的效果，支持冒泡。  -->
-   手写 lodash.get()

```js
function get(source, path, defaultValue = undefined) {
    // a[3].b -> a.3.b
    const paths = path.replace(/\[(\d+)\]/g, ".$1").split(".");
    let result = source;
    for (const p of paths) {
        result = Object(result)[p];
        if (result === undefined) {
            return defaultValue;
        }
    }
    return result;
}
```

-   js 如何处理 64 位的数字

### 二面

-   TCP/UDP 区别
-   数组和链表的区别以及优缺点
-   孤岛算法题
    ~~- 手写 Promise.all~~

```js
function MyPromiseAll(list) {
    const res = [];
    return new Promise((resolve, reject) => {
        for (let i = 0; i < list.length; i++) {
            list[i]
                .then(data => {
                    res.push(data);
                    res.length === list.length && resolve(res);
                })
                .catch(err => {
                    reject(err);
                });
        }
    });
}

const p1 = new Promise(resolve => {
    setTimeout(() => {
        resolve("我是p1");
    }, 1000);
});
const p2 = new Promise(resolve => {
    setTimeout(() => {
        resolve("我是p2");
    }, 2000);
});

const p3 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject("我是err: p3");
    }, 2000);
});

Promise.all([p1, p3, p2])
    .then(data => {
        console.log("promise data:", data);
    })
    .catch(err => {
        console.log("promise err", err);
    });

MyPromiseAll([p1, p3, p2])
    .then(data => {
        console.log("MyPromiseAll data:", data);
    })
    .catch(err => {
        console.log("err", err);
    });
```

### 三面

-   七层网络模型
-   输入 url 到显示页面的过程
-   二叉查找树是什么，如何从小到大输出
<!-- -  搜索二叉树、排序二叉树。 1、 若左子树不空，则左子树上所有结点的值均小于它的根结点的值；2、 若右子树不为空，则右子树上的所有节点的值均大于根节点的值；3、 左右子树也分别为二叉搜索树；   -->
<!-- 从小到大输出只需要中序遍历 -->
-   数组转成二叉查找树再顺序输出
-   缓存的目的是什么？什么场景需要使用缓存

## 智云

### 一面

-   flex 布局
-   原型链
-   diff 算法
-   执行上下文的概念
-   计算属性的实现
-   watch 函数
-   vuex 的理解
-   vue-router 的原理
-   浏览器缓存机制

### 二面

-   父子组件的生命周期 （父 beforeCreate->父 created->父 beforeMount->子 beforeCreate->子 created->子 beforeMount->子 mounted->父 mounted）
    ~~- 手写 new~~

```js
function MyNew(Obj, ...args) {
    const obj = {};
    Obj.apply(obj, args);
    obj.__proto__ = Obj.prototype;
    return obj;
}

function Student(name) {
    this.name = name;
}
const p1 = new Student("xiaoM");
const p2 = MyNew(Student, "xiaoM");

console.log("p1", p1);
console.log("p2", p2);
```

-   手写深拷贝
-   vue 如何处理数组的问题
-   defineProperty 的参数
-   Proxy 的参数
-   WeakMap 和 Map 的区别

### 三面

-   变量作用域的查找
-   v-model 的实现，以及如何在组件中使用 v-model（使用 prop 接收，再 this.\$parent.value 赋值）
-   手写节流函数

```js

```

## 微店

-   Object.create 方法
-   浅拷贝
-   vue 监听数组的处理
-   diff 算法
-   原型链继承的优缺点（子类没有属于自己的方法，共用父类方法）

## 欢聚时代

### 一面

-   csrf/xss 异同
-   浏览器缓存机制

## 滴滴

### 一面

-   html 为什么会忽略空白，文本换行问题怎么处理，有一个 css 属性控制(white-space)
-   手写动态拼接 script
-   对象解析为 key=value 的格式
-   手写 Promise.all

## e 签宝

### 一面

-   watch 原理
-   qiankun 原理
-   webpack 流程
-   循环依赖
-   根据注释生成文档实现思路
-   babel 流程
-   如何帮助组员成长
-   架构一个 npm 包需要考虑哪些

## 政采云

### 一面

-   箭头函数 this 指向
-   diff 的过程
-   如何冻结一个对象
-   换肤的时候有没有考虑使用构建工具解决
-   如何控制包的版本
    ~~- 实现一个最高并发量为 10 的 Promise 队列~~
-   在项目有突发情况（未按时、重难点无法解决等）的时候如何去解决
-   如何控制代码质量

### 二面

-   vue 如何代理数组
-   qiankun 和 Single-spa 原理
-   如何帮助同事成长

## 钉钉

### 一面

-   个人公司经历以及离职原因
-   现在做的项目展示之类的
-   项目内的技术难点
-   展示自己的作品
-   算法题 数字转千分位展示
-   算法题 版本比较

## 微策略

### 笔试

-   全英文笔试题(10 道选择题 + 3 道算法题)
-   选择题主要是基于 java 的，还有一些算法数据结构
-   二叉搜索树会否存在目标值
-   字符串前缀是否相等
-   反转链表

## 得物

### 笔试

#### 第一题

```js
// 实现一个 arrange 函数，可以进行时间和工作调度
// arrange('William').execute();
// > William is notified

// arrange('William').do('commit').execute();
// > William is notified
// > Start to commit

// arrange('William').wait(5).do('commit').execute();
// > William is notified
// 等待 5 秒
// > Start to commit

// arrange('William').waitFirst(5).do('push').execute();
// 等待 5 秒
// > William is notified
// > Start to push

function arrange(name) {
    this.promises = [];
    this.promises.push(Promise.resolve(`${name} is notified`));

    this.do = function (todo) {
        this.promises.push(Promise.resolve(`Start to ${todo}`));

        return this;
    };

    function genWaitPromise(second) {
        return new Promise(resolve => {
            setTimeout(() => {
                resolve(`等待${second}秒`);
            }, second * 1000);
        });
    }

    this.wait = function (second) {
        this.promises.push(genWaitPromise(second));

        return this;
    };

    this.waitFirst = function (second) {
        this.promises.unshift(genWaitPromise(second));

        return this;
    };

    this.execute = async function () {
        const len = this.promises.length;
        let i = 0;

        while (i < len) {
            const res = await this.promises[i];
            console.log(res);
            i++;
        }
    };

    return this;
}
```

#### 第二题

```js
// 扁平数据转树形结构
function buildTree(arr) {
    const len = arr.length;
    const result = [];
    const map = {};
    const notFoundMap = {};

    for (let i = 0; i < len; i++) {
        const { id, parentId } = arr[i];
        const node = { ...arr[i], children: [] };
        map[id] = node;

        if (notFoundMap[id]) {
            node.children = notFoundMap[id];
            delete notFoundMap[id];
        }

        if (!parentId) {
            result.push(node);
        } else if (map[parentId]) {
            map[parentId].children.push(node);
        } else if (!notFoundMap[parentId]) {
            notFoundMap[parentId] = [node];
        } else {
            notFoundMap[parentId].push(node);
        }
    }

    // 有未找到的就放入根
    Object.values(notFoundMap).forEach(item => {
        result.push(...item);
    });

    return result;
}
```

### 第三题

```js
// 实环一个异步任务处理区数，区数第一个参数 asyncTasks 代表需要处理的任务列表
// 第一个参数 n 代表可么同的发起的任务数。所有任务完成后把处理结果按顺序放在数组里返回
// 1s
// > resolve task 1
// 1s
// > resolve task 2 // 1s
// > resolve task 3
const asyncTask1 = () =>
    new Promise(resolve =>
        setTimeout(() => {
            console.log("resolve task 1");
            resolve();
        }, 1000)
    );
const asyncTask2 = () =>
    new Promise(resolve =>
        setTimeout(() => {
            console.log("resolve task 2");
            resolve();
        }, 2000)
    );
const asyncTask3 = () =>
    new Promise(resolve =>
        setTimeout(() => {
            console.log("resolve task 3");
            resolve();
        }, 2000)
    );
const asyncTasks = [asyncTask1, asyncTask2, asyncTask3];
handleAsyncTasks(asyncTasks, 2);

function handleAsyncTasks(array, poolLimit) {
    let i = 0;
    const result = [];
    // 保存正在执行的promise
    const executing = [];

    function enqueue() {
        // 边界处理，array为空数组
        if (i === array.length) return Promise.resolve();

        // 每调用一次enqueue，执行并返回一个Promise
        const fn = array[i++];
        const p = Promise.resolve().then(fn);
        // 将初始化的promise放入promises数组
        result.push(p);

        // promise执行完毕，从executing数组中删除
        const e = p.then(() => executing.splice(executing.indexOf(e), 1));
        executing.push(e);

        let r = Promise.resolve();
        // 使用Promise.race，获得executing中promise的执行情况
        // 每当正在执行的promise数量高于poolLimit，就执行一次 否则继续实例化新的Promise达到poolLimit时执行
        if (executing.length >= poolLimit) r = Promise.race(executing);

        // 递归，直到遍历完array
        return r.then(() => enqueue());
    }

    //所有的 promise 都执行完了，调用Promise.all返回
    return enqueue().then(() => Promise.all(result));
}
```

#### 第四题

```js
// 力扣 40 题 组合总数 II
function getAllArrays(array, value) {
    array.sort((a, b) => a - b);
    const len = array.length;
    const result = [];

    // 深度优先遍历 + 循环
    function dfs(pos, value, path) {
        if (value === 0) {
            result.push(path);
            return;
        }

        for (let i = pos; i < len; i++) {
            const item = array[i];

            // 如果超过了value直接break
            if (value < item) break;
            // 如果不是第一个，并且和前一个相同则表示
            if (i > pos && item === array[i - 1]) continue;
            dfs(i + 1, value - item, [...path, item]);
        }
    }

    dfs(0, value, []);

    return result;
}
```

## 字节

### 一面

-   flex
-   模块化机制 commonJs ESM
-   ~~hash 和 history 区别~~
-   ~~如何判断一个变量是否数组，考察原型链~~
    Object.prototype.toString.call(arg)
-   ~~作用域链~~ function.**proto** === Function.prototype
-   csrf / xss
-   ~~Promise 机制，常用函数~~ then\ all\ race\ allSettled
-   ~~span 的 margin 怎么生效~~ 转成非行内元素 diasplay
-   ~~浏览器缓存~~ cookie\localStorage\sessionStorage\indexDB
-   ~~请求头常见属性~~ request Url、method\ Status Code\ remote Address\ \accept\accept-encode\accept language\content length\origin\referer\user-agent\cache-control\cookie\range
-   ~~http 状态码，404 代表什么~~ 1**:临时响应； 2**：代表成功；3**：代表重定向；4**：请求错误；5\*\*：服务器错误
-   手写 setTimeout 的相关题目，考察 js 运行机制
-   ~~手写算法题，数字转千分位展示~~

```js
// 千分位分隔
function seprate(str) {
    const list = str.toString().split(".");
    const res = [[], []];
    for (let i = list[0].length; i > 0; i = i - 3) {
        const element = list[0].substring(i - 3, i);
        console.log(element);
        res[0].unshift(element);
    }
    for (let j = 0; j < list[1].length; j = j + 3) {
        const element = list[1].substring(j, j + 3);
        res[1].push(element);
    }
    return res.map(e => e.join(",")).join(".");
}

console.log(seprate(12345678.1415926));
```

### 二面

-   304 ：未修改 自上次请求后，请求的网页未修改过。服务器返回此响应，不会返回网页的内容。last-modify-since
-   协商缓存的 etag 和 LastModify 信息是哪来的
-   缓存的应用场景
-   cdn 原理
-   js 内存机制，内存泄漏场景
-   qiankun 架构做了那些深入应用
-   npm 包如何控制版本变更
-   首屏优化思路
-   gc 流程
-   离职原因
-   this 指向的一道题
-   手写内存泄漏场景
-   实现包含数组和加减乘除的字符串的计算(不含括号)

### 三面

-   qiankun 和 single-spa 原理
-   工程化有什么实践，说了下自己写的 babel 插件
-   换肤功能具体怎么做的，现在做的话还有什么思路吗？说了使用 postcss 处理
-   经历介绍，离职原因
-   组件库的规划有什么想法
-   最长回文子串

## 网易

### 一面

主要是根据工作经历提问

-   promise.then 的返回值是什么
-   离职原因
-   手写事件监听器
-   为什么要 fork element-ui
-   低代码平台的难点在哪
-   > 易用性和灵活性之间的取舍与平衡

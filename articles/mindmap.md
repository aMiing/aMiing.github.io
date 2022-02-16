# 前端技能树

## 领域知识
### 可视化方向
#### SVG
#### Canvas
##### 基本用法
##### 2D上下文
##### 伪3D

#### WebGl

### 3D
#### ThreeJs

### 地图相关
### 异常监控
### 大数据
#### ODPS
##### 阿里云基于云计算技术研发的开放数据处理服务
#### Hadoop(HDFS)
##### 由Apache基金会开发的分布式系统基础架构

#### Hive
##### 基于Hadoop的一个数据仓库工具

#### HBase
##### 分布式、面向列的开源数据库
##### Apache软件基金会开放的开源流处理框架

#### Cassandra
##### 一个高可靠的大规模分布式存储系统

#### Spark
##### 专为大规模数据处理而设计的快速通用的计算引擎

#### ClickHouse
#### 数据湖

### low-code、no-code

### 微前端
#### [qiankun工作原理](https://app.yinxiang.com/fx/15c8d254-e1f3-47a8-b828-ef5ec7f54505)
##### 沙箱机制
##### css样式隔离

#### Garfish

#### singleSpa

### 移动端
#### H5
#### 小程序
#### RN
#### Weex

### 微服务
### ServerLess


## JavaScript

### 数据类型
#### 基础数据类型
##### String
##### Number
###### NaN
###### 浮点数精度问题

##### Boolean
##### Undefined
##### Null

#### 引用数据类型
##### Object
##### Array
##### Date
##### RegExp
##### Function


### 变量
#### var
#### let
##### [变量提升、暂时性死区](http://https://juejin.cn/post/7062684000002768933)

#### const

### 作用域、执行上下文
#### bind、apply、call
#### [this的指向](https://juejin.cn/post/7063047546205110279)

### BOM
#### window对象
#### location对象
#### navigator对象
#### screen
#### history



### DOM
#### 选择符API
#### 元素遍历
##### NodeInterator
##### TreeWalker

#### children()
#### contains()
#### 样式
##### 访问元素样式
##### 操作样式表
##### 元素大小


### 事件
#### 事件流
##### 冒泡
##### 捕获

#### 事件类型
##### UI
##### 焦点事件
##### 鼠标与滚轮
##### 键盘与文本
##### 变动事件
##### 设备事件
##### 触摸与手势

#### 事件代理、事件委托
#### 事件模拟与触发

### [事件循环机制(event loop)](https://www.jianshu.com/p/12b9f73c5a4f/)
#### 函数调用栈(call stack)
#### 任务队列(task queue)
#### macro-task（宏任务）
#### micro-task（微任务）

### 异步编程
#### 回调函数
#### Promise (all、race、allSettled, any)
#### Generator 函数
##### 可迭代协议（for of循环）
##### 迭代器协议：（Iterator对象）
#### async/await


### 面向对象
#### 继承
#### 原型链

### 函数表达式
#### 闭包
#### 递归
#### 私有变量
#### 模仿块级作用域
#### 柯里化

### 编程技巧
#### [节流、防抖](https://amiing.github.io/articles/debounce)
#### 懒加载

### 模块化
#### commonJs
#### cmd\amd\seaJs
#### ESM 
#### [模块化机制（commonjs、ESM）](https://segmentfault.com/a/1190000017466120)


### JSON
#### 语法
#### 解析与序列化
##### 序列化选项
##### 解析选项
##### JSON.parse原理
#### 深浅克隆

### 新兴API
#### requestAnimationFrame()
#### Page Visibility API
#### Geolocation API
#### File API
#### Web 计时
#### Web Workers
#### scrollIntoView()
#### Observer API
##### [交叉监视器IntersetionObserver](https://juejin.cn/post/7057335588688494622)
##### ResizeObserver
##### MutationObserver
##### PerformanceObserver

### 严格模式

## TypeScript
### 类型基础
### 基础类型
### 枚举类型
### 接口
### 函数
### 类
### 泛型
### 类型检查机制
### 高级类型
### infer


## ES6+语法和特性
### ECMAScript介绍
### let\cont
### 变量的解构与赋值
### 扩展功能
#### 字符串的扩展
##### String.trimStart()和String.trimEnd()
##### String.prototype.matchAll

#### 正则的扩展
#### 数值的扩展
##### 指数操作符

#### 函数的扩展
##### 箭头函数
##### 参数默认值

#### 数组的扩展
##### includes()
##### Array.flat()和Array.flatMap()

#### 对象的扩展
##### Object.values()
##### Object.entries()
##### Object.getOwnPropertyDescriptors()
#### 运算符的扩展

### 新增方法
#### 字符串的新增方法
#### 对象的新增方法
#### globalThis 顶层作用域

### Set 和 Map 数据结构
### Symbol、 BigInt()
### Proxy
### Reflect
### Promise (all、race、allSettled, any)
### Iterator 和 for...of 循环
### Generator 函数
### async 函数
### Class 
### ESModule
### 可选链 (?.)
### 空值处理 (??)
### replaceAll()
### WeakRefs
### 数字分隔符(_)
### ArrayBuffer 对象


## 正则
### [语法](https://www.runoob.com/regexp/regexp-syntax.html)
#### 普通字符
#### 非打印字符
#### 特殊字符
#### 限定符
#### 定位符
#### 选择
#### 反向引用

### 修饰符
#### g
#### i
#### m
#### s

### [元字符](https://www.runoob.com/regexp/regexp-metachar.html)
### [运算符优先级](https://www.runoob.com/regexp/regexp-operator.html)
### [匹配规则](https://www.runoob.com/regexp/regexp-rule.html)


## HTML
### 元数据（meta）
###  html标签元素、语义化
### [Dom树](https://blog.poetries.top/browser-working-principle/guide/part5/lesson22.html#javascript-%E6%98%AF%E5%A6%82%E4%BD%95%E5%BD%B1%E5%93%8D-dom-%E7%94%9F%E6%88%90%E7%9A%84)
### 媒体元素
### <script>元素
#### 标签位置
#### 延迟脚本
#### 异步脚本

### <noscript>元素

## 浏览器
### [浏览器架构](https://xie.infoq.cn/article/5d36d123bfd1c56688e125ad3)
### [浏览器工作原理](https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/)
### [webWorker](https://app.yinxiang.com/fx/b6e98c3d-06c1-4c6b-871b-6321ac6ad875)
### [serviceWorker](https://app.yinxiang.com/fx/93d0f8a1-51be-48bb-aba7-e8bfbb5533d0)
### [跨标签页通信方式](https://juejin.cn/post/6844903811232825357)
### [缓存策略](https://juejin.cn/post/6844903593275817998)
### [history和hash路由模式](https://blog.csdn.net/Charissa2017/article/details/104779412)
### [事件模型](https://javascript.ruanyifeng.com/dom/event.html)
### [V8垃圾回收机制](https://juejin.cn/post/6844904016325902344)

## CSS
### [盒子模型](https://segmentfault.com/a/1190000013069516)
### 布局
#### flex
#### grid布局
#### [浮动、空间塌陷](https://segmentfault.com/a/1190000012739764)
#### [BFC 块级格式化上下文](https://zhuanlan.zhihu.com/p/25321647)
#### [IFC 行内格式上下文](https://juejin.cn/post/6844903507007553550#heading-5)

### css选择器、伪类
#### [css优先级](https://zhuanlan.zhihu.com/p/41604775)

### 过渡与动画
### 预编译
#### less
#### sass
#### postcss
### 命名规范


### css样式隔离
### [css性能](https://blog.csdn.net/weixin_43883485/article/details/103504171)
### css3新特性
### [层叠上下文](https://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/)


## 错误处理与调试
### 浏览器报告的错误

### 错误处理
#### try-catch
#### 错误抛出
#### 错误类型
#### 错误事件
### 调试
#### 调试工具
#### 调试方法

## 网络原理
### 应用层
#### [http与https](https://zhuanlan.zhihu.com/p/26682342)
#### http1.0、http2.0、http3.0
##### 状态码

#### Websocket
#### DNS
#### FTP
#### SMTP

### 传输层
#### TCP
##### [TCP的三次握手与四次挥手](https://www.cnblogs.com/amingxiansen/p/9072074.html)
#### UDP
### OSI七层模型

## Ajax与Comet
### Ajax封装
### Fetch
### XMLHttpRequest对象
#### 请求类型

### XMLHttpRequest2
### 进度事件
#### load
#### progress

### 跨源资源共享
#### CROS的实现
#### 图像Ping
#### JSONP
#### Comet
#### 服务器发送事件

## 离线应用与客户端存储
### 离线检测
### 应用缓存
### 数据存储
#### [Web存储机制](https://segmentfault.com/a/1190000013896386)
##### Cookie
##### localStorage
##### sessionStorage
##### IndexDB
##### WebSQL

### PWA

## 性能
### 性能评价指标
#### FCP
#### FPS

### [重排与重绘](https://juejin.cn/post/6844904083212468238)
### [白屏优化](https://cloud.tencent.com/developer/article/1508941)
### [大量图片加载优化](https://zhuanlan.zhihu.com/p/33370207)
### [前端性能优化指标RAIL](https://juejin.cn/post/6850037273312886797)
### 性能优化手段
#### [preload、prefetch、preconnect](https://www.naeco.top/2021/02/28/preload-prefetch-preconnect/)
#### 具体手段
### [动画性能](https://www.jianshu.com/p/d24a891d4de6)
### 大数据渲染优化（虚拟滚动）



## 数据结构
### 堆Heap
### 栈Stack
### 队列Queue
### 树Tree
### 数组Array
### 链表Linked List
### 几何Set
### 哈希表Map
### 图Graph

## 算法
### 动态规划
### 双指针
### 快慢指针
### 二分法
### 滑动窗口
### BFS
### 递归
### DFS
### 栈
### 回溯
### [排序算法](https://www.cnblogs.com/onepixel/articles/7674659.html)
#### 交换排序
##### 冒泡
##### 快速
#### 选择排序
#### 简单选择排序
#### 堆排序
#### 插入排序
#### 简单插入排序
#### 希尔排序
#### 归并排序
##### 二路归并排序
##### 多路归并排序

## 前端框架
### Vue（Vue3）
#### Vuex
#### Vue-Router
#### Vue-SSR
#### Vue-Loader
#### 核心概念
##### [数据绑定原理](https://juejin.cn/post/6844903869730799629)
##### watch原理
##### nextTick原理
##### keep alive
##### 检查数组变化
##### vue采用异步更新的原因
##### 模板编译原理
##### vue diff算法原理
##### v-[for中为什么要用key](https://juejin.cn/post/7054473358070513701)
##### [vue组件中data为什么必须是函数](https://juejin.cn/post/7055310157063913480)
##### Vue.$set原理
##### 父子组件生命周期调用顺序
##### 双向绑定与vuex是否冲突
##### [组件通信方法](https://juejin.cn/post/7054576471758602247)
##### 虚拟DOM
###### 为什么需要Virtual DOM
###### Virtual DOM Tree的创建
###### Virtual DOM 的更新
###### Virtual DOM 的diff
###### Virtual DOM的优化


### React
#### Redux
#### React Hooks
#### React-Router
#### React-SSR
#### 核心概念
##### 单向数据流
##### diff算法

#### 衍生
##### umi
##### next.js
##### dva
##### mobx

### Angular

## 编程思想 
### [常用设计模式](https://refactoringguru.cn/design-patterns/catalog)

#### 创建型模式
##### 工厂方法
##### 抽象工厂
##### 生成器
##### 原型
##### 单例

#### 结构型模式
##### 适配器
##### 桥接
##### 组合
##### 装饰
##### 外观
##### 代理
##### 行为模式
##### 中介者
##### 观察者
##### 访问者
##### 策略
##### 命令

### 设计原则
#### 单依职责原则
#### 开闭原则
#### 里式替代原则
#### 依赖倒置原则
#### 接口隔离原则
#### 最小知晓原则

### 编程范式
#### OOP
#### AOP
#### 函数式编程

## 安全
### xss攻击
### 同源策略与跨域方案
#### [跨域cookie获取](https://juejin.cn/post/7054034108531359774)
### CSRF攻击

## 工程化
### 代码版本管理
#### git
### 依赖管理
#### npm
#### yarn
#### pnpm

### babel原理

### 打包构建工具
#### webpack

##### [loader机制](https://github.com/youngwind/blog/issues/101)
##### [webpack工作流程](https://developer.aliyun.com/article/61047)
##### [webpack插件](https://juejin.cn/post/6844903789804126222)
##### webpack热更新原理

#### rollup
#### gulp

### 容器化
#### docker

### 代码质量
#### eslint
#### tslint

### [tree shaking](https://juejin.cn/post/6844903544756109319)



## 操作系统
### 进程与线程
### 进程通信
### 进程调度策略
### 死锁
### IO多路复用
### 文件系统


## 后端基础
### NodeJs
#### 常用框架
##### Express
##### Meteor
##### NestJS
##### Koa\Koa2
##### sails
##### Egg
##### fastify
##### Loopback
##### hapi
##### polemo


#### httpServer
##### 反向代理
##### 静态服务
##### BFF
##### SSR
##### RPC调用

#### 性能
##### QPS
##### 压测
##### 吞吐量

#### 模块机制
#### require原理
#### 进程通信
#### 守护进程
#### 异常处理
#### 事件循环
#### 文件操作

### 静态服务器
#### Nginx

### 其他编程语言
#### Java
#### Go
#### Python
#### PHP
#### .NET

### 数据库
#### SQL
##### MySql
##### Oracle

#### NoSQL
##### MongoDB
##### redis


## 设计架构
### MVVM
### MVC
### MVP
### API
#### RESTFul
#### GraphQL

### 鉴权方案
#### session-cookie
#### Tooken验证
#### OAUTH[开放授权]
#### HTTP Basic Authentication

## [自动化测试](https://zhuanlan.zhihu.com/p/46499606)
### 单元测试
#### jest
#### mocha
#### jasmine

### 集成测试
#### [puppeteer](http://www.puppeteerjs.com/)
#### Karma
#### Mocha
#### PhantomJS
# qiqnkun 原理深入

## 简介

qiankun 孵化自蚂蚁金融科技基于微前端架构的云产品统一接入平台， 是一个基于 single-spa 的微前端实现库，旨在帮助大家能更简单、无痛的构建一个生产可用微前端架构系统。

## 特性
📦 基于 single-spa 封装，提供了更加开箱即用的 API。
📱 技术栈无关，任意技术栈的应用均可 使用/接入。
💪 HTML Entry 接入方式，让你接入微应用像使用 iframe 一样简单。
🛡​ 样式隔离，确保微应用之间样式互相不干扰。
🧳 JS 沙箱，确保微应用之间 全局变量/事件 不冲突。
⚡️ 资源预加载，在浏览器空闲时间预加载未打开的微应用资源，加速微应用打开速度。
🔌 umi 插件，提供了 @umijs/plugin-qiankun 供 umi 应用一键切换成微前端架构系统。

## qiankun如何配合single-spa
qiankun最主要的两个函数：

RegisterMicroApps:
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/063f759c151f4790834786ab5414b0df~tplv-k3u1fbpfcp-watermark.image?)
> RegisterMicroApps主要功能：处理自定义配置、判断是否重复注册、注入app函数、registerApplication();
start:
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f094f965a4504b1bbe0bab8b1d834a5a~tplv-k3u1fbpfcp-watermark.image?)
> start的主要功能包括：处理自定义配置、是否prefetch、自动对低版本浏览器降级沙箱、start();

## app函数主要执行流程
app函数最核心的部分就是 `loadApp()`函数。
1. 获取入口的html，脚本执行函数和静态资源路径。
2. 处理template并包裹统一外壳。（`import-template-entry`）
3. 渲染到指定的容器。
4. 创建沙箱容器并使用沙箱。
5. 执行脚本并获取钩子函数 `bootstrapt、mount、update、unmount`。
6. 钩子函数提供给`single-spa`调用。

## import-template-entry

`import-template-entry` 主要功能是将原始的html文档转换成供qiankun渲染的组件文档。

主要做了以下三件事：
1.  拉取 url 对应的 html 并且对 html 进行了一系列的处理
    - 去掉注释
    - 注释所有的外联 js 以及删除掉所有的页级 js (当然都收集起来了)
    - 注释所有的外联 css，保留页级 css
1.  拉取上述 html 中所有的外联 css 并将其包裹在 `style` 标签中然后嵌入到上述的 html 中
1.  支持执行页级 js 脚本 以及 拉取上述 html 中所有的外联 js 并支持执行


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b466e7e235ff46b9aeab791c73755823~tplv-k3u1fbpfcp-watermark.image?)

左侧是执行之后，右侧是执行之前的代码。

想对`import-template-entry`更深入了解可[ 看看这篇 ](https://blog.csdn.net/qq_41800366/article/details/122093720)。

## qiankun沙箱机制

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7880b5d9cd9144cc9f41d966a9de23d5~tplv-k3u1fbpfcp-watermark.image?)
实现原理：使用ES6的Proxy来代理window。
>激活沙箱后，每次对window取值的时候，先从自己沙箱环境的fakeWindow里面找，如果不存在，就从rawWindow(外部的window)里去找；当对沙箱内部的window对象赋值的时候，会直接操fakeWindow，而不会影响rawWindow。


## css样式隔离
### 子应用之间的样式隔离
**Dynamic Stylesheet动态样式表，当应用切换时移除旧的应用样式，添加新应用样式。**
> 方案其实很简单，我们只需要在应用切出/卸载后，同时卸载掉其样式表即可。 原理是浏览器会对所有的样式表的插入、移除做整个 CSSOM 的重构，从而达到 插入、卸载 样式的目的。这样即能保证，在一个时间点里，只有一个应用的样式表是生效的。
### 主子应用的样式隔离
1.   BEM(Block Element Modifier) 约定项目前缀（只是约定）
2.   CSS-Modules 打包时生成不冲突的选择器名
3.   Shadow DOM 真正意义上的隔离

如果还不了解 shadow dom 可以看看[这篇](https://www.cnblogs.com/coco1s/p/5711795.html)。


## 更多

关于 qiankun 的 prefetch 属性和 prefetchApps列表的关系是怎么样的？带我后续研究透彻之后再来补充。


[返回首页](/)














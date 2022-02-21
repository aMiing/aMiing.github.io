# 交叉监视器IntersectionObserver实践和问题总结


> 点赞再看，喜乐相伴。`收藏==学会`

## 前言

> 监测目标元素是否在可视区域内，应该是toC网站经常会遇到的需求，我们以前的实现方式之一是通过js判断目标元素距离视口的头部距离是否小于视口的高度，如果是则说明完成一次曝光（即被用户看到）。

从实现的层面来讲，确实没什么问题。

但如果是从性能上分析，频繁的获取元素的**几何属性**信息，每次的获取操作都会触发页面的**重排和重绘**，对重排和重绘不熟悉的同学可以参考下[这篇笔记](https://link.juejin.cn?target=https%3A%2F%2Fapp.yinxiang.com%2Ffx%2F0c175b9a-e432-46c4-9b7a-d38878698791 "https://app.yinxiang.com/fx/0c175b9a-e432-46c4-9b7a-d38878698791")。而重排和重绘对性能的开销是昂贵的，特别是在页面元素多，布局相对复杂的场景下。

所以，既然是高频需求，ECMA自然会关注到并会有相应的解决方案推出。没错，就是题目中提到的交叉监视器`IntersectionObserver`。

## `IntersectionObserver`介绍

> 温馨提示：如果有同学对这个API已经有一定的了解了，建议可以跳过这一部分的基础介绍，直接去往第三部分：[**问题与现状**]

1.  定义

[IntersectionObserver API](https://link.juejin.cn?target=https%3A%2F%2Fwicg.github.io%2FIntersectionObserver%2F "https://wicg.github.io/IntersectionObserver/")，可以自动"观察"元素是否可见，Chrome 51+ 已经支持。由于可见（visible）的本质是，目标元素与视口产生一个交叉区，所以这个 API 叫做"交叉观察器"。

2. 使用

```
const io = new IntersectionObserver(callback, option);
// 开始观察, 监视多个dom，就写多次
io.observe(document.getElementById('example'));
// 停止观察
io.unobserve(element);
// 关闭观察器
io.disconnect();
```

3.  callback

```
const io = new IntersectionObserver(
  entries => {
    console.log(entries);
  }
);
```

> `entries`是一个数组，每个成员都是一个[`IntersectionObserverEntry`](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FAPI%2FIntersectionObserverEntry "https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry")对象。举例来说，如果同时有两个被观察的对象的可见性发生变化，`entries`数组就会有两个成员。

4.  option

`root`: 所监听对象的具体祖先元素([`element`](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FElement "https://developer.mozilla.org/zh-CN/docs/Web/API/Element"))。如果未传入值或值为`null`，则默认使用顶级文档的视窗。

`rootMargin`: 计算交叉时添加到**根(root)** 边界盒[bounding box (en-US)](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FGlossary%2Fbounding_box "https://developer.mozilla.org/en-US/docs/Glossary/bounding_box")的矩形偏移量。所有的偏移量均可用**像素(pixel)** (`px`)或**百分比(percentage)** (`%`)来表达, 默认值为"0px 0px 0px 0px"。可以简单类比元素的margin属性。

`thresholds`：一个包含阈值的列表, 按升序排列, 列表中的每个阈值都是监听对象的交叉区域与边界区域的比率。[这个属性笔者也不是很懂，没用过]

这一部分主要参考 [MDN](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FIntersectionObserver "https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver")和[阮一峰大佬](https://link.juejin.cn?target=https%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2016%2F11%2Fintersectionobserver_api.html "https://www.ruanyifeng.com/blog/2016/11/intersectionobserver_api.html")的API介绍部分。

## 问题与现状

最近迁移文档有一个需求，就是当用户滚动页面的时候，右侧菜单要根据当前滚动到的位置，定位目录并增加active状态。相信很多博文，还有类似于带tabBar的详情页都有这样的功能。

掘金也有这个：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c004e8fcba3e4e3a87568e2314711ec1~tplv-k3u1fbpfcp-zoom-1.image)

很容易就能想到，其实就是监测标题是否在可见区域内，如果在说明滚动到了目录位置，给对应的目录增加激活状态。

有了上面的基础Api的学习，我们很快就能动手实践。写出监视标题并返回当前可见区域标题的代码。

```
function createIntersectionObserver() {
  const lis = markdown.value.$el.querySelectorAll('a.header-anchor');
  observe.value = new IntersectionObserver(observeCallback, {
    rootMargin: `-80px 0px -80px 0px`,
  });
  for (let i = 0; i < lis.length; i++) {
    const element = lis[i];
    observe.value.observe(element);
  }
}
复制代码
```

可以看到，我这里对文章内容区域所有的标题加了监视。rootMargin的上下为负值，是为了缩小监视区域。具体原因后面会讲到。

当监视元素穿过监视区域时会触发回调事件，回调逻辑如下：

```
function observeCallback(entries) {
  if (entries[0].intersectionRatio <= 0) return;
  const id = entries[0].target.attributes.href.value;
  toc.value.querySelectorAll('a.active').forEach(ele => {
    ele.removeAttribute('class');
  });
  toc.value.querySelector('[href="' + id + '"]')?.setAttribute('class', 'active');
}
复制代码
```

我这里这对返回的第一个元素进行处理（某一时刻多个元素都穿越了交叉区，一般是滑动比较快的情况下，其实处理几个关系不是很大，这里简单处理），`intersectionRatio`大于零才说明是在监视器的内部。后面的逻辑比较简单，就是取可是元素的id，然后找到目录区域的这个元素设置为激活状态，不再赘述。

这里其实有个问题，就是我在刚刚进入页面的时候，第一个标题甚至是前几个标题都在可见区域，滑动也不会触发交叉区的检测，导致目录区域无法给他们添加激活态，好像不是很合理。看一下饿了么Plus的文档：

![chrome-capture (2).gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/224cfeddfc94481383b164c29b935253~tplv-k3u1fbpfcp-zoom-1.image)

> 显然，这是我们应该达到的效果，就是在初始状态右侧无选中，第一个标题滑动到最上方的时候，目录的第一个标题就应该被激活。

我们不妨大胆的猜想一下，是否它的监视区域只有上面那一部分呢？如图：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ff2a04458604795822ed2f145500b20~tplv-k3u1fbpfcp-zoom-1.image)

我们按照这个思路来实现一下试试。

获取当前视口高度,调整监视范围

```
const height = window.innerHeight > 280 ? window.innerHeight : 280;
observe.value = new IntersectionObserver(observeCallback, {
// 尽量缩小交叉范围，高度限制在120px
rootMargin: `-80px 0px -${height - 80 - 120}px 0px`,
});
复制代码
```

把高度限制在头部的120px范围内，这样当监视对象划过监视区域就会触发目录的激活态。 我们来看一下效果：

![chrome-capture (3).gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/772df1341a7e43268a0bfbca434dbd8b~tplv-k3u1fbpfcp-zoom-1.image)

好像已经达到了预期的效果。

> 不要忘记在组件卸载（销毁）的时候，取消监视！

那么问题来了，当我们修改窗口大小的时候，要怎么保证我们的监视区域大小不变呢？？毕竟我们是用像素绝对者设置的监视器。

你可能会想到，监听 window的resize事件，回调修改 rootMargin 不就好了吗！没错，我也这么想的。不过遇到了问题。

## 问题

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f172b05075444b9884d750978608cbcf~tplv-k3u1fbpfcp-zoom-1.image) 很明显实例的这几个属性都是只读的，原型上只有get方法，官方并没有提供动态修改的方法给我们。

在google无果的情况下， 秉持着遇到问题先抛出来的原则，我去github仓库提了个[issue](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fw3c%2FIntersectionObserver%2Fissues%2F490 "https://github.com/w3c/IntersectionObserver/issues/490")，希望能引起官方的关注和回复。有兴趣的同学也可以去点个赞，助个力。

## 解决

我们不可能把希望寄托在官方修改api上，了解ECMA的提案流程的同学肯定知道，那将是个十分漫长的过程，所以眼下我们就应该找到切实的办法来解决这个问题。

我们最终通过重置IntersectionObserve实例解决了问题，可能不是很优雅。实现逻辑是这样的：

1.  监听resize事件，触发IntersectionObserve实例化

```
debounceResize.value = debounce(handleResize, 300);
window.addEventListener('resize', debounceResize.value);
function handleResize() {
  console.log('resize');
  createIntersectionObserver();
}
复制代码
```

2.  `createIntersectionObserver`判断如果实例化过，就取消检测，并清空实例化

```
observe.value && (observe.value.disconnect(), (observe.value = null));
复制代码
```

> 注意在卸载销毁组件时，移除resize监听事件，避免出现内存泄漏现象。`window.removeEventListener('resize', debounceResize.value);`

## 写在最后

> 关于作者：
>
> -   本人前端开发者一枚，向往自由职业。
> -   工作两年有余，最近在夯实基础，查漏补缺。
> -   日常开发中遇到的问题也会总结成文章，加深记忆，温故知新。
>
> 如果你也和我一样，对前端充满好奇，对技术充满热爱，不妨加个关注，大家一起学习成长。

相关文章：[ 利用IntersectionObserver 分分钟实现图片懒加载 ](/articles/IntersectionObserver实现图片懒加载实践.md)


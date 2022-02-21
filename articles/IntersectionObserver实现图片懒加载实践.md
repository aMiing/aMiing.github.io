# IntersectionObserver 实现图片懒加载实践

## 前言

现代浏览器 `Observer` 类接口共实现了四个不同类型的监视器（观察者）：

1. Intersection Observer，交叉监视器：异步检测目标元素与祖先元素或 viewport 相交情况的变化。
2. Mutation Observer，变动监视器：提供了监视对DOM树所做更改的能力，替代`Mutation Event`。
3. Resize Observer，视图监视器： 高性能监视元素大小的改变。（可以尝试替代window.resize来减轻性能开销）
4. Performance Observer，性能监视器：用于监测性能度量事件。

具体的解释和用法，大家可以去参考 [MDN上的解释](https://developer.mozilla.org/zh-CN/docs/Web/API)。

## 图片懒加载的实现逻辑

业界有很多的实现图片懒加载的方案，思路都大同小异。大体分为两种实现方案：
1. 脚本检测元素距离视口顶部的距离与视口大小作比较，判断是否在可视区域。
   可能会用到的API： `offsetTop`和`getBoundingClientRect`
2. 交叉监视器：IntersectionObserver，判断是否进入监视区，性能更佳。

我们这里仅以`IntersectionObserver`的方式实践，探究具体实现细节。

## 动手实践
### 获取图片资源
首先，我们需要获取图片资源，方便我们展示效果。这里有两个不错的开放接口可供我们选择：
1. <https://picsum.photos/v2/list?limit=50&page=1> 他可以随机生成图片，稳定高效，我推荐用它。
2. <https://44c63a76-69a2-41f9-b8c6-12b9af85bde9.bspapp.com/getWallpaper?cid=26&start=20> 360的壁纸接口，也可以用

### vite起一个vue项目
`npm create vite@latest`

我这里选择的是vue3框架。开箱即用，不需要修改配置。

在`app.vue`里面引入我们的组件 `LazyImages`
代码如下：
```vue
<script setup>
import { onMounted, ref, nextTick, onUnmounted } from "vue";

let imageData = ref([]);
async function getMoreData(page = 0) {
  const response = await fetch(
    "https://picsum.photos/v2/list?limit=50&page=" + page
  );
  let newData = await response.json();
  newData = newData.map((e) => {
    return {
      ...e,
      download_url:
        e.download_url.split("/").slice(0, -2).join("/") + "/300/200",
    };
  });
  imageData.value = [...imageData.value, ...newData];
}
let observer = ref(null);
const root = ref(null);
onMounted(async () => {
  await getMoreData();
  await nextTick();
  InitObserve();
});
// 初始化监视器
function InitObserve() {
  observer.value = new IntersectionObserver(handlerObserve, {
    root: root.value,
    rootMargin: "0px 0px 800px 0px", // 监视区向下拓展800px
  });
  addObserve();
}
function addObserve() {
  const list = document.querySelectorAll(".image-item-box");
  for (let i = 0; i < list.length; i++) {
    const element = list[i];
    observer.value.observe(element);
  }
}
// 处理监视器回调事件
function handlerObserve(entries) {
  entries.forEach(({ isIntersecting, target }) => {
    if (isIntersecting) {
      const targetImg = target.children[0];
      // console.log("target", target);
      targetImg.src = targetImg.dataset.src;
      // 修改过src属性之后，即可取消监视
      observer.value.unobserve(target);
    }
  });
}
// 关闭观察器
onUnmounted(() => {
  observer.value.disconnect();
});
</script>

<template>
  <div class="lazy-content" ref="root">
    <div class="image-item-box" v-for="item in imageData" :key="item.id">
      <!-- 最好能给src一张默认缺省图片 -->
      <img
        class="observeImg"
        src=""
        :data-src="item.download_url"
        alt="正在加载"
        :key="item.id"
        width="400"
      />
    </div>
    <!-- 拓展一下: 监视该区域出现在交叉区，向服务器请求更多数据 -->
    <div class="bottom-text">
      <!-- <span v-if="page < 50">正在为您加载更多...</span> -->
      <span>加载完毕~</span>
    </div>
  </div>
</template>

<style scoped>
.lazy-content {
  width: 100%;
  height: 100%;
  overflow-y: auto;
  /* display: flex;
  flex-direction: column;
  align-items: center; */
}
.image-item-box {
  width: 400px;
  height: 267px;
  background: #ccc;
  margin: 0 auto 1px;
}
</style>
```

核心三个步骤： 1 初始化监视器，传入监视区容器，和监视区域边距（rootMargin）； 2 修改过src属性之后，取消对该元素的监视； 3 组件卸载之前关闭监视器，避免内存泄漏。

## 效果展示
![lazy-images.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ed0576be158417f97286391ca95e1d1~tplv-k3u1fbpfcp-watermark.image?)
## 问题

当我想要让图片居中显示时，设置 `lazy-content` 为flex布局，发现此时懒加载不“懒”了！
初始进来就请求了所有的图片资源，去掉flex布局，懒加载又恢复了。


> 通过控制台我们发现是所有的元素都进入到了交叉监视区，导致所有图片请求。

原因竟然是`lazy-content`占据height: 100%, 所以`image-item-box`会以父容器为基础做布局，就是全部都在`lazy-content`中，虽然出现了`image-item-box`溢出。

我的理解是初始状态下，`lazy-content`会将所有的`image-item-box`填充到自己的内部，然后加载到`image-item-box`的大小之后，出现溢出现象，所以初始状态小都会触发`handlerObserve`事件，全部加载。

我再slow-3G下截取到了这张图验证了我的猜想。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3958e04171c547dab13f0286fc2b6412~tplv-k3u1fbpfcp-watermark.image?)

## 写在最后

以上是交叉监视器在图片懒加载上的实践，有不足之处欢迎评论区指正交流。

另一篇利用交叉监视器实现目录导航的文章看这里[交叉监视器IntersectionObserver实践和问题总结](/articles/IntersectionObserver1.md)





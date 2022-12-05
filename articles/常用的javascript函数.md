# 那些在开发中提高效率的代码片段

## 1. 随机数/随机 id

```js
function getUuid(len = 32) {
  const S4 = () => Math.random().toString(16).substring(2);
  let res = "";
  while (res.length < len) {
    res += S4().substring(0, len - res.length);
  }
  return res;
}
```

### 代码分析

- `Math.random().toString(16)` 获取随机数，转成 16 进制
- `substring(2)`前两位是 `0.`所以从第二位开始取
- `while (res.length < len)` 循环，直到取到符合长度要求的字符串
- `substring(0, len - res.length)`,这一步也很关键，特别是最后一次循环，确保不会多截取

## 2. 获取文本确切宽度

```js
function getActualWidthOfChars(text, options = {}) {
  const { size = 14, family = "Microsoft YaHei" } = options;
  const canvas = document.createElement("canvas");
  const ctx = canvas.getContext("2d");
  ctx.font = `${size}px ${family}`;
  const metrics = ctx.measureText(text);
  return Math.abs(metrics.actualBoundingBoxLeft) + Math.abs(metrics.actualBoundingBoxRight);
}
```

### 代码分析

将文本绘制到 canvas 上，再使用 canvas 的`measureText`api 获取物理宽度。具体可参考我的上海篇文章[面试官：你是如何获取文本宽度的？](https://juejin.cn/post/7091990279565082655)

## 3. 判断某个对象是否发生变化

1. 可以通过序列化的方式
   `JSON.stringify(objA) === JSON.stringify(_objA)`

   > 在页面未保存拦截的文章中有使用到这个方法，判断表单内容是否有修改未保存
   > 可以参考[如何实现页面跳转拦截？](https://juejin.cn/post/7069033418222207006#heading-0)

2. 更多情况下我们不需要判断整个对象是否变化，而是对象里面的某个属性

相似的逻辑：

```js
equalProps(newV, oldV, props) {
    if (!newV || !oldV || newV.length !== oldV.length) return false;
    return newV.every((e, i) => JSON.stringify(e[props]) === JSON.stringify(oldV[i][props]));
}
```

### 代码分析

1. 首先如果`newV, oldV`有一个为 null 或 undefined 或空，或者二者长度不一致，直接返回 false，`props`一定发生了变化
2. 循环遍历`newV, oldV`，只需要遍历一遍，通过`i`下标访问另一个，序列化方式比较是否相等
3. every 确保找到一个不满足判断条件之后，终止。（也可以选择用 some）

我这里有个实际的例子，帮助大家体会下：

```js
watch: {
  cardList: {
    handler: function (newV, oldV) {
      const equal = this.equalProps(newV, oldV, 'operators');
      // 如果operators发生了变化，重绘缩略图
      !equal && this.initImagesMap();
    },
    immediate: true,
    deep: true,
  }
}
```

功能描述： 监听`cardList`的变化更新（重绘）card 上的缩略图。

但是缩略图的数据存在于`operators`上，当我们修改`cardList`上的其他属性，因为是 deep，所以也会触发缩略图的重绘。
我们现在加了个收藏的属性，用户点击收藏图标，发现这个 list 的图片都重刷了一遍，体验极差。

用上`equalProps`方法之后，避免了其他属性变化对缩略图的影响。

> 还有其他好办法吗？？ 我本寄希望于 Lodash,文档翻了一遍没找到对应的方法。 大家如果有更好的方式，可以在评论区交流。

## 4. 数组排序

`[4,3,9,5,2].sort(), ['b', 'a', 'c'].sort()`

数字根据大小排序，字符串根据 unicode 码排序。 这些最基础的用法相信大家都比较熟悉了，不做赘述。

根据 MDN 的描述，其实 sort 是接收一个回调函数`(a, b) => {}`,
**回调返回值 等于 0: 顺序不变； 小于零：a 在前 b 在后， 大于 0：b 在前 a 在后；。**

> 【这里可以记住 sort()不传参数是从小到大排列的，帮助记忆】

`[4,3,9,5,2].sort((a,b) => a - b)`, `a, b`分别代表迭代的前一项和后一项。

基于以上，如何实现下面的需求：
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecd050043f1f46b18f90c76ae802d655~tplv-k3u1fbpfcp-watermark.image?)
这样的一个组件，需要向后端传递一个数组，当前展示的属性在前，折叠在下面的属性在后，并且每一项上包含该属性是正序还是逆序排列。
数据结构长这样：

```js
const ORDERLIST = [
  {
    label: "热度",
    value: "HEAT",
    order: "DESC",
  },
  {
    label: "更新时间",
    value: "UPDATE_TIME",
    order: "DESC",
  },
];
```

很容易想到，把当前展示的属性移动到上面就可以了。

具体要怎么做呢？ 从两项中查找并删除，然后再插入到数组前面？ 能实现但是不太优雅~

我们用 sort 来是实现下：
`ORDERLIST.map(e => ...).sort((a, b) => a.value === currentProp ? -1 : 1)`

当当前项等于目标值时，返回 -1， 小于零， a 会排在 b 的前面。

> 在每次两两比较的时候，目标对象都会排在另一个前面，也就意味着它一定会排在数组的第一个。（可以想下冒泡排序的原理）

如果数组存在很多项同样使用此方法。

比如：
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/476a32e7003b48668d8cb335449ef415~tplv-k3u1fbpfcp-watermark.image?)

我们在下拉项目列表里面将当前项目提到第一个，刚好就可以用上面的排序方式！

## 5. 巧用 reduce 减少代码量

首先我们有这样的一份数据，从中筛选出包含 id 的菜单项组合成数组

```js
const modules = {
  modulesA: {
    name: "modelA_name",
    menus: [
      {
        id: "menu-A1",
        name: "菜单项A-1",
      },
      {
        name: "菜单项A-2",
      },
    ],
  },
  modulesB: {
    name: "modelB_name",
    menus: [
      {
        id: "menu-B1",
        name: "菜单项B-1",
      },
    ],
  },
};
```

同事的原始代码：

```js
function allTasks() {
  const _modules = Object.keys(modules).filter(
    key => modules[key].menus && modules[key].menus.some(i => i.id)
  );
  let tasks = [];
  _modules.forEach(key => {
    tasks = tasks.concat(
      modules[key].menus
        .filter(e => !!e.id)
        .map(e => {
          return {
            ...e,
            moduleCode: key,
          };
        })
    );
  });
  return tasks;
}
```

reduce 优化后的代码：

```js
function allTasks() {
  return Object.keys(modules).reduce((res, key) => {
    let item = [];
    if (modules[key].menus && modules[key].menus.some(i => i.id)) {
      item.push(
        ...modules[key].menus
          .filter(e => !!e.id)
          .map(e => {
            return {
              ...e,
              moduleCode: key,
            };
          })
      );
    }
    return res.concat(item);
  }, []);
}
```

我们成功从两次遍历缩减为了一次遍历，具体的细节就不展开了，仔细看下，还是比较好理解的。

## 总结

以上就是笔者京城在项目中经常实用的一些 javascript 小技巧，掌握这些技巧，确实能够在一定程度上提高开发效率。

更文不易，如果对你有帮助还请点个赞支持下，帮助笔者输出更多优质文章~

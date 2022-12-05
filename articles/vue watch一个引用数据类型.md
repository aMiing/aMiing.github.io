# Vue watch 监视一个引用类型的数据带来的问题？

## 现象

watch 到的 oldVal 和 newVal 是同一个应用对象，完全相等。

## 背景说明

最近在重构以前开发的一套数据可视化系统，在做 redo 和 undo 功能的时候，我的设计方案是这样的：

> 1. watch 监听 数据 的变化
> 2. `json-diff`比较数据的变化 <https://github.com/andreyvit/json-diff> 备选<https://github.com/benjamine/jsondiffpatch>
> 3. 存储并维护快照数组 <https://juejin.cn/post/6990296167468761095>

当我满怀希望地去监听数据变化的时候，发现 diff 出来的数据都是空数据。 纳尼？？（黑人问号.gif）

```js
watch: {
  screen: {
    handler(newV, oldV) {
      console.log(oldVal === trueVal); //结果为true
      this.recordScreenChange({ oldV, newV });
    },
    deep: true,
  }
}
```

`recordScreenChange`会执行 diff 操作，并把 diff 数据存入历史记录队列。

```js
recordScreenChange({ commit, dispatch, state }, value) {
  const { oldV, newV } = value;
  dispatch(
    "history/addHistory",
    { type: "screen", id: UUID(), diff: diff(oldV, newV) },
    { root: true }
  );
}
```

打印`oldVal === trueVal`结果为`true`！
是不是直接懵掉了，什么情况。不过冷静下来还是很容易想明白的。

## 原因

- 引用数据类型变量指向的是一块内存空间。watch 被触发的时候说明数据已经发生了变化，不管是 oldVal 还是 newVal 都是指向同一个内存空间的，所以 watch 到的 oldVal 和 newVal 完全是同一个对象，完全相等。
- 所谓的 oldVal 已经被更新了，所有并不能获取到旧的对象数据。

这样不行啊，我们希望拿到新旧数据，两者进行 diff 来着~

## 解决方案

> 既然出现问题的原因是新旧数据都指向了同一个内存地址，那么我们是否可以深拷贝一份数据，使他们指向两个不同的存储空间呢？

1. 方案一：

初始化时克隆一份数据 `oldScreen`，每次 watch 到变化，先完成新数据与 oldScreen 的 diff， 然后更新 oldScreen;

2. 方案二：
   computedScreen 克隆一份数据，我们不直接监视 screen， 而是监听 computedScreen
   ```js
    computedScreen() {
      return cloneDeep(this.screen);
    }
   ```
   这样我们在 watch 里面拿到的 oldVal 和 newVal 就是指向两份不同内存空间的数据了~ 可以开心的 diff 了！

好了，可以洗洗睡了~

## 总结

> Watch 只是侦听到数据改变了，并不会把之前的老数据给缓存下来，如果是引用类型就拿不到变化之前的数据了。 可以借助 computed 具有缓存的特性，拿到 oldVal.

# vue 中组件传值问题一文彻底搞懂
> 组件之间状态的传递是非常高频的需求，我们通常能想到的就是 props、$emit 这一对，进行上传下达的操作，在父与子之间的传递尚且能够应付，但是如果是三层或者更多层之间的传递，又牵涉很多参数的情况下，我们应该怎么处理呢？
> 本文将枚举项目中的各种场景，旨在全部覆盖 vue 中的状态传递，从此再也不怕面试官问 vue 传值的问题了！

## 父子组件传值

props+$emit()

```js
// 父组件
<my-component :disabled="true" @changeState="changeState"></my-component>
export default {
    methods: {
        changeState(state) {
            console.log(state)
        }
    }
}

```

```js
// my-component.vue
export default {
    props: {
        disabled: {
            type: Boolean,
            default: false,
        },
    },
    methods: {
        changeState() {
            this.$emit("changeState", false);
        },
    },
};
```
小结： 父组件通过v-on传递属性给子组件，子组件通过props接收父组件传递过来的属性。***子组件不能直接修改props的属性值***，可以通过`this.$emit()`,触发父组件绑定的方法，并将数据传递给方法，达到向父组件传值的目的。
> 这里涉及到子组件修改父组件props传递下来数据的知识点，后面会单独开个篇幅来细讲，有兴趣可关注或者自行search。

## 祖先组件向后代组件传值
+ 就像开头提到的那样，如果我们的组件层次结构是这样的:
![components.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8bed2a208c244ffb2a09db721c68b07~tplv-k3u1fbpfcp-watermark.image?)
+ 此时组件D需要用到组件A中的数据，难道我们要通过props下传到C，再从C通过props传到D？好像也可以就是麻烦了点，但是如果是下面这种情况呢？

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb7d4875d03a4d929a9af3246d2d27a3~tplv-k3u1fbpfcp-watermark.image?)
+ 啊这？！ 在比较复杂的中后台项目里面5、6层的组件封装也不足为奇，你总不会说依然通过这种方式传递吧？
+ 这种场景下，我们可以请出我们的*事件总线*来帮忙解决。
具体实现方式：

1. 注册事件总线
在根节点给 `Vue` 挂载一个空的 `Vue` 对象
```js
Vue.prototype.bus = new Vue();
```
2. 发射（上传）事件
```js
// F组件
this.bus.$emit('updateAttribute', true)
```
3. 触发（接收）事件
```js
// A组件
this.bus.$on('updateAttribute', (value) => {
    //do Something about value
})
```
这样一个流程下来就完成了数据从F组件向A组件的传递。我们不难发现整个传递过程都是通过对事件的监听和触发来实现的，所以叫时间总线，很好理解也很好记吧。

> 其实很多复杂场景下我们可以借助vue状态管理工具`vuex`这个神器来达到事半功倍的效果。也是复杂项目的必背和首选。

vuex通过维护 `state`(相当于组件里的`data`),所有组件都可以访问这个state，也都可以修改这个状态库，避免了数据在多层组件之间传来传去的繁琐。

比较复杂的场景下，我们可能需要维护多个`state`,vuex也给我们提供了module来应对复杂场景下的状态管理。每个module可以有自己独立的命名空间。
> 比如我在做收银系统的时候，收银模块、会员模块、商品库存模块等，糅合在一个状态Map里面会非常糟糕，想想这么多状态变量光命名都是个令人头大的问题。

关于vuex这里就不深入展开了，有需要可以异步[官方文档](https://vuex.vuejs.org/zh/guide/modules.html#%E6%A8%A1%E5%9D%97%E7%9A%84%E5%B1%80%E9%83%A8%E7%8A%B6%E6%80%81)。

## 组件属性透传 
当第三方组件不能完全满足我们的需要时，我们需要对其进行二次封装，此时就会面临一个问题，我们要怎么把参数传递下去呢？
> 分析：第三方组件假如有6个参数，即使我们没有扩展参数，我们封装后的组件也应该至少能接受这6个参数，那么我们要怎么将拿到的参数传给第三方组件呢？总不能一个个接收下来再一个个传下去吧？那如果是16个？26个呢？

显然一个个传递是不现实的，此时我们需要借助`$attrs`和`$listeners`来帮忙了。

$attrs的官方解释:
> 包含了父作用域中不作为 prop 被识别 (且获取) 的 attribute 绑定 (`class` 和 `style` 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (`class` 和 `style` 除外)，并且可以通过 `v-bind="$attrs"` 传入内部组件——在创建高级别的组件时非常有用。

$listeners的官方解释：
> 包含了父作用域中的 (不含 `.native` 修饰器的) `v-on` 事件监听器。它可以通过 `v-on="$listeners"` 传入内部组件——在创建更高层次的组件时非常有用。

如果我们在封装过程中没有用到这些参数和事件，不需要在props里面接收，只要在组件上加上`v-bind="$attrs" v-on="$listeners"`就能直接传递下去了，还是很方便的有没有。

目前整理的就这么多，如果还有其他好的方式，也欢迎在评论区讨论补充。
如果有帮助到你，不妨点个赞鼓励我一下吧！🤡
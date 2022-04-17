# 为什么 vue 组件里 data 必须是个函数？

> 你是否有这样的疑问：为什么在 vue 的 hello world 里面 data 是个对象，那为什么在组件里面就一定要写成一个函数呢？

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90649c2fba374427b552e458e8823bb9~tplv-k3u1fbpfcp-watermark.image?)

> 带着这样的疑问，我们来一探究竟：为何组件的 data 一定得是函数结构？

## 组件的复用性

我们都知道复用性是 vue 组件存在的必要性之一，当我们复用组件时候，我们可以把组件理解成是一个对象（或者类）`class Component`,当我们调用组件时候，就相当于调用`new Component()`。

当我们生成两个组件实例：

```js
// component.vue
export default {
    data: {
        name: "superman",
        age: 18,
    },
};
// componentA.vue
import component from "./component.vue";
const data = component.data;
data.name = "前端切图仔";
console.log(data);
// { name: '前端切图仔', age: 18 }

// componentB.vue
import component from "./component.vue";
console.log(component);
// { name: '前端切图仔', age: 18 }
```

> vue 项目中使用了 ESM 的导出语法，ESM 导出的是对象的引用，也就是说我们在两个组件里面在修改同一个对象
> 不难看出此时`componentA`和`componentB`的 data 都指向同一个对象，导致我们的组件数据相互污染。

如果我们想要组件之间的 data 能够相互独立，就需要维护两个 data 对象。
当我们组件以函数形式返回 data 时，上面的 case 将变为这样：

```js
export default {
    data() {
        return {
            name: "superman",
            age: 18,
        };
    },
};
// componentA.vue
import component from "./component.vue";
const data = component.data();
data.name = "前端切图仔";
console.log(data);
// { name: '前端切图仔', age: 18 }

// componentB.vue
import component from "./component.vue";
const data = component.data();
console.log(data);
// { name: 'superman', age: 18 }
```

这样就能在组件 A 和组件 B 维护两个独立的 data 互不干扰。

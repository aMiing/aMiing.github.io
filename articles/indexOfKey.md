# 为什么不建议用index作为v-for的key？
> vue中循环数组对象不建议用index作为v-for的key，可能初学者也都知道这个事实，但是具体是什么原因，会造成什么样的后果呢？

我们今天就从其原理:虚拟dom的differ算法`vdom-differ`说起。

当我们修改了数据，数据的变动会触发订阅该数据变化的dom对象的更新，这个过程大致可分为三个阶段：
1. 首选是会重新构建虚拟dom树
2. 然后对比新旧虚拟dom树的节点,这里使用的就是大名鼎鼎的 `differ` 算法，也是这里要展开讲述的核心
3. 根据对比结果，删除、添加、更新真实的dom节点。而非全部更新替换，这也就是虚拟dom的优势所在。

### 我们不妨通过几个用例来说明
用例1：
> 我们向要遍历展示的数组`List`的最前面添加添加一个数据项， `List.unshift('test')`

此时我们希望在dom中的表现是在dom 中 li 的最前面添加一个li元素，innerHtml是 test， 而原来的dom li 不发生变化。

当我们使用index作为key会发生什么呢？
![list1](/source/images/list1.png)
请记住我们此时展开的li是第二个，innerHtml是` index: 1, item: 222 `;

然后我们点击新增按钮，在最前面添加一项；

![list2](/source/images/list2.png)

我们不难发现此时的第二个li 里面的内容更新了， 原本在第二个li里面的元素被移动到了第三个li

不仅如此，我们在点击的时候还看到除了第一个li是新增的，其他的li都发生了更新。

这显然违背了我们的意愿。

导致这个问题的根本原因也就发生在上面所提到的虚拟dom比较的这一步：

由于我们使用index作为Li的key，那么在differ比较的时候，index随着插入数据发生了变化，原来第二个li的index从 1 变成了 2 ，会被认为dom发生了变化，所以执行了更新过程。造成了不必要的性能开销，我们知道dom更新是特别消耗浏览器性能的，特别是如果for里面dom比较复杂，多层嵌套的情况下，对性能的开销还是不容小觑的。

那么，如何避免上述问题呢？
答案已经呼之欲出了，index变化导致了上述问题，那就不要用index作为key不就好了。
我们验证一下这个猜想。
```js
 <ul>
    <li v-for="(item, index) in list" :key="item">
      index: {{ index }}, item: {{ item }}
      <button @click="deleteItem(index)">删除</button>
    </li>
  </ul>

```
我们这里使用`item`自身作为key，前提是保证item不会重复，我们使用`list.unshift("amingxiansen" + Math.random().toString(16).slice(2));`
来保证新添加数据不重复。
我们来看下效果：
![list3](/source/images/list3.png)
还是一样的套路，我们先展开第二个dom li 

点击添加
![list4](/source/images/list4.png)

然后我们发现添加了一个Li在最上面，而当前展开的li内容并没有发生变化，说明当前li和其后的li都未发生更新。仅仅是插入了一个li。


::tip
在写demo的时候，刚开始有个小误区，就是我再dom li 里面写了index展示在页面上，这样不管我的key是否发生变化，都会导致dom li内容的更新，原因也就是index变化了，内部dom元素也会更新。注意 没有标签的元素也是dom元素的一种。 

所以，你弄明白了为什么不建议使用index作为key了吧？



[返回首页](/)


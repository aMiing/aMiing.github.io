# 数组去重策略

数组去重应该前端比较常见的题目了，今天总结一下都有哪些常用和不常用的去重方法，更多解法欢迎jy在评论区补充~

## 常规问题

假如有这么个数组`arr = ['shanghai','beijing', 'hangzhou', 'guangzhou','nanjing','beijing','shanghai']`,如何去重？
### 方法一
```js
let arr = ['shanghai','beijing', 'hangzhou', 'guangzhou','nanjing','beijing','shanghai'];
function unrepeated(arr) {
  const res = [];
  arr.forEach(e => {
    !res.includes(e) && res.push(e);
  })
  return res;
}
unrepeated(arr);
```
时间复杂度O（n2）,空间占用O（n）；

### 方法二
不应额外的空间存储，直接在原数组上面操作：
```js
let arr = ['shanghai','beijing', 'hangzhou', 'guangzhou','nanjing','beijing','shanghai'];
function unrepeated(arr) {
  return arr.filter((e,i) => arr.indexOf(e) === i);
}
unrepeated(arr);
```
时间复杂度O（n2）,空间占用O（1）；

### 方法三
用对象存储，并在对象中直接通过key查找。key存储，值置为null就行。
```js
let arr = ['shanghai','beijing', 'hangzhou', 'guangzhou','nanjing','beijing','shanghai'];
function unrepeated(arr) {
  const res = {};
  arr.forEach(e => {
    !res[e] && (res[e]=null);
  })
  return Object.keys(res);
}
unrepeated(arr);
```
时间复杂度O（n）,空间占用O（n）；

### 方法四
和方法三类似，存储方式换成Map,Map检索的性能更好；
```js
let arr = ['shanghai','beijing', 'hangzhou', 'guangzhou','nanjing','beijing','shanghai'];
function unrepeated(arr) {
  const res = new Map();
  arr.forEach(e => {
    !res.has(e) && res.set(e);
    // 不加判断，直接覆盖也行
    // res.set(e);
  })
  return Array.from(res.keys());
}
unrepeated(arr);
```
时间复杂度O（n）,空间占用O（n）；


### 方法五
> Set对象是值的集合，你可以按照插入的顺序迭代它的元素。Set 中的元素只会出现一次，即 Set 中的元素是唯一的。

所以也就是说，我们可以直接通过Set去重！
```js
let arr = ['shanghai','beijing', 'hangzhou', 'guangzhou','nanjing','beijing','shanghai'];
function unrepeated(arr) {
  return [...new Set(arr)]
}
unrepeated(arr);
```

## 问题升级

假如是个比较特殊的数组，包含不同的数据类型，而且有引用数据类型。

`let arr = [123, '123', [], {}, {}, null, '', null, undefined, new Date(), new RegExp(),false];`
> 说明： 虽然 {} 和 {} 指向不同的内存的地址，但是都表示空对象，也需要去重。 即**去除含义重复的数据**。


- 分析

  乍一想，能不能通过序列化的方式，把数据都转成字符串再借用上面的任一方法去重呢？
- 好像可以，又好像不行···

  数字123和字符串123，肯定会被去除，但是这显然是违背初衷的
  转成字符串之后，最后还能不能恢复回来呢？

ok, 问题明确，我们逐个解决！

1. 不同类型的数据，强转字符串之后，不能区分原始类型 怎么办？
   > 带上类型去重，比如 `123_string`和`123_number`显然不会被判重复去除

2. 转成字符串之后，最后还能不能恢复回来呢？
  > 记录类型，最后恢复。但是不能简单将Date和[]都记为Object

### 代码实践
借用方法四：
```js
let arr = [123, '123', [], {}, {}, null, '',null, undefined, new Date(), new RegExp(), false];
// 数据类型判断的通用方法
function getType(variable) {
  return Object.prototype.toString.call(variable).replace(/\[|\]|object /g, "");
}

function unrepeated(arr) {
  const res = new Map();
  arr.forEach(e => {
    !res.has(JSON.stringify(e)+'_'+getType(e)) && res.set(e+'_'+getType(e), e);
  })
  return Array.from(res.values());
}
unrepeated(arr);

// 验证一下
unrepeated(arr).map(e => {
  return [e, getType(e)]
});

```

### 执行结果
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0a01e7b2db14158b1c67673c4485a89~tplv-k3u1fbpfcp-watermark.image?)
### 验证
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e9a9066a8594b8e8f6066080c3ffa9a~tplv-k3u1fbpfcp-watermark.image?)

可以看到最后结果拿到的数据类型也是对的~


### 缺陷
当数组中包含Symbol类型时，会报错

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d11752f51a7f4e789a897736d0c3111e~tplv-k3u1fbpfcp-watermark.image?)

- Symbol类型不能通过JSON.stringify处理

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/036ca823c3c44aa890f248bb432a4431~tplv-k3u1fbpfcp-watermark.image?)

打印，发现Symbol原型上有toString方法，那么它不仅能够通过`Object.prototype.toString().call()`获取其类型，也能通过toString转成string；

### 进一步完善
最终版代码如下：

```js
let arr = [123, '123', [], {}, {}, null, '',null, undefined, new Date(), new RegExp(), false, Symbol()];
// 数据类型判断的通用方法
function getType(variable) {
  return Object.prototype.toString.call(variable).replace(/\[|\]|object /g, "");
}
// 拼接key
function getKey(e) {
  let res;
  try{
    res += JSON.stringify(e)
  }
  catch(e){
    // Symbol类型
    res += e.toString();
  }
  return res + '_'+getType(e)
}

function unrepeated(arr) {
  const res = new Map();
  arr.forEach(e => {
    const key = getKey(e);
    !res.has(key) && res.set(key, e);
  })
  return Array.from(res.values());
}
unrepeated(arr);

```
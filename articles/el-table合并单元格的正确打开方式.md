# el-table合并单元格的正确打开方式

## 前言
elementUi官方给出的合并单元格的例子非常简单，我们通常的使用场景并不能静态的指定合并第几行到第几行，或者第几列到第几列。

通常我们从接口拿到一组数据，数据中的某一列可能会出现大量重复，这样产品就会要求说按照这个字段分组聚合，然后产品闭着眼都会想到说

”**这么多重复展示的表格项为啥不合并下单元格呢？**“

这个时候你总不能偷懒说”实现不了“吧，否则产品又会打开elementUI的官网给你battel一番了。

## 基础使用方式

其实官方给出的api已经足够清晰，
```js
spanMethod({ row, column, rowIndex, columnIndex }) { 
    if (rowIndex % 2 === 0) { 
        if (columnIndex === 0) { return [1, 2]; } 
        else if (columnIndex === 1) { return [0, 0]; } 
     } 
}
```
spanMethod会在渲染单元格的时候被遍历，参数可以使用行、列数据和行列的下标。
返回的数据是 `[行宽, 列宽]`,也可以对象方式返回： `{rowspan: 1, colspan: 1}`。

## 业务中使用
> 需求其实不复杂，只是实现起来有点麻烦

想说实现思路：
1. 将数据按照某个字段分组，得到分组的数据
2. 分组数据再进行扁平化处理

具体的实现代码如下，我加了详细的注释：

1. 数据分组
```js
groupedByTargetField(data, target) {
   // 获取全部的目标字段的值数组
  const modules = data.map(e => e.target);
  // 数组去重
  const uni_modules = [...new Set(modules)];
  // 遍历去重后的数组，返回分组的数据
  return uni_modules.map(item => {
      // 按照target字段进行分组
    const member = data.filter(i => i.target === item);
    return {
      module,
      member,
      count: member.length || 1,
    };
  });
}
```

2. 扁平化数据

```js
transFormatData() {
  const groupedData = this.groupedByTargetField(this.originData);
  // 返回扁平化数组
  return groupedData.reduce((res, cur) => {
      // 给每一组的第一项添加span字段，cur.count是需要合并的数据项长度
    cur.member[0].span = [cur.count, 1];
    return res.concat(cur.member);
  }, []);
}
```

3. 使用

```html
<el-table :data="transFormatData" border :span-method="spanMethod">
        <el-table-column prop="target" label="模块" width="120"> </el-table-column>
        <el-table-column prop="type" label="类型" width="200"> </el-table-column>
        <el-table-column prop="data" label="资源" width="600"> </el-table-column> 
</el-table>
```
```js
spanMethod({ row, columnIndex }) {
// 因为target字段在第一项，所以处理columnIndex === 0 的列
  if (columnIndex === 0) {
    return row.span || [0, 0];
  } else {
    return [1, 1];
  }
}
```

## 或许你会有这样的疑问？
- 为什么要分组？

> 分组的意义有两点： 1、分组再扁平化，能够将需要聚合的数据排在相邻的位置
> 2、分组之后方便计算聚合项的长度，方便后面返回rowSpan的长度。

- 返回[0, 0] 什么意思？

> `spanMethod`返回的rowSpan\colSpan是当前数据项占用的列表的宽高，[1,1]表示一行一列，[0, 0]则表示0行0列，即不占用空间，这样聚合项就能达到预期的占位了。



## 结语

合并单元格是业务中比较常见的需求，谈不上有什么难度，希望各位看官有更简洁的思路可以在评论区探讨~










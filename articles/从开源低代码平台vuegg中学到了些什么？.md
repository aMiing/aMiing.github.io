# 阅读 vuegg 源码我都学到了些什么？

## 目的

了解到 vuegg 的整体设计和我最近在重构的可视化项目非常接近，带着几个问题探究一下：

1. 数据本地化存储的设计
2. undo\redo 的设计
3. 组件属性表单的设计
4. 导出代码

## 本地化存储

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d29f5cb0ab14604bd06a563a666ec06~tplv-k3u1fbpfcp-watermark.image?)
打开控制台发现 vuegg 使用 indexDB 存储数据，为什么在 localstorage、sessionStorage、indexDB 中选择 indexDB 作为本地化存储方案呢？
带着疑惑我在代码里面搜索关键字，发现它使用的是 `localforage` 这个第三方库。
查找相关文档了解到：

> localForage 是一个 JavaScript 库，通过简单类似 localStorage API 的异步存储来改进你的 Web 应用程序的离线体验。它能存储多种类型的数据，而不仅仅是字符串。
> localForage 有一个优雅降级策略，若浏览器不支持 IndexedDB 或 WebSQL，则使用 localStorage。在所有主流浏览器中都可用：Chrome，Firefox，IE 和 Safari（包括 Safari Mobile）。
> localForage 提供回调 API 同时也支持 ES6 Promises API，你可以自行选择。

基本可以概括为三点原因：

1. localForage 能存储多种类型的数据
2. 兼容性好，有相应的降级策略
3. 支持 promise

综上，仅仅是第一点就能够让我想要拥抱`localForage`了。 [文档地址](http://localforage.docschina.org/)

## undo、redo 的设计思路

vuegg 中的 undo 和 redo 的设计看起来还是非常简洁的，他是放在了一个 mixin 中维护，同时将前进后退的方法挂载到全局，供其他组件调用。

代码地址：[redoundo.js](https://github.com/vuegg/vuegg/blob/master/client/src/mixins/redoundo.js)

其中有几个 trick 我觉得设计的还是很巧妙的：

### 添加历史记录

`this.$store.subscribe`监听 mations 提交，这一点和我们的实际思路是一样的。但是 `if (mutation.type.charAt(0) !== '_')` 这一步排除掉初始化等非操作性提交的设计，还是挺不错的。 带有下划线开头的 mations 不添加到历史记录中。

### 绑定全局方法

```JavaScript
this.$root.$on('undo', this.undo)
this.$root.$on('redo', this.redo)
```

这样就可以在其他任意组件内调用前进后退的方法了。

### 历史记录的存储

vuegg 是在 mixin 里面维护`done\undone`两个数组，当后退时从 done 最后取出一项（pop）, 推入到 undone 中（push）。

相比我之前维护一个历史记录数组，记录当前 index，然后维护 index 的加减，这样做的好处大概有两个：

1. 不需要维护 index,在变化的那一条永远在数组的最后。（index 的维护其实蛮累的，要区分各种场景）
2. 只要是来自操作的变化（非 undo）都可以清空 undone 数组，比较清晰

### 全局 store 的更新

使用`store.replaceState`api 替换 store 的根状态， 当然存储历史的时候，也是存储了整个根 store: `this.done.push(cloneDeep(state))`。

> 疑惑：
> 这样整体思路虽然清晰，但是存储根 store 然的优点粗暴，因为可能根 store 里面还存储了用户信息、登录状态、项目列表诸如此类的信息，这样会造成数据的冗余，甚至是异常，也会是历史记录变得很臃肿。
> 这个地方是否应该像我目前的方案一样，仅存储差异数据呢？ 不过这样做的成本就是计算逻辑变得相对复杂一些。

## 组件属性表单的设计

vuegg 关于组件配置表单的设计还是比较传统的，拆分成一套通用配置和组件特有配置，特有配置根据不同组件进行显示与隐藏的切换。

我们以文本属性为例：

```html
<menu-toggle menuHeader="Text" :hidden="hideTextSettings">
  <div class="menu">
    <text-align
      @change="newValue => onStyleChanges('text-align', newValue)"
      :textAlign="sty['text-align']"
    ></text-align>

    <font-style
      @change="({prop, value}) => onStyleChanges(prop, value)"
      :fontWeight="sty['font-weight']"
      :fontStyle="sty['font-style']"
      :textDecoration="sty['text-decoration']"
    ></font-style>

    <slider
      label="Font size"
      icon="system/editor/font_size"
      min="1"
      max="100"
      step="1"
      :value="parseInt(sty['font-size']) || 16"
      @change="currentValue => onStyleChanges('font-size', (currentValue + 'px'))"
    ></slider>

    <color-picker
      label="Text Color"
      icon="system/editor/text_color"
      :color="sty['color']"
      @input="newColor => onStyleChanges('color', newColor)"
    ></color-picker>

    <icon-select
      class="text-item"
      icon="system/editor/font"
      label="Font family"
      :value="sty['font-family'] || 'Roboto, sans-serif'"
      @change="newValue => onStyleChanges('font-family', newValue)"
    >
      <optgroup v-for="fontFamily in fonts" :key="fontFamily.family" :label="fontFamily.family">
        <option v-for="font in fontFamily.fonts" :key="font.name" :value="font.definition">
          {{font.name}}
        </option>
      </optgroup>
    </icon-select>

    <mdc-textfield
      class="text-item"
      v-model="att.value"
      label="Text"
      dense
      v-if="(typeof att.value !== 'undefined' && att.value !== null)"
      @keyup.native="e => onAttrsChanges('value', e.target.value)"
    />
    <mdc-textfield
      class="text-item"
      v-model="txt"
      label="Text"
      dense
      v-else
      @keyup.native="e => emitChanges('text', e.target.value)"
    />
  </div>
</menu-toggle>
```

不过比较值得学习的是他还是定制了一些颗粒度比较小的样式组件，比如`font-style`这样的小组件。

关于组建颗粒度的方案探讨可以看下我的这篇 [如何设计组件配置项的颗粒度](https://juejin.cn/post/7107262121842327560)

### 导出代码

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a6c65ee148224e14872bc851f96cfb12~tplv-k3u1fbpfcp-watermark.image?)

这部分也是我比较感兴趣的部分，vuegg 可以把配置好的项目，导出为 `.gg`项目和 `vue source(.zip)`，可以脱离系统独立运行的源码项目。

可视化大屏的一个核心的规划功能就是实现大屏的导出，然后能够独立运行。

我看这部分逻辑是在后端 nodejs 中完成的

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a903d801516d49ccaceafc7713cebac2~tplv-k3u1fbpfcp-watermark.image?)

具体实现还没有深究，目前来看和这个 temp 有很大关系，猜测是类似于缓存数据文件，将核心源码与缓存数据组装之后压缩，返回给前端下载。 这部分后续补上~

### 总结

之前在实现 redo、undo 上走了漫长的一段弯路，也许早点看看前辈们的开源项目，能够汲取一些好的实现方案。
所以，磨刀不误砍柴工，还是要多阅读下源码，汲取营养！

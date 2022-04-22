# 面试题总结

## svg优化？
1. 将不绑定事件的元素属性“Pointer-event”设置为None。
2. 重复标签定义在defs中
   > defs元素的子元素在解析的时候不会渲染，只有在use的时候才会渲染，所以这就是一次解析，到处渲染。
3. Filter属性百分比
   > 在filter元素中的属性x、y、width、height最好用百分比，能够将计算路径效果的区域减少到最小，加快图形的显示速度。
4. 减少透明度使用
   > 如果有可能的话尽量使用fill-opacity或者stroke-opacity这两个开销小些的属性。
5. 重绘区域尽可能小
6. 减少色彩渐变和滤镜
7. 减少stroke属性使用
   > Stroke属性也是非常消耗性能，主要是内部计算的开销会比较大，所以如果能用fill的地方就不要用stroke属性。
8. 从服务器端生成SVG发送到客户端
9. 使用预定义的SVG图形
    >使用预定义的SVG图形如<rect>，<circle>，<ellipse>，<line>和<polygon>取代路径。优选预定义的形状有助于减少生成最终图像所需的标签量，也意味着较少的浏览器解析和点阵描述代码。减少SVG复杂度也意味着浏览器可以更快地显示它。
    
10. 如果您必须使用路径（Path），请尝试减少曲线路径，尽量简化和合并它们
11. 避免使用组（Group）。如果不能，请尝试简化它们。
12. 删除不可见的图层。
13. 使用Minify和gzip压缩您的SVG文件。

## svg和canvas性能比较？
1. canvas是基于像素的，使用单个html元素绘制，只能脚本驱动，依赖设备分辨率，保存格式jpg、png位图，适用于绘制区域小，数据量大的场景。
2. svg是基于图形元素的，使用多个图形元素绘制，支持脚本和css修改，不依赖设备分辨率，保存格式svg、适用于绘制面积达，数据量较小的场景。

## webpack都做了那些事？
1. 模块打包：commonjs、ESM
2. 编译、兼容。编译：.less, .vue, .jsx； 兼容：polyfill；
3. 能力扩展。 webpack的Plugin： 按需加载，代码压缩。

## Loader
> 对非JS类型文件先进行必要的转换。
1. file-loader: 把文件输出到一个文件夹中，处理图片和字体。
2. url-loader: 设置阈值，低于阈值编码成base64,超过阈值使用file-loader.
3. svg-inline-loader: 将压缩后的svg注入代码中。
4. image-loader: 加载并且压缩图片文件.
5. json-loader: 加载json文件。
6. babel-loader: 将ES6转换成ES5。
7. i18n-loader: 国际化。
8. ts-loader: 将 TypeScript 转换成 JavaScript.
9. awesome-typescript-loader: 将 TypeScript 转换成 JavaScript, 性能更好。
10. sass-loader: 将SCSS/SASS代码转换成CSS.
11. css-loader: 加载css、支持模块化、压缩、文件导入等特性。
12. style-loader: 把 CSS 代码注入到 JavaScript 中，通过 DOM 操作去加载 CSS.
13. postcss-loader: 扩展css语法，使用下一代css，可以配合autoprefixer插件自动补齐css3前缀。
14. tslint-loader: 通过tslint检查javascript.
15. eslint-loader: 通过eslint检查tsScript。
16. vue-loader: 加载vue.js单文件组件。
17. cache-loader: 可以在一些性能开销较大的loader之前添加，将结果缓存到内存中。

## Plugin
> webpack基于发布订阅模式，在运行的生命周期中会广播出许多事件，插件通过监听这些事件，就可以在特定的阶段执行自己的插件任务，从而实现自己想要的功能。
1. html-webpack-plugin：简化 HTML 文件创建 (依赖于 html-loader)
2. web-webpack-plugin: 可方便地为单页应用输出HTML,比 html-webpack-plugin好用。
3. speed-measure-webpack-plugin：可以看到loader和plugin的执行耗时，整个打包耗时。
4. define-plugin： 定义环境变量（webpack4之后指定mode会自动配置）
5. ignore-plugin: 忽略部分文件。
6. terser-webpack-plugin： 支持压缩ES6（webpack4）
7. mini-css-extract-plugin: 分离样式文件，css提取为单独文件，支持按需加载。（替代extract-text-webpack-plugin）

## Loader和Plugin的区别？
> Loader负责文件转换，Plugin负责功能扩展。
 Loader 本质就是函数，在函数中对接收到的内容进行转换，返回转换后的结果。因为webpack只认识javascript,所以Loader就成了翻译官，对其他资源进行转换和预编译的工作。

 Plugin 就是插件，基于事件流框架Tapable，插件可以扩展webpack的功能，在webpack运行的生命周期内会广播出很多事件，Plugin可以监听这些事件，在合适的时机通过webpack提供的API改变输出结果。

 Loader在modules.rule里面配置，类型是数组。每一项都是Object,内部包含了test、loader、options等属性。
 Plugin在Plugins里面单独配置，类型为数组，每一项都是一个Plugin实例，参数通过构造函数传入。

## 链表和数组的区别？
数组是物理空间连续的内存地址，通过下标维护的数据结构。通过下标查询速度比较快O（1），插入删除都需要将其后的元素向前\向后移动，时间复杂度是O（n）。

链表是物理空间无序、不连续的数据结构。其逻辑上的有序是通过指针维护的。查询由于需要遍历查找，时间复杂度O（n），插入删除则是从某个位置断开，修改相邻元素的指针指向，时间复杂度O（1）.

## SourceMap: 将编译、打包、压缩后的代码映射回源代码的技术
## 原型链？Number原型？
> js里面一切皆对象，Number的prototype是Object,Object的__proto__指向null。 避免出现无限循环。

> 原型链的核心思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。

[原型与原型链详解](/articles/PrototypeChain.md)

## 虚拟dom的优点？
1. 尽量少的操作DOM，避免频繁操作Dom造成重排与重绘，优化性能。
2. 更好的跨平台，用于服务端渲染等。
3. 相比直接操作Dom，虚拟Dom效率更高。

## 虚拟Dom优化
snabbdom.js **双端比较算法**，vue2主要借鉴它的原理。
inferno.js 性能强悍不仅仅是算法 但确实是目前最快的diff, vue3借鉴了它的实现。


[返回首页](/)

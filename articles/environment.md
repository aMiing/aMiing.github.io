# 判断是 node 环境还是浏览器环境

## window

只有浏览器中才会有 window 对象（webworker 中除外）

```js
if (window && typeof window === "object") {
    // browser
} else if (Object.prototype.toString().call(process) === "[object process]") {
    // nodejs
}
```

## module.exports

nodejs 环境是遵守 commonjs 模块规范的，所以可以通过判断是否有`module.exports`来判定环境。

```js
if (typeof module !== "undefined" && module.exports) {
    //node
} else {
    //浏览器
}
```

## axios 的实现方式

axios 是通过浏览器环境有`XMLHttpRequest`对象、nodejs 环境有`process`对象来判定的。

```js
function getDefaultAdapter() {
    var adapter;
    if (typeof XMLHttpRequest !== "undefined") {
        // For browsers use XHR adapter
        adapter = require("./adapters/xhr");
    } else if (typeof process !== "undefined" && Object.prototype.toString.call(process) === "[object process]") {
        // For node use HTTP adapter
        adapter = require("./adapters/http");
    }
    return adapter;
}
```

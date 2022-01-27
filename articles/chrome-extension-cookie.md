# 利用 chrome 插件解决 cookie 跨域的问题

> 由于 chrome 浏览器对 cookie 的限制不断增加，当然是出于安全方面的考虑，如果 cookie 设置了 samesite、domain、httponly，那么我们想要在脚本中获取 cookie 几乎就不太可能，更别说是跨域的 cookie 了。

但是我们在本地开发的时候，通常要获取代理环境（开发服务器或测试服务器、线上服务器等）的 cookie，来获取数据供本地调试代码。

明确需求：本地 domain 获取服务器 domain 的 cookie。
那么有什么好办法能满足我们的需求呢？

经过一番调研发现 chrome 扩展程序能获得更高的权限，突破 cookie 的同源限制。

## 具体方案：cookie 搬运

> 我们借助 chrome-extension 的能力，从 source 源搬运 cookie 到 target。

具体实现：

### 1、为扩展声明权限列表

```js
// manifest.json
"permissions": ["storage", "webRequest", "webRequestBlocking", "*://*/", "cookies"],

```

### 2、popup 界面配置信息

```js
// manifest.json
 "browser_action": {
        "default_icon": "sources/icon128.png",
        "default_title": "解决Chrome浏览器Cookie携带问题",
        "default_popup": "index.html"
}
```

![cookie 配置页面](/source/images/cookie.png)

### 3、配置信息存储到 localstorage 中

```js
localStorage.setItem("domainList", JSON.stringify(domainList));
```

### 4、监听 storage 变化，执行搬运

```js
//background.js
window.addEventListener("storage", ({ key, newValue, oldValue }) => {
    console.log("key", key);
    if (key === "domainList") {
        // 比较变化的数据，判断是增添或者删除
        // chrome.cookies.remove(object details)
        newValue = JSON.parse(newValue) || [];
        oldValue = JSON.parse(oldValue) || [];
        storageList = newValue;
        if (newValue.length > oldValue.length) {
            // 新增，最后一条为新增，仅取最后一条
            init(newValue.slice(-1));
        } else {
            // 移除cookie
            const deleteValue = oldValue.find(e => !newValue.some(n => n === e));
            console.log("deleteValue", deleteValue);
            chrome.cookies.remove({
                url: "https://" + deleteValue.to,
                name: deleteValue.name,
            });
        }
    }
});

function setCookie(cookie, obj) {
    // 向target添加cookie
    const res = cookie;
    chrome.cookies.set(
        {
            url: "https://" + obj.to,
            domain: obj.hostname,
            value: res.value,
            name: res.name,
            httpOnly: res.httpOnly,
        },
        function (cookie) {}
    );
}
const init = domainList => {
    // 遍历domainList,添加规则
    for (const iterator of domainList) {
        // 从source源获取cookie
        chrome.cookies.get(
            {
                url: "https://" + iterator.from,
                name: iterator.name,
            },
            cookie => {
                setCookie(cookie, iterator);
            }
        );
    }
};
```

### 5、监听 source 源 cookie 的变化，并更新 target 源的 cookie

```js
chrome.cookies.onChanged.addListener(function (changeInfo) {
    const fromList = storageList.map(e => e.from);
    if (fromList.includes(changeInfo.cookie.domain)) {
        const target = storageList.find(e => e.from === changeInfo.cookie.domain);
        // 移除
        if (changeInfo.removed) {
            chrome.cookies.remove(
                {
                    url: "https://" + target.to,
                    name: changeInfo.cookie["name"],
                },
                function (cookie) {}
            );
        }
        // 设置、更新
        else {
            chrome.cookies.set(
                {
                    url: "https://" + target.to,
                    name: changeInfo.cookie["name"],
                    path: "/",
                    value: changeInfo.cookie["value"],
                    expirationDate: changeInfo.cookie["expirationDate"],
                    secure: true,
                    sameSite: "no_restriction", // 不阻止跨域cookie
                },
                function (cookie) {}
            );
        }
    }
});
```

## 源码仓库 

源码开源在[https://github.com/aMiing/chrome-extension-cookie](https://github.com/aMiing/chrome-extension-cookie)


[返回首页](/)
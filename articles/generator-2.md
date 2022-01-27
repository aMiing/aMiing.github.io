# 生成器函数异步执行的实际场景

<!-- 异步执行三个函数： 分别是 获取用户信息、根据用户获取订单列表、根据订单查询商品，顺序执行，间隔一秒 -->

```js
// 定义这三个函数

function getUsers() {
    setTimeout(() => {
        const data = '用户列表';
        iterator.next(data);
    }, 1000)
}

function getOrders() {
    setTimeout(() => {
        const data = '订单列表';
        iterator.next(data);
    }, 1000)
}

function getGoods() {
    setTimeout(() => {
        const data = '商品列表';
        iterator.next(data);
    }, 1000)
}

// 定义生成器函数
function * gen() {
    const users = yield getUsers();
    console.log(users, 'users');
    const orders = yield getOrders();
    console.log(orders, 'orders');
    const goods = yield getGoods();
    console.log(goods, 'goods');
}
const iterator = gen();
iterator.next();

// 执行结果
// 间隔一秒分别打印
// 用户列表 users
// 订单列表 orders
// 商品列表 goods
```

[返回首页](/)
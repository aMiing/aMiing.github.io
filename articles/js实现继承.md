# ES5 和 ES6分别如何实现继承？

## 前言
继承这个概念在面向对象编程思想里面十分重要，也是面试必考的考点之一。
javascript的继承主要是依托其原型与原型链的概念来实现的。
> ECMAscript将原型链作为实现继承的主要方法。

对原型与原型链还不是很了解的同学可以看下[原型与原型链，你真的学废了吗？](https://juejin.cn/post/7065640525193281573)

## 先来看看ES6的实现
ES6提供了Class关键字来实现类的定义，Class 可以通过extends关键字实现继承，让子类继承父类的属性和方法。

这句话概括性很强，具体的就没要展开了，实现的细节部分后面会单独在我的专栏[【重学ES6】](https://juejin.cn/column/7067420864710459406)系列中重点剖析,感兴趣的可以关注下，共同学习成长。

咱们重点讲一下ES5的三种常用的实现方式。
## ES5实现的三种方式

### 1. 原型链继承
原型链继承的原理很简单，直接让子类的原型对象指向父类实例，当子类实例找不到对应的属性和方法时，就会往它的原型对象，也就是父类实例上找，从而实现对父类的属性和方法的继承
```js
function Person() {
    this.name = 'Back_kk';
}
Person.prototype.getName = function() {
    return this.name;
}
function Student() {}
Student.prototype = new Person();
// 根据原型链的规则,顺便绑定一下constructor, 这一步不影响继承, 只是在用到constructor时会需要
// 原型的实例等于自身
Student.prototype.constructor = Student;

const student = new Student();
console.log(student.name); // Back_kk
console.log(student.getName()); // Back_kk

```

#### 缺陷
1. 由于所有Student实例原型都指向同一个Person实例, 因此对某个Student实例的来自父类的引用类型变量修改会影响所有的Student实例

例如：
```js
function Person() {
    this.obj = {
        name: 'Back_kk',
        age: 18
    };
}
function Student() {}
Student.prototype = new Person();
// 根据原型链的规则,顺便绑定一下constructor, 这一步不影响继承, 只是在用到constructor时会需要
// 原型的实例等于自身
Student.prototype.constructor = Student;

const student1 = new Student();
student1.obj.name = '佩奇';
const student2 = new Student();
console.log(student2.obj.name); // 佩奇
```

2. 在创建子类实例时无法向父类构造传参, 即没有实现super()的功能
> 那么能不能实现super()功能呢？大家有兴趣可以思考下。

### 2. 构造函数继承
构造函数继承，即在子类的构造函数中执行父类的构造函数，并为其绑定子类的this，让父类的构造函数把成员属性和方法都挂到子类的this上去，这样既能避免实例之间共享一个原型实例，又能向父类构造方法传参。

```js
function Person(name) {
    this.name = name
}
Person.prototype.getName = function() {
    return this.name;
}
function Student() {
    Person.apply(this, arguments);
}

const student = new Student('Back_kk');
console.log(student.name); // Back_kk
```
#### 缺陷
- 继承不到父类原型上的属性和方法
 
  Students类实际上是调用Person类来生成的实例

  ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/443b5a4914794b93a5976ccab98285dd~tplv-k3u1fbpfcp-watermark.image?)

  能否交加修改让其获取到Person原型上的属性和方法呢？

    ```js
    function Person(name) {
        this.name = name
    }
    Person.prototype.getName = function() {
        return this.name;
    }
    function Student() {
        // 这里偷偷用了ES6的解构，不影响大局不要在意哈
       return new Person(...arguments);
    }
    const student = new Student('Back_kk');
    console.log(student); // Back_kk
    ```
    这是这样顾此失彼，student的构造方法变成了Person,这显然违背了我们的初衷。
    ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7acb4551fab446c6a131b17c99b12f3b~tplv-k3u1fbpfcp-watermark.image?)

### 3. 组合式继承
组合是继承结合了原型集成和构造函数继承的特点。
```js
function Person(name) {
    this.name = name;
}
Person.prototype.getName = function() {
    return this.name;
}
function Student() {
    // 构造函数继承
    Person.apply(this, arguments)
}
// 原型式继承
Student.prototype = new Person();

// 原型的实例等于自身
Student.prototype.constructor = Student;

const student = new Student('Back_kk');
console.log(student.name); // Back_kk
console.log(student.getName()); // Back_kk

```
#### 缺陷
- 每次创建子类实例都执行了两次构造函数(Person.apply和new Person())，虽然这并不影响对父类的继承，但子类创建实例时，原型中会存在两份相同的属性和方法，这并不优雅。
  
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d7e70a265e7466fb678a8d1f2b4ccd2~tplv-k3u1fbpfcp-watermark.image?)
> **“原型中会存在两份相同的属性和方法”**，这句没有很理解，哪位大佬可以帮忙解答下？
### 4. 寄生式组合继承
解决构造函数被执行两次的问题, 我们将指向父类实例改为指向父类原型, 减去一次构造函数的执行。
```js
function Person(name) {
    this.name = name;
}
Person.prototype.getName = function() {
    return this.name;
}
function Student() {
    // 构造函数继承
    Person.apply(this, arguments)
}
// 原型式继承
// Student.prototype = new Person();
Student.prototype = Object.create(Person.prototype);

// 原型的实例等于自身
Student.prototype.constructor = Student;

const student = new Student('Back_kk');
console.log(student.name); // Back_kk
console.log(student.getName()); // Back_kk
```

关于Object.create()在原型链上的指向问题可以康康 [这里](https://juejin.cn/post/7065640525193281573#:~:text=%E5%90%8C%E4%B8%80%E7%A7%8D%E6%83%85%E5%86%B5%E3%80%82-,%E9%80%9A%E8%BF%87,-Object.create()%E5%88%9B%E5%BB%BA)

这是目前ES5中比较成熟的继承方式了。


## 总结
- 说到js继承，最开始想到的应该是是原型链继承，通过把子类实例的原型指向父类实例来继承父类的属性和方法，但原型链继承的缺陷在于对子类实例继承的引用类型的修改会影响到所有的实例对象以及无法向父类的构造方法传参。
- 构造函数继承, 通过在子类构造函数中调用父类构造函数并传入子类this来获取父类的属性和方法，但构造函数继承也存在缺陷，构造函数继承不能继承到父类原型链上的属性和方法。
- 后面有了组合式继承，但也有了新的问题，每次都会执行两次父类的构造方法，最终有了寄生式组合式继承。

## 预告
+ 我们在上面继承实现里面是用到了`new`关键字，那么new的底层实现原理是什么？它是如何实现的呢？我们将在下一篇文章中具体分析下这个问题，敬请期待。
+ 关于ES6的继承实现原理，也会在后面的文章继续深入讲解。

## 相关推荐
+ [原型与原型链，你真的学废了吗？](https://juejin.cn/post/7065640525193281573)

+ [你说的this是哪个this?](https://juejin.cn/post/7063047546205110279)


## 参考
<https://www.cnblogs.com/0314dxj/p/13886141.html>



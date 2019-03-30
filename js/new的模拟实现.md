#### new的原理

1、创建一个新对象

2、将构造函数的作用域赋给新对象（因此 this 就指向了这个新对象）

3、执行构造函数中的代码（为这个新对象添加属性）

4、返回新对象

#### new的简单使用

例如：

```javascript
function P(firstName, lastName) {
    this.age = 10
    this.getName = function() {
        return `${firstName}-${lastName}`
    }
}

var p1 = new P('chen', 'yh')
p1.getName()
// 'chen-yh'
```

模拟实现的 new 的用法如下:

```javascript
var p1 = newF('构造函数', '构造函数的参数')
```

#### 实现

```javascript
function newF() {
    let obj = {}
    // 创建一个新的空对象，其实也是方法将要返回的实例对象
    let Constructor = Array.prototype.shift.call(arguments)
    // 利用shift方法取出第一个参数并且赋给Constructor，即我们传入的构造函数
    // 此操作也将伪类数组变成真正的数组，去除第一个参数后，arguments剩余的就是构造器中的参数
    obj._proto_ = Constructor.prototype
    // 我们知道实例对象的 _proto_ 指向构造函数的原型对象prototype
    // 这里将obj的原型指向构造函数，此时obj可以访问构造函数原型中的属性
    let result = Constructor.apply(obj, arguments)
    // 构造函数的this指向obj，此时obj也可以访问构造函数中的属性了
    return typeof result === 'object' ? result : obj
    // 确保new出来的是个对象
}
```

#### 使用

```javascript
var p1 = newF(P,'chen', 'yh')
p1.getName()
// 'chen-yh'
```

#### 参考

[彻底捋清楚 new 的实现](https://github.com/amandakelake/blog/issues/37)


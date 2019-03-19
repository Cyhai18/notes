#### 思路

举个简单例子：

```javascript
var foo = {
    value: 1
}
function bar() {
    console.log(this.value)
}
bar.call(foo)
// 1
```

上面其实可以把`bar`看成是`foo`对象的一个方法即：`foo.bar()`。所以模拟`call`的实现的思路可以是：`将被改变this指向的函数设置为对象的属性，执行完毕这个函数后，再删除该函数`。

#### 模拟call

```javascript
Function.prototype.myCall = function(context) {
    // call是函数对象的一个方法
    var context = context || window
    // 参数是代替原来this指向的对象，比如上文的foo对象，不传第一个参数，默认是window
    context.fn = this
    // 给context添加一个属性，这个this指向调用mycall的函数，比如上文的bar
    var args = [...arguments].slice(1)
    // args表示除开context参数后剩下的参数
    var result = context.fn(...args)
    // 执行函数
    delete context.fn
    // 删除函数
    return result
}
```

上面关键是对参数的处理，以及如何理解里面的`this`

#### 模拟apply

和call不同的是，apply的第二个参数是数组

```javascript
Function.prototype.myApply = function(context) {
    var context = context || window
    context.fn = this
    var result
    
    // 判断是否存在第二个参数，这是一个数组
    if(arguments[1]) {
        result = context.fn(...arguments[1])
    } else {
        result = context.fn()
    }
    
    delete context.fn
    return result
}
```

#### 模拟bind

`bind`特别一点的是：`会返回一个函数`。

```javascript
var foo = {
    value: 1
}

function bar(name, age) {
    console.log(this.value)
    console.log(name)
    console.log(age)
}

var bindFoo = bar.bind(foo, 'chen')
bindFoo(18)
// 1
// chen
// 18
// 在bind时候只传了一个name，在调用bind返回的函数时再传了age
```

分以下阶段从易到难来实现比较好理解：

- 返回函数的模拟实现
- 传参的模拟实现（这里先实现的是这个）
- 构造函数效果的模拟实现（也就是说当 bind 返回的函数作为构造函数的时候，bind 时指定的 this 值会失效，但传入的参数依然生效）

```javascript
Function.prototype.myBind = function(context) {
    var self = this
    // 这里的this指的是调用bind方法的函数对象，例如bar.bind(foo)中的bar
    var args = [...arguments].slice(1)
    // 获取传入myBind函数中从第二个一直到最后的参数
    // 或者利用 Array.prototype.slice.call(arguments, 1)
    return function() {
        // myBind方法返回的这个函数
        var reArgs = [...arguments]
        // 这里的arguments是指bind返回的函数传入的参数
        return self.apply(context, args.concat(reArgs))
        // 哈哈，体会这里连接两个参数数组的妙处
    }
}
```

#### 三者区别

相同点：

- 都是用来改变函数的this对象的指向的；
- 第一个参数都是this要指向的对象，也就是想指定的上下文；
- 都可以利用后续参数传参；

不同点：

- bind是返回对应函数，便于稍后调用；
- apply、call则是立即调用，call直接传入每个参数，apply以数组的形式传入参数

#### 三者的妙用

迟点补上

#### 函数对象内部的属性和方法

注意一点就是：call、apply、bind都是函数内部的方法，一般都是函数去调用它们，所以可以在每个模拟函数开头添加：

```javascript
if(typeof this !== 'function') {
    throw new Error('Error')
}
```

接下来顺便引出来其他一些内部属性和方法：

迟点补上

#### 参考

[模拟实现call、apply、bind](https://github.com/amandakelake/blog/issues/16)

[JavaScript深入之bind的模拟实现](https://github.com/mqyqingfeng/Blog/issues/12)

[函数的内部属性和方法（arguments、callee）](https://github.com/amandakelake/blog/issues/6)














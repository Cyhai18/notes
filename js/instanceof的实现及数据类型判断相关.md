#### typeof

在 JS 中，存在着 6 种原始类型，分别是：`number`、`boolean`、`string`、`null`、`undefined`、`symbol`。需要注意的是：原始类型不包含 object 类型。

- 首先原始类型存储的都是值，是没有函数可以调用的，比如`undefined.toString()`，但明明`'1'.toString()`是可以使用的，在这种情况下，`'1'`已经不是原始类型了，被强制转换成了`String`类型也就是对象类型。
- JS 在底层存储变量的时候，会在变量的机器码的低位1-3位存储其类型信息
  - 对象：000
  - 字符串：100
  - null：所有机器码均为0

所以`typeof null`会输出`object`就是这个原因。

`typeof`对于原始类型来说，除了`null`都可以显示正确的类型。

```javascript
typeof 1 // 'number'
typeof '1' // 'string'
typeof true // 'boolean'
typeof undefined // 'undefined'
typeof Symbol() // 'symbol'
```

`typeof`对于对象类型来说，除了函数都会显示`object`，所以说`typeof`并不能准确判断变量到底是什么类型。

```javascript
typeof [] // 'object'
typeof {} // 'object'
typeof function() {} // 'function'
```

#### 利用Object.prototype.toString.call()

`toString()`方法返回一个表示该对象的字符串

```javascript
Object.prototype.toString.call('1') // "[object String]"
Object.prototype.toString.call(1) // "[object Number]"
Object.prototype.toString.call(true) // "[object Boolean]"
Object.prototype.toString.call([]) // "[object Array]"
Object.prototype.toString.call(function() {}) // "[object Function]"
Object.prototype.toString.call(null) // "[object Null]"
```

但不能直接使用`true.toString()`、`[].toString()`来判断，为什么？

```javascript
[1, 2, 3].toString() // "1,2,3"
"1".toString() // "1"
```

由`Object`构造出来的`Array`，`Function`等对象实例，本身也有一个`toString`方法，这是重写的。但如果要判断数据的类型，我们要用到的是`Object`的原型对象上的`toString`方法。可以测试一下：

```javascript
var arr = [1, 2, 3]
console.log(Array.prototype.hasOwnProperty('toString')) // true
// hasOwnProperty() 方法会返回一个布尔值，指示对象自身属性中是否具有指定的属性,不会包含继承的属性
console.log(arr.toString()) // "1,2,3"
delete Array.prototype.toString // 删除这个方法
console.log(Array.prototype.hasOwnProperty('toString')) // false
console.log(arr.toString()) // "[object Array]"
//这里删除了自身的tostring方法，会沿着原型链，找到Object的toString方法
```

#### 利用Object.prototype.constructor

此属性返回创建实例对象的构造函数的引用，所有对象都会从它的原型上继承一个`constructor`属性。可以利用构造函数来帮我们判断数据类型。

```javascript
[1, 2, 3].constructor === Array // true
"1".constructor === String // true
{}.constructor === Object // true
function() {} === Function // true
```

#### instanceof

`object instanceof constructor`

左边是要测试的对象，右边是构造函数

用来测试一个对象在其原型链中（顺着&#95;proto&#95;一直往上找）是否存在一个构造函数的原型对象

```javascript
function _instanceof(L, R) {
    // L表示左边的object，R表示右边的constructor
    const P = R.prototype
    // P表示构造函数的原型对象
    L = L.__proto__
    // L其实是等于P的，因为_proto_指向创建该对象的构造函数的原型对象
    // 接下来会判断两者是否相等
    while(true) {
        // 循环条件为true,表示在原型链中一直找，
        // 直到L为Object.prototype.__proto__ === null即没有找到返回false
        if(L === null) {
            return false
        }
        if(L === P) {
            return true
        }
        L = L.__proto__
        // 顺着原型链重新被赋值
    }
}
```

测试一下：

```javascript
console.log(_instanceof(Object, Object))
// true
// Object.__proto__ = Function.prototype
// Function.prototype.__proto__ = Object.prototype
```

但要注意一点就是，对于原始类型来说，想直接通过`instanceof`来判断类型是不行的，需要先把原始类型转为对象类型才行。

```javascript
var str = '1'
str instanceof String // false

var str1 = new String('1')
str1 instanceof String // true
```

#### 参考

[你真的了解 instanceof 吗？](https://github.com/amandakelake/blog/issues/36)

[为什么用Object.prototype.toString.call(obj)检测对象类型?](https://www.cnblogs.com/youhong/p/6209054.html)


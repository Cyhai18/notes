- 先给出测试的代码：

  ```javascript
  var data = { name: 'chen' }
  observe(data)
  function update(value) {
      document.querySelector('div').innerText = value
  }
  // 模拟更新数据后，view层也能够得到同步更新
  new Watcher(data, 'name', update)
  data.name = 'luo'
  
  // view层更新后，如何派发更新到model层呢？简陋的方法就是：
  // 在属性的getter中获取view层的内容
  // get: function() {
  //     return document.querySelector('div').innerText
  // }
  // 当读取属性的时候，自然就获得更新过后的view层的内容了
  ```

- Vue 内部使用`Object.defineProperty()`来实现数据响应式。

  ```javascript
  function observe(obj) {
      // 判断类型
      if (!obj || typeof obj !== 'object') {
          return
      }
      // Object.keys()返回一个由一个给定对象的自身可枚举属性组成的数组
      Object.keys(obj).forEach(key => {
          defineReactive(obj, key, obj[key])
      })
  }
  
  function defineReactive(obj, key, val) {
      // 递归子属性
      observe(val)
      // let dp = new Dep()
      Object.defineProperty(obj, key, {
          // 可枚举
          enumerable: true,
          // 可配置
          configurable: true,
          get: function reactiveGetter() {
              console.log('get value')
              // 将Watcher添加到订阅，注意一下Dep.target已经指向Watcher了 
              // if (Dep.target) {
              //     dp.addSub(Dep.target)
              // }
              return val
          },
          set: function reactiveSetter(newVal) {
              console.log('change value')
              val = newVal
              // 执行Watcher的update方法
              // dp.notify()
          }
      })
  }
  ```

  上面这样子还不行，我们需要在属性有更新的时候，自动地去派发更新（就像 Vue 实例中的 data 对象有数据更新，能自动派发更新 view 层）。要实现这个，需要先触发依赖收集。

- 实现一个`Dep`类，用于属性的依赖收集和派发更新操作。

  ```javascript
  class Dep {
      constructor() {
          this.subs = []
      }
      // 添加依赖
      addSub(sub) {
          this.subs.push(sub)
      }
      // 更新
      notify() {
          this.subs.forEach(sub => {
              sub.update()
          })
      }
  }
  // 全局属性，通过该属性配置Watcher，实际相当于一个判断标识
  Dep.target = null
  ```

- 触发依赖收集的操作：

  ```javascript
  class Watcher {
      constructor(obj, key, cb) {
          // 将Dep.target指向自己
          Dep.target = this
          this.cb = cb
          this.obj = obj
          this.key = key
          // 触发属性的getter添加监听
          // 这里可以说是最核心的一点：手动触发一次属性的getter来实现依赖收集
          this.value = obj[key]
          // 最后将Dep.target置空，免得再次触发属性的getter时，再将Watcher添加到订阅
          Dep.target = null
      }
      update() {
          // 获得新值
          this.value = this.obj[this.key]
          // 调用update方法更新DOM
          this.cb(this.value)
      }
  }
  ```

  在执行上面的构造函数时候将`Dep.target`指向自身，从而使得收集到了对应的`Watcher`，在派发更新的时候取出对应的`Watcher`然后执行`update`函数。

- 接下来，需要对上面的`defineReactive`函数进行改造，添加上依赖收集和派发更新的代码。（将会用注释的形式给出）

- `Object.defineProperty`的缺陷：

  如果通过下标方式修改数组数据或者给对象新增属性并不会触发组件的重新渲染，因为`Object.defineProperty`不能拦截到这些操作，更准确的来说，对于数组而言，大部分操作都是拦截不到的，只是 Vue 内部通过重写函数的方式解决了这个问题。

- 参考：

  yck 的小册 | 掘金


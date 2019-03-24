#### $on、$emit / props

利用`$on`与`$emit`来进行子组件向父组件传递数据，利用`props`来进行父组件向子组件传递数据，**不支持跨级**。

> 用`$emit`在当前组件实例上触发自定义事件，并传递一些参数给监听器的回调，在父级调用这个组件时，使用`@on`的方式来监听自定义事件。
>
> 下面的自实现 dispatch 和 broadcast 会用到它们。

有个骚操作：组件使用`$emit`在自己实例上触发事件，自己也可以用`$on`来监听得到。

```vue
<template>
	<div><button @click="handleEmitEvent">触发自定义事件</button></div>
</template>
<script>
    export default {
        methods: {
            handleEmitEvent() {
                // 在当前组件上触发自定义事件test，并传值
                this.$emit('test', 'hello vue.js')
            }
        },
        mounted() {
            // 监听自己触发的自定义事件test
            // 好似多此一举，大可在handleEmitEvent里直接alert
            // 但如果handleEmitEvent不是在自己组件里面被button调用，而是给其它组件调用的，那这个                用法就大有可为了
            this.$on('test', (text) => {
                window.alert(text)
            })
        }
    }
</script>
```

#### bus

用一个空的 Vue 实例作为中央事件总线（bus），也就是一个中介。

```html
<div id="app">
    {{ message }}
    <component-a></component-a>
</div>
<script>
	var bus = new Vue()
    Vue.component('component-a',{
        template: '<button @click="handleEvent">传递事件</button>',
        methods: {
            handleEvent: function() {
                // 通过bus把事件on-message发出去
                bus.$emit('on-message', '来自组件component-a的内容')
            }
        }
    })
    var app = new Vue({
        el: '#app',
        data: { message: "" },
        mounted: function() {
            var _this = this
            // 在实例初始化时，监听来自bus实例的事件
            bus.$on('on-message', function(msg) {
                _this.message = msg
            })
        }
    })
</script>
```

这种方法巧妙而轻量地实现了任何组件间的通信，包括父子、兄弟、跨级。

#### $parent / $children

在子组件中，使用`this.$parent`可以直接访问该组件的父组件，父组件也可以通过`this.$children`访问它所有的子组件。也不要忘了`ref`。

#### provide / inject

Vue 2.2.0 新增的 API，两者需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。

Vue 不建议在业务中使用这对 API，而是在插件 / 组件库使用。

> 只要一个组件使用了`provide`向下提供数据，那其下所有的子组件都可以通过`inject`来注入，不管中间隔了多少代，而且可以注入多个来自不同父级提供的数据。

```javascript
// A.vue
export default {
    provide: { name: 'chen' }
    // 将name这个变量提供给它的所有子组件
}

// B.vue（是A.vue的子组件）
export default {
    inject: ['name'],
    // 注入了从A组件中提供的name变量
    mounted() {
        console.log(this.name) // chen
        // 在B组件中，就可以直接通过this.name来访问这个变量了
    }
}
// A.vue的name如果改变了，B.vue的this.name是不会改变的
```

> provide 和 inject 绑定并不是可响应的，这是刻意为之的，然而，如果你传了一个可监听的对象，那么其对象的属性还是可响应的。

想一下如何用 provide / inject 来代替 Vuex？

在 webpack 中使用 Vue，一般一个入口文件 main.js 里面会有一个入口组件 app.vue 作为跟组件，将 app.vue 用来存储所有需要的全局数据和状态，所以可以把**整个 app.vue 实例通过 provide 对外提供**。

```javascript
// app.vue
export default {
    provide() {
        // 把整个app.vue的实例命名为app
        return { app: this }
    }
}
// app.vue是整个项目第一个被渲染的组件，而且只会渲染一次（即使切换路由，也不会被再次渲染），利用这个特性，很适合做一次性全局的状态数据管理
```

在没有这对 API 之前，可以用`$parent`和`计算属性`达到相同效果：

```javascript
computed: {
    // 假设想在某个子组件里要获得父组件为form的实例
    form() {
        let parent = this.$parent
        while (parent.$options.name !== 'form') {
            parent = parent.$parent
    	}
        return parent
    }
}
// 每个组件都设置name选项，作为组件名的标识，利用这个特点，通过向上遍历，直到找到需要的组件
// 相比 inject 来说太费劲了，别人一行代码搞定
export default {
    inject: ['form']
}
```

provide / inject 主要解决了跨级组件间的通信问题，不过它的使用场景，**主要是子组件获取上级组件的状态（因为是非响应式的）**，跨级组件间建立了一种主动提供与依赖注入的关系。还是没有很好解决两种场景：

- 父组件向子组件（支持跨级）传递数据
- 子组件向父组件（支持跨级）传递数据

#### $dispatch / $broadcast

Vue.js 1.x 的产物，在 Vue.js 2.x 里废弃了。还是有必要去了解一下。

> 前者用于向上级派发事件，只要是它的父级（一级或多级以上），都可以在组件内通过`$on`监听到。
>
> 后者相反，是由上级向下次广播事件的。
>
> 这两种方法一旦发出事件后，任何组件都是可以接收到的，就近原则，而且会在第一次接收到后停止冒泡，除非返回true。

```vue
// 注意：需用Vue.js 1.x版本
// 子组件
<template>
	<button @click="handleDispatch">派发事件</button>
</template>
<script>
    export default {
        methods: {
            handleDispatch() {
                this.$dispatch('test', 'hello Vue.js')
            }
        }
    }
</script>
```

```vue
// 父组件
<template>
	<child-component></child-component>
</template>
<script>
    export default {
        mounted() {
            this.$on('test', (text) => {
                console.log(text) // hello Vue.js
            })
        }
    }
</script>
```

在 Vue.js 2.x 中自行实现 dispatch 和 broadcast 方法，不能保证跟原生具有完全相同的体验，基本功能是一样的，都是**解决父子组件（含跨级）间的通信问题**。

自实现的方法将具有以下功能：

> 在子组件调用 dispatch 方法，向上级**指定的**组件实例（最近的）上触发自定义事件，并传递数据，且该上级组件已预先通过 $on 监听了这个事件。
>
> broadcast 刚好相反。

实现这对方法的关键点在于**如何向上或向下准确地找到组件实例**，并在它上面触发方法。肯定是通过遍历来匹配组件的`name`选项。

> 在设计一个新功能时，先确定这个功能的 API 是什么，也就是说方法名、参数、使用样例，确定好 API，再来写具体的代码。

1、Vue.js 内置的方法，才以`$`开头，我们的 dispatch 和 broadcast 方法名前不加。

2、可能很多组件中都会使用该方法，复用起见，封装在混合（mixins）里。

3、接收3个参数，第一个是组件的`name`值，第二个是自定义事件名，第三个是要传递的数据。

```vue
// 所以使用样例可以是这样的：
<script>
	import Emitter from '../mixins/emitter.js'
    export default {
        mixins: [ Emitter ],
        methods: {
            handleDispatch() {
                this.dispatch()
            }
            handleBroadcast() {
            	this.broadcast()
        	}
        }
    }
</script>
```

**emitter.js** 的代码如下：

```javascript
export default {
    methods: {
        dispatch(componentName, eventName, params) {
            // 当前实例的父实例，没有的话，就返回当前组件树的根实例
            let parent = this.$parent || this.$root
            while (parent) {
                if (parent.$options.name === componentName) {
                    return parent.$emit(eventName, params)
                }
                parent = parent.$parent
                // 根组件的parent是undefined，所以循环会结束的
            }
        },
        
        broadcast(componentName, eventName, params) {
            // 注意一下，$children表示当前实例的直接子组件，一个组件是可以有多个直接子组件的
            this.$children.forEach(child => {
                const name = child.$options.name
                if (name === componentName) {
                    // 这里其实不用apply也是可以的了
                    child.$emit.apply(child, [eventName].concat(params))
                } else {
                    // 继续向下查找，轮到子组件的子组件，从而实现跨级通信
                    broadcast.apply(child, [componentName, eventName].concat([params]))
                }
            })
		}
    }
}
// 因为是用mixins导入，在methods里定义的这两个方法会被混合到组件里，自然可以用this.dispatch和this.broadcast来调用
```

具体用法：

```vue
// parent.vue（parent和child可能是跨级的），父向子通信
<template>
	<button @click="handleClick">触发事件</button>
</template>
<script>
	import Emitter from '../mixins/emitter.js'
    export default {
        name: 'parent',
        mixins: [Emitter],
        methods: {
            handleClick() {
                this.broadcast('child', 'on-message', 'hello Vue.js')
            }
        }
    }
</script>
```

```vue
// child.vue
<script>
    export default {
        name: 'child',
        created() {
            this.$on('on-message', this.showMessage)
        },
        methods: {
            showMessage(text) {
                window.alert(text)
            }
        }
    }
</script>
```

#### findComponents 系列方法

这一系列方法可以帮助我们**找到任意组件实例**，可以说是组件通信的终极方案。

findComponents 系列方法最终都是返回组件的实例，进而可以读取或调用该组件的数据和方法。

系列方法的实现原理：**都是通过递归、遍历，找到指定组件的`name`选项匹配的组件实例并返回**。

- **向上找到最近的指定组件——findComponentUpward**

  ```javascript
  function findComponentUpward(context, componentName) {
      // 第一个参数是当前组件的上下文（实例），也就是传入this
      // 第二个参数是要找的组件的name
      let parent = context.$parent
      let name = parent.$options.name
      while (parent && (!name || [componentName].indexof(name) < 0)) {
          parent = parent.$parent
          if (parent) name = parent.$options.name
      }
      return parent
  }
  ```

- **向上找到所有的指定组件——findComponentsUpward**

  ```javascript
  function findComponentsUpward(context, componentName) {
      let parents = []
      const parent = context.$parent
      if (parent) {
          if (parent.$options.name === componentName) parents.push(parent)
          return parents.concat(findComponentsUpward(parent, componentName))
      } else {
          return []
      }
  }
  // 返回的是一个数组，包含了所有找到的组件实例
  ```

  使用场景较少，一般只用在递归组件里面，只有递归组件的父级才是自身（componentName没有相同的）

- **向下找到最近的指定组件——findComponentDownward**

  ```javascript
  function findComponentDownward(context, componentName) {
      const childrens = context.$children
      let children = null
      if (childrens.length) {
          for (const child of childrens) {
              const name = child.$options.name
              if (name === componentName) {
                  children = child
                  break
              } else {
                  children = findComponentDownward(child, componentName)
                  if (children) break
              }
          }
      }
      return children
  }
  ```

- **向下找到所有指定的组件——findComponentsDownward**

  ```javascript
  function findComponentsDownward(context, componentName) {
      return context.$children.reduce((components, child) => {
          if (child.$options.name === componentName) components.push(child)
          const foundChilds = findComponentsDownward(child, componentName)
          return components.concat(foundChilds)
      }, [])
  }
  ```

- **找到指定组件的兄弟组件——findBrothersComponents**

  ```javascript
  function findBrothersComponents(context, componentName, exceptMe = true) {
      // 参数exceptMe意思为是否把本身除外，默认是true
      // 寻找兄弟组件的方法，是先获取父组件的全部子组件，这里面当然包含了本身
      let res = context.$parent.$children.filter(item => {
          return item.$options.name === componentName
      })
      // Vue.js在渲染组件时，都会给每个组件加一个内置的属性_uid，这个_uid是不会重复的，借此从一系        列兄弟组件中把自己排除掉。
      // findIndex返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1。
      let index = res.findIndex(item => item._uid === context._uid)
      if (exceptMe) res.splice(index, 1)
      return res
  }
  ```

  不是说 componentName 是不重复的吗？res 为什么还会存在当前组件本身？举个例子，组件A是组件B的父级，在B中找到所有在A中的兄弟组件（也就是所有在A中的B组件）。

以上是我的读书笔记，来自

**Vue.js 组件精讲 | 掘金小册**

**Vue.js 实战 | 梁灏**
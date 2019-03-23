- `keep-alive`的理解：简单说，就是把一个组件的编译缓存起来，避免组件的重复渲染。

- `v-once`只渲染元素和组件**一次**，随后的重新渲染，元素/组件及其所有的子节点将被视为静态内容并跳过，这可以用于优化更新性能。

- 计算属性（computed）的 set。大多数时候，我们只是用它默认的 get 方法。

  ```javascript
  computed: {
      fullName: function() {
          return `${this.firstName}-${this.lastName}`
      }
  }
  // 这里的fullName也可以写为一个Object，而非Function，只是Function形式是我们默认使用它的get方法，当写为Object时，还能使用它的set方法
  computed: {
      fullName: {
          get() {
              return `${this.firstName}-${this.lastName}`
          },
          set(val) {
              const names = val.split('-')
              this.firstName = names[0]
              this.lastName = names[names.length - 1]
  		}
      }
  }
  // 如果执行 this.fullName = 'chen-yh'，computed 的 set 就会调用，firstName 和 lastName 会被赋值为 chen 和 yh
  ```

- 注意一下`$set`。有两种情况会用到它：

  - 由于 JS 的限制，Vue 不能检测以下变动的数组：

    1、当利用索引直接设置一个项时，例如：`this.items[index] = value`

    2、当修改数组的长度时，例如：`this.items.length = newLength`

  - 由于 JS 的限制，Vue 不能检测对象属性的添加或删除

  ```javascript
  var app = new Vue({
      data: {
          items1: [1, 2, 3],
          items2: { a: 1 }
      },
      methods: {
          handler: function() {
              this.items1[1] = 'x' // 不是响应式的
              this.$set(this.items1, 1, 'x') // 是响应式的
              this.items2.b = 2 // 不是响应式的
              this.$set(this.items2, 'b', 2) // 是响应式的
          }
      }
  })
  ```

  另外数组的以下方法，都是可以触发视图更新的，也就是响应性的：

  `push()`、`pop()`、`shift()`、`unshift()`、`splice()`、`sort()`、`reverse()`。

  还有一种小技巧，就是先 copy 一个数组，然后通过 index 修改后，再把原数组整个替换。

  ```javascript
  handler() {
      const data = [...this.items1]
      data[1] = 'x'
      this.items1 = data
  }
  ```

- 组件名推荐使用小写加减号分割的形式命名。

  ```javascript
  Vue.component('my-component', {
      // 选项
  })
  ```

- vue 组件的模板在某些情况下会受到 HTML 的限制，比如`<table>`内规定只允许是`<tr>`、`<td>`、`<th>`等这些表格元素，所以在`<table>`内直接使用组件是无效的。**使用特殊的 is 属性来挂载组件**。

  ```html
  <div id="app">
      <table>
          <tbody is="my-component"></tbody>
      </table>
  </div>
  ```

  tbody 在渲染时，会被替换为组件的内容。常见的限制元素还有：`<ul>`、`<ol>`、`<select>`。

  如果用的是字符串模板（例如`.vue`单文件用法）就没有这个限制。

- **组件中的选项 data 必须是函数，将数据 return 出去。**

  我们知道 JS 对象是引用关系，return 出的对象如果引用的是外部的一个对象，这个对象一修改，会导致同步修改。

  ```html
  <div id="app">
      <my-component></my-component>
      <my-component></my-component>
      <my-component></my-component>
  </div>
  <script>
  	var data = { counter: 0 }
      Vue.component('my-component', {
          template: '<button @click="counter++">{{ counter }}</button>',
          data: function() {
              return data
          }
      })
  </script>
  ```

  任意点击一个组件，3个数字都会加1，这就是引用外部对象的后果，所以给组件返回一个新的 data 对象来独立。

- 父组件向子组件传递数据会用到`props`。在组件中，使用选项`props`来声明需要从父级接收的数据，`props`的值可以是两种，字符串数组或者对象。

  由于 HTML 特性不区分大小写，当使用 DOM 模板时，驼峰命名（camelCase）的 props 名称要转为短横分隔命名（kebab-case）。

  ```html
  <div id="app">
      <my-component warning-text="提示信息"></my-component>
  </div>
  <script>
      Vue.component('my-component', {
          props: ['warningText'],
          template: '<div>{{ warningText }}</div>'
      })
  </script>
  ```

  如果使用的是字符串模板，仍然可以忽略这些限制。

  如果要想传递来自父组件的动态数据，就使用指令`v-bind`来动态绑定 props 的值，父组件数据变化时，也会传递给子组件。

  **如果直接传递数字、布尔值、数组、对象，而且不使用 v-bind，传递的仅仅是字符串**

  ```html
  <div>
      <my-component message="[1, 2, 3]"></my-component> // 7
      <my-component :message="[1, 2, 3]"></my-component> // 3
  </div>
  <script>
      Vue.component('my-component', {
          props: ['message'],
          template: '<div>{{ message.length }}</div>'
      })
  </script>
  ```

  Vue 2.x 通过 Props 传递数据是**单向**的了，也就是子组件数据变化不能通过它传递给父组件。而 Vue 1.x 提供了`.sync`修饰符来支持双向绑定。之所以这样设计，为了尽可能将父子组件解耦，避免子组件无意中修改了父组件的状态。

  不过在 Vue 的**2.3.0**版本又增加了`.sync`修饰符，用法与 1.x 的不完全相同。2.x 的`.sync`不是真正的双向绑定，而是一个语法糖，修改数据还是在父组件完成，并非在子组件。

  ```html
  <my-component :value.sync="value"></my-component>
  <script>
      Vue.component('my-component', {
          template: '<button @click="increase(1)">{{ value }}</button>',
          props: {
              value: { type: Number }
          },
          methods: {
              increase: function(val) {
                  this.$emit('update:value', this.value + val)
              } 
          }
      })
      var app = new Vue({
          data: {
              value: 1
          }
      })
  </script>
  // 这个例子和下面的联系起来，看起来要比v-model的实现简单多，实现的效果是一样的。
  // 注意：v-model在一个组件中只能有一个，但.sync可以设置很多个。
  // .sync虽好，但也有限制，不能和表达式一起使用，不能用在字面量对象上。
  ```

  因为 props 也是数组或者是对象，它的数据在子组件内改变是会影响父组件的。所以最好先将它们作为data内变量的初始值保存起来，避免直接操作。

  **一般组件提供给别人使用时，推荐都进行数据验证**，这时 props 就用对象写法。

  ```javascript
  // 当prop验证失败时，在开发版本下会在控制台抛出一条警告
  props: {
      // 必须是数字类型
      propA: Number,
      // 必须是字符串或数字类型
      propB: [String, Number],
      // 布尔值，如果没有定义，默认值就是 true
      propC: {
      	type: Boolean,
          default: true
      },
      // 数字，而且是必传
      propD: {
      	type: Number,
      	required: true
      },
      // 自定义一个验证函数
      propE: {
      	validator: function(value) {
      		return value > 10
      	}
      },
      // 如果是数组（或对象），默认值必须是一个函数来返回
      propF: {
      	type: Array,
          default: function() {
          	return []
          }
      }
  }
  ```

- Vue 提供了子组件索引的方法，用特殊的属性`ref`来为子组件指定一个索引名称。$refs 只在组件渲染完成后才填充，并且它是非响应式的。它仅仅作为一个直接访问子组件的应急方案，应当避免在模板或计算属性中使用 $refs。

  ```html
  <component-a ref="comA"></component-a> // 使用ref指定一个名称
  <script>
  	var app = new Vue()
      this.$refs.comA
      // 父组件实例通过$refs来访问指定名称的子组件
  </script>
  ```

- 用自定义事件 event 由子组件向父组件传递数据。子组件用`$emit()`来触发事件，父组件用`$on()`来监听子组件的事件。`$emit()`方法的第一个参数是自定义事件的名称，后面的参数都是要传递的数据，可以不填或填写多个。

  ```html
  <my-component @increase="handleGetTotal"></my-component>
  <script>
      Vue.component('my-component', {
          template: '<div><button @click="handleIncrease"></button></div>',
          data: function() {
              return { counter: 0 }
          },
          methods: {
              handleIncrease: function() {
                  this.counter++
                  this.$emit('increase', this.counter)
              }
          }
      })
      var app = new Vue({
          el: '#app',
          data: { total: 0 },
          methods: {
              handleGetTotal: function(total) { this.total = total }
          }
      })
  </script>
  ```

  除了用`v-on`在组件上监听自定义事件外，也可以监听 DOM 事件，这时可以用`.native`修饰符表示监听的是一个原生事件，监听的是该组件的根元素：

  ```html
  <my-component v-on:click.native="handleClick"></my-component>
  ```

  ```html
  // 在自定义组件上使用v-model指令
  // 仍然是点击按钮加1的效果
  <div id="app">
      <p>{{ total }}</p> // 这里的total是在父组件实例上定义的数据，省略了代码
      <my-component v-model="total"></my-component>
      // 并没有使用@input="handler"，而是直接用了v-model绑定的一个数据total，这也可以称作是一个        语法糖
  </div>
  <script>
      Vue.component('my-component', {
          template: '<button @click="handleClick">+1</button>',
          data: function() {
              return { counter: 0 }
          },
          methods: {
              handleClick: function() {
                  this.counter++
                  this.$emit('input', this.counter)
                  // 事件名是特殊的input
              }
          }
      })
  </script>
  // v-model是一个语法糖，可以拆解为 props: value 和 events: input
  // 就是说组件必须提供一个名为 value 的 prop，以及名为 input 的自定义事件
  // 满足这两个条件，使用者就能在自定义组件上使用 v-model
  // 从Vue的2.2.0版本开始，提供了一个model选项可以指定prop和event的名字，而不一定非要用value和input，因为这两个名字在一些原生表单元素里，有其它用处
  ```

  ```html
  // v-model还可以用来创建自定义的表单输入组件，进行数据的双向绑定
  <div id="app">
      <p>{{ total }}</p>
      <my-component v-model="total"></my-component>
      <button @click="handleReduce">-1</button>
  </div>
  <script>
      Vue.component('my-component', {
          props: ['value'],
          template: '<input :value="value" @input="updateValue">',
          methods: {
              updateValue: function(event) {
                  this.$emit('input', event.target.value)
              }
          }
      })
      var app = new Vue({
          el: '#app',
          data: { total: 0 },
          methods: {
              handleReduce: function() {
                  this.total--
              }
          }
      })
  </script>
  ```

  实现这样一个具有双向绑定的`v-model`组件要满足下面两个要求：

  - 接收一个value属性
  - 在有新的value时触发input事件

- **props 传递数据、events 触发事件、slot 内容分发构成了 Vue 组件的3个 API 来源，再复杂的组件也是由这3部分构成的。**

- slot 分发的内容，作用域是在父组件上的。子组件`<slot>`内的备用内容，它的作用域是子组件本身。

  在子组件内使用特殊的`<slot>`元素就可以为这个子组件开启一个 slot（插槽），在父组件模板里，插入在子组件标签内的所有内容将代替子组件的`<slot>`标签及它的内容。

  **在组合使用组件时，内容分发 API 至关重要**

  ```html
  // 作用域插槽的用法
  <child-component>
      // 这里的props只是一个临时变量，类似v-for="item in items"里面的item一样
      // template内可以通过props访问来自子组件插槽的数据msg
  	<template scope="props">
      	<p>来自父组件的内容</p>
          <p>{{ props.msg }}</p>
      </template>
  </child-component>
  <script>
      Vue.component('child-component', {
          // <slot>元素上的msg会传到插槽
          template: `<div>
      					<slot msg="来自子组件的内容"></slot>
      			   </div>`
      })
  </script>
  ```

  通过`$slots`可以访问某个具名 slot，`this.$slots.default`包括了所有没有被包含在具名 slot 中的节点。

- 组件在它的模板内可以递归地调用自己，只要给组件设置 name 的选项就可以了。注意的是，必须给一个条件来限制递归数量，否则会抛出错误。

- Vue 提供了一个内联模板的功能，在使用组件时，给组件标签使用`inline-template`特性，组件就会把它的内容当作模板，而不是把它当内容分发，这样模板更灵活。父组件的数据和子组件的数据都可以渲染（如果同名，优先使用子组件的数据），这反而是内联模板的缺点。

  ```html
  <child-component inline-template>
  	<div>
          <h2>在父组件中定义子组件的模板</h2>
          <p>{{ message }}</p> // 可以是父组件的数据
          <p>{{ msg }}</p> // 也可以是子组件的数据
      </div>
  </child-component>
  ```

- Vue 提供了一个特殊的元素`<component>`用来动态地挂载不同的组件，使用`is`特性来选择要挂载的组件。

  ```html
  <div>
      <component :is="currentView"></component>
      <button @click="handleChangeView('A')">切换到A</button>
  </div>
  <script>
      var app = new Vue({
          el: '#app',
          components: {
              comA: { template: '<div>组件A</div>' },
              comB: { template: '<div>组件B</div>' },
              comC: { template: '<div>组件C</div>' }
          },
          data: { currentView: 'comA' },
          methods: {
              handleChangeView: function(component) {
                  this.currentView = 'com' + component
                  // 改变参数component的值就可以动态挂载组件了
              }
          }
      })
  </script>
  ```

- **异步更新队列**。Vue 在观察到数据变化时并不是直接更新 DOM，而是开启一个队列，并缓冲在同一事件循环中发生的所有数据改变。在缓冲时会去除重复数据，从而避免不必要的计算和 DOM 操作。然后在下一个事件循环 tick 中，Vue 刷新队列并执行实际（已去重的）工作。

  如果你用一个 for 循环来动态改变数据100次，其实它只会应用最后一次改变，如果没有这种机制，DOM 就要重绘100次，这固然是一个很大的开销。

  Vue 会根据当前浏览器环境优先使用原生的 promise.then 和 MutationObserver，如果都不支持，就会采用 setTimeout 代替。

  ```html
  <div id="app">
      <div id="div" v-if="showDiv">这是一段文本</div>
      <button @click="getText">获取div内容</button>
  </div>
  <script>
      var app = new Vue({
          el: "#app",
          data: { showDiv: false },
          methods: {
              getText: function() {
                  // 事实上在执行完后，div仍然还是没有被创建出来，直到下一个vue事件循环时，才开始				   创建
                  this.showDiv = true
                  // var text = document.getElementById('div').innerHTML
                  // 这样会抛出“获取不到div元素”的错误
                  // console.log(text)
                  // $nextTick就是用来知道什么时候DOM更新完成的
                  // 将回调延迟到下次DOM更新循环之后执行。在修改数据之后立即使用它，然后等待DOM更				   新
                  this.$nextTick(function() {
                      var text = document.getElementById('div').innerHTML
                      console.log(text)
                  })
              }
          }
      })
  </script>
  ```

  理论上，我们应该不用去主动操作 DOM，本来 Vue 的核心思想就是数据驱动 DOM。

- vue 提供了`Vue.extend`和`$mount`两个方法来手动挂载一个实例。`Vue.extend`是基础 Vue 构造器，创建一个子类，参数是一个包含组件选项的对象。如果 vue 实例在实例化时没有收到 el 选项，它就处于未挂载状态，没有关联的 DOM 元素。可以使用`$mount()`手动挂载一个未挂载的实例，这个方法返回实例自身，因而可以链式调用其他实例方法。

  ```html
  <div id="mount-div"></div>
  <script>
      var myComponent = Vue.extend({
          template: '<div>hello: {{ name }}</div>',
          data: function() {
              return { name: "chen" }
          }
      })
      new myComponent().$mount('#mount-div')
  </script>
  ```

- 其他的点：

  1、v-show 与 v-if 的区别（使用 v-if 在性能优化上有什么经验）

  2、绑定 class 的数组用法

  3、计算属性和 watch 的区别

  4、事件修饰符

  5、Vuex 中 mutations 和 actions 的区别

  6、Render 函数（单独写）

  7、生命周期（可能会单独写）

  8、路由的跳转方式

  9、什么是 MVVM，与 MVC 有什么区别

- 以上是我的读书笔记，来自

  **Vue.js 实战 | 梁灏**

  **Vue.js 组件精讲 | 掘金小册**

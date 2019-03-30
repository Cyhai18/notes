#### Virtual Dom

React 和 Vue 2 都使用了 Virtual Dom（虚拟 DOM） 技术。

Virtual Dom 并不是真正意义上的 DOM，而是一个轻量级的 JavaScript 对象，在状态发生变化时，Virtual Dom 会进行 Diff 运算，来更新只需要被替换的 DOM，而不是全部重绘，从而来提升渲染性能。

与 DOM 操作相比，Virtual Dom 是基于 JavaScript 计算的，所以开销会小很多。

平时写 Vue 组件时，模板都是写在 template 里的，但是在 Vue.js 编译时，都会解析为 Virtual Dom。

```html
// 正常的DOM节点在HTML中是这样的：
<div id="main">
    <p>文本内容</p>
    <p>文本内容</p>
</div>
```

```javascript
// 用Virtual Dom创建的JavaScript对象一般会是这样的：
var vNode = {
    tag: 'div',
    attributes: {
        id: 'main'
    },
    children: [
        // p 节点
    ]
}
// vNode 对象通过一些特定的选项描述了真实的DOM结构
// 可参考资料看下完整的vNode
```

在 Vue.js 2 中，Virtual Dom 就是通过一种 VNode 类表达的，每个 DOM 元素或组件都对应一个 VNode 对象。

![](..\images\Virtual Dom运行过程.png)

使用 Virtual Dom 就可以完全发挥 JavaScript 的编程能力。在多数场景中，使用 template 就足够了，但在一些特定的场景下，使用 Virtual Dom 会更简单，Render 函数就是 Vue 用于实现 Virtual Dom 的一种手段。

#### 用法

```javascript
render: function(createElement) {
    return createElement()
}
```

Render 函数通过 createElement 参数来创建 Virtual Dom，createElement 构成了 Vue Virtual Dom 的模板，它有 3 个参数。

```javascript
createElement(
	// {String | Object | Function}
    // 第一个参数必选，可以是一个HTML标签、组件或函数
    'div',
    
    // {Object}
    // 可选，一个对应属性的数据对象，具体的选项如下。可以在template中使用
    {   
        class: { // 和v-bind:class一样的API
            foo: true,
            bar: false
        },
        style: { // 和v-bind:style一样的API
            color: 'red',
            fontSize: '14px'
        },
        attrs: { // 正常的HTML特性
            id: 'foo'
        },
        props: { // 组件props
            myProp: 'bar'
        },
        domProps: { // DOM属性
            innerHTML: 'baz'
        },
        on: { // 自定义事件监听器"on"，不支持如V-on:keyup.enter的修饰器，需要手动匹配keyCode
            click: this.clickHandler
        }
        // 剩下的可查资料
    },
    
    // {String | Array}
    // 可选，子节点（VNodes），用法一致
    // 包括：文本节点、普通元素节点、组件节点、没有内容的注释节点、克隆节点（可以是任意类型的节点）
    [
        createElement('h1', 'hello world'),
        createElement(MyComponent, {
            props: {
                someProp: 'foo'
            }
        }),
        'bar'
    ]
)
```

然后来个例子：（要注意在合适的场景使用 Render 函数，否则只会增加负担）

```html
// 传统的template写法
<ele></ele>
<script>
    Vue.component('ele', {
        template: '<div id="element" :class="{show: show}" @click="down">文本</div>',
        data: function() {
            return { show: true }
        },
        methods: {
            down: function() { console.log('down') }
        }
    })
</script>

// 使用Render改写后的代码如下：
<script>
    Vue.component('ele', {
        render: function(createElement) {
            return createElement(
            	'div',
                {
                    class: {
                        show: this.show
                    },
                    attrs: {
                        id: 'element'
                    },
                    on: {
                        click: this.down
                    }
                },
                '文本'
            )
        }
    })
</script>
```

#### 注意点

1、如果 VNode 是组件或含有组件的 slot，那么 VNode 必须唯一。对于重复渲染多个组件（或元素）的方法有很多，例如：

```javascript
// 局部声明组件
var Child = {
    render: function(createElement) {
        return createElement('p', 'text')
    }
}
Vue.component('ele', {
    render: function(createElement) {
        return createElement('div',
        	Array.apply(null, { length: 5}).map(() => return createElement(Child))
            // 生成一个长度为5的空数组，然后返回包含5个子组件Child的数组
        )
    }
})

Vue.component('ele', {
    render: function(createElement) {
        // 创建一个组件节点，使用组件Child
        var ChildNode = createElement(Child)
        return createElement('div', [
            ChildNode,
            ChildNode
        ])
    }
})
// 这个示例是错误的。期望在子节点内（div里）渲染出两个Child组件，实际只渲染了一个，因为在这种情况下，VNode受到了约束
```

2、使用 JavaScript 代替模板功能。在 Render 函数中，没办法使用 v-if、v-for 等，但可以利用 JS 去实现这些功能。

```html
// 实现 v-if
<div id="app">
    <ele :show="show"></ele>
    <button @click="show = !show">切换show</button>
</div>
<script>
    Vue.component('ele', {
        render: function(createElement) {
            if (this.show) {
                return createElement('p', 'show的值为true')
            } else {
                return createElement('p', 'show的值为false')
            }
        },
        props: {
            show: {
                type: Boolean,
                default: false
            }
        }
    })
    var app = new Vue({
        el: '#app',
        data: { show: false }
    })
</script>
```

3、在一开始接触 Render 写法时，可能会有点不适应，说到底也只是 js 的一个普通函数而已，写习惯后就没有那么难理解了。

```html
<div id="app">
    <ele :list="list"></ele>
    <button @click="handleClick">显示列表</button>
</div>
<script>
    Vue.component('ele', {
        render: function(createElement) {
            if (this.list.length) {
                return createElement('url', this.list.map(item => {
                    return createElement('li', item)
                }))
            } else {
                return createElement('p', '列表为空')
            }
        },
        props: {
            list: {
                type: Array,
                default: function() {
                    return []
                }
            }
        }
    })
    var app = new Vue({
        el: '#app',
        data: { list: [] },
        methods: {
            handleClick: function() {
                this.list = ['chen', 'luo', 'zhang']
            }
        }
    })
</script>
```

上面的 Render 函数对应的 template 写法如下：

```html
<ul v-if="list.length">
    <li v-for="item in list"> {{ item }} </li>
</ul>
<p v-else>列表为空</p>
```

#### 函数化组件

- Vue 提供了一个 **functional** 的布尔值选项，设置为 true 可以使组件无状态和无实例，也就是没有 data 和 this 上下文。

  这样用 render 函数返回虚拟节点可以更容易渲染，因为函数化组件只是一个函数，渲染开销要小很多。

- 使用函数化组件时，Render 函数提供了第二个参数 **context** 来提供临时上下文。

  组件需要的 data、props、slots、children、parent 都是通过这个上下文来传递的，比如：

  `this.level`要改为`context.props.level`

  `this.$slots.default`要改为`context.children`

#### JSX

JSX 是一种看起来像 HTML，但实际是 JavaScript 的语法扩展，它用更接近 DOM 结构的形式来描述一个组件的 UI 和状态信息。

> 为了让 Render 函数更好地书写和阅读，Vue 提供了插件 babel-plugin-transform-vue-jsx 来支持 JSX 语法。

```html
// template书写
<Anchor :level="1">
	<span>一级</span> 标题
</Anchor>

// 使用createElement改写
<script>
    render: function(createElement) {
        return createElement('Anchor', {
            props: {
                level: 1
            }
        }, [
            createElement('span', '一级'),
            '标题'
        ])
    }
</script>

// 使用JSX改写
<script>
    render (h) {
        return (
        	<Anchor level={1}>
    			<span>一级</span> 标题
    		</Anchor>
        )
    }
</script>
```

#### 来自

Vue.js 实战 | 梁灏


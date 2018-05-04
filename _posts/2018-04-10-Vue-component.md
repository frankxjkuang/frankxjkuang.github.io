---
layout: post
title:  Vue学习笔记（组件）
date:   2018-04-10 5:00:00 +0800
categories: Vue
tag: note
---

* content
{:toc}


组件是什么？
====================================
[Vue 组件](https://cn.vuejs.org/v2/guide/components.html#%E4%BB%80%E4%B9%88%E6%98%AF%E7%BB%84%E4%BB%B6%EF%BC%9F) 组建的一些基本知识在开发中扮演的角色在几大框架中基本一样，我在这里只记录一些注意的东西：

`DOM` 模板解析注意事项
-----------------------------------
像 `<option> <tr> <td>` 这样的元素只能出现在某些特定元素的内部，在自定义组件中使用这些受限制的元素时会导致一些问题，例如官网的例子：

```html
<!-- <my-row> 会被当作无效的内容 -->
<table>
  <my-row>...</my-row>
</table>

<!-- 使用特殊的 is 特性 -->
<table>
  <tr is="my-row"></tr>
</table>  
```

`data` 必须是函数
-----------------------------------
这里推荐看一下[官网的例子](https://cn.vuejs.org/v2/guide/components.html#data-%E5%BF%85%E9%A1%BB%E6%98%AF%E5%87%BD%E6%95%B0)，我的理解是主要是为了返回一个单独的data拷贝，避免data与其它的对象公用一个对象引用。

组件通信
====================================
组建通信应该是最为重要以及核心的部分，一旦涉及到数据以及事件的传递，组建间的数据共享这些都是开发过程中必定要遇到的问题。

`Prop`
-----------------------------------
**Vue中prop向下传递，事件向上传递**，因此 `Prop` 主要是父组件向子组件传递数据的

```javascript
Vue.component(child, {
    prop: ['msg'],
    // prop 的使用跟 data 一样
    template: `<h1>{{ msg }}</h1>`
})
```

在父组件中可以通过 msg 属性传递数据给子组件：
```html
<child msg="Hello World!"></child>
```

> **PS:**HTML 特性是不区分大小写的。所以，当使用的不是字符串模板时，camelCase (驼峰式命名) 的 prop 需要转换为相对应的 kebab-case (短横线分隔式命名)：
```js
Vue.component('child', {
    // 在 JavaScript 中使用 camelCase
    props: ['myMsg'],
    template: `<h1>{{ myMsg }}</h1>`
})
```
```html
<!-- 在 HTML 中使用 kebab-case -->
<child my-msg="hello!"></child>
```

动态 `Prop`
-----------------------------------
用 `v-bind` 来动态地将 `prop` 绑定到父组件的数据。每当父组件的数据变化时，该变化也会传导给子组件：
```html
<!-- 同上只有细微差别 -->
<child :my-msg="myMsg"></child>
```

如果想把一个对象的所有属性作为 `prop` 进行传递，可以使用不带任何参数的 `v-bind` (即用 v-bind 而不是 v-bind:prop-name)。例如:
```js
todo: {
  text: 'Learn Vue',
  isComplete: false
}
```
```html
<todo-item v-bind="todo"></todo-item>

<!-- 等价与 -->
<todo-item
  v-bind:text="todo.text"
  v-bind:is-complete="todo.isComplete"
></todo-item>
```

`Prop` 验证
-----------------------------------
为组件的接受的 prop 指定验证规则，如果传入的数据不符合要求，Vue 会发出警告，这里记录一下基本的例子：

```js
Vue.component('example', {
  props: {
    // 基础类型检测 (`null` 指允许任何类型)
    propA: Number,
    // 可能是多种类型
    propB: [String, Number],
    // 必传且是字符串
    propC: {
      type: String,
      required: true
    },
    // 数值且有默认值
    propD: {
      type: Number,
      default: 100
    },
    // 数组/对象的默认值应当由一个工厂函数返回
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```

自定义事件
-----------------------------------
解决子组件跟父组件通信

- 使用 `$on(eventName)` 监听事件
- 使用 `$emit(eventName, optionPayload)` 触发事件

下面举例说明：
```html
<div id="example">
  <p>from son: {{ msg }}</p>
  <child v-on:receiveMsg="getMsg"></child>
</div>
```

```javascript
Vue.component('child', {
  template: '<button v-on:click="ask">叫爸爸</button>',
  data: function () {
    return {
      
    }
  },
  methods: {
    ask: function () {
      this.$emit('receiveMsg', '爸爸给钱！')
    }
  },
})

new Vue({
  el: '#example',
  data: {
    msg: ''
  },
  methods: {
    getMsg: function(msg) {
      this.msg = msg
    }
  }
})
```


非父子组建之间的通信
-----------------------------------
有的情况下需要非父子组件之间进行通信，在 `angular` 之前的版本中使用了:
(子组件)$emit --> $on(父组件) $broadcast --> $on(其它子组件)，这样写不用说你也看得出来，最大的问题就是冗余。下面看看在 `vue` 中如何进行非父子组件的通讯问题的

```javascript
// 使用一个空的 Vue 实例作为事件总线
var bus = new Vue()

// 触发组件 A 中的事件
bus.$emit('ask', 'hello')

// 在组件 B 创建的钩子中监听事件
bus.$on('ask', function(msg) {
  // ...
})
```

相较于 `angular`，形式上只是少了根组件转发的步骤，在通讯简单的情况下，这样足矣。

但是如果项目足够复杂，有较多的数据需要在各个组件之间共享，机智的你还会选择这种方式进行数据共享和组件通讯吗？那又有什么好的解决方案呢？

有需求就会有产品[vuex](https://vuex.vuejs.org/zh-cn/intro.html)


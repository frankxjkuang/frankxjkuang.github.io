---
layout: post
title:  Vue学习笔记（指令）
date:   2018-04-03 11:11:00 +0800
categories: document
tag: Vue
---

* content
{:toc}


写在前面
====================================

由于需要，最近在学习 [Vue](https://cn.vuejs.org/) ，之前有过angular相关的开发经验，因此上手起来也是比较快的，加之之前的项目中使用了 [Redux](https://github.com/reactjs/redux) ，因此对于 `Vuex` 的认识也是非常容易，感觉更简单更好用了...使用 `vue-cli` 开发就不说，这里主要记录一下相关的知识点，工具以及用法。

创建 `Vue` 实例
====================================

通过 `new Vue` 创建根 ç 实例：
```javascript
let vm = new Vue({

})
```

数据与方法
------------------------------------

```javascript
let data = {
    a: 1
}

let vm = new Vue({
    data: data, // 数据源
    el: '#app'  // 渲染的根元素 id
})

vm.a === data.a   // => true
vm.$data === data // => true
```
- `data` 对象中能找所有的属性，并与视图层双向绑定
- 只有当实例被创建时 `data` 中存在的属性才是响应式的
- `Object.freeze()`，这会阻止修改现有的属性
- `Vue` 实例暴露了一些有用的属性与方法,它们都有前缀 $，以便与用户定义的属性区分开来

`Vue` 组件
====================================

.vue 文件是一个自定义的文件类型，用类 HTML 语法描述一个 Vue 组件。每个 .vue 文件包含三种类型的顶级语言块 `<template>、<script> 和 <style>`，还允许添加可选的自定义块：

```
<template>
  <div class="example">{{ msg }}</div>
</template>

<script>
export default {
  data () {
    return {
      msg: 'Hello world!'
    }
  }
}
</script>

<style>
.example {
  color: red;
}
</style>

<custom1>
  This could be e.g. documentation for the component.
</custom1>
```
vue-loader 会解析文件，提取每个语言块，如有必要会通过其它 loader 处理，最后将他们组装成一个 CommonJS 模块，module.exports 出一个 Vue.js 组件对象。

> **PS:**这里补充一下在 `.vue` 文件中使用 `sass`
1. 安装 `sass` 依赖包
```npm
npm install sass-loader node-sass --save-dev
```
2. 在 build 文件夹下的 `webpack.base.conf.js` 文件中添加 `rules` 配置：
```javascript
{
    test: /\.scss$/,
    loader: ['style', 'css', 'sass']
}
```
3. 使用 
在 `style` 标签上增如下属性：
```html
<style lang="scss"></style>
```


模板语法
====================================

插值
------------------------------------
数据绑定使用 `Mustache` 语法 (双大括号) 的文本插值(双向数据绑定)：

```html
<p>{{ msg }}</p>

<!-- 一次性的插值使用 v-once, 当数据改变时，插值处的内容不会更新 -->
<p v-once>{{ msg }}</p>
```

将一段 html 字符串解析为html代码，需要使用 `v-html` 
：
```html
<span v-html="htmlStr"></span>
```

`Mustache` 语法不能作用于 `html` 特性上，遇到这种情况应该使用 `v-bind` 指令：
```html
<a v-bind:href="url">这是一个链接</a>
```

> **PS:** 在 `Mustache` 中我们还可以写 `Javascript` 表达式(但是这个有限制，每个绑定都只能包含单个表达式):

```javascript
{{ number += 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ msg.split('').reverse().join('') }}
```


指令
====================================

官方对于指令的描述：指令 (Directives) 是带有 `v-` 前缀的特殊属性。指令属性的值预期是单个 JavaScript 表达式 (`v-for` 是例外情况)。指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM。

这里的 `v-` 跟 `ng-` 就一摸一样了，应该是借鉴于后者。其用法就不多提了，这里主要记一下有趣的东西。

缩写
-----------------------------------
`Vue` 为 `v-bind` 和 `v-on` 这两个最常用的指令提供了缩写（前者是操作 html 特性，后者是绑定事件）：
```html
<!-- v-bind 缩写 -->
<!-- 完整语法 -->
<a v-bind:href="url">...</a>
<!-- 缩写 -->
<a :href="url">...</a>

<!-- v-on 缩写 -->
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>
<!-- 缩写 -->
<a @click="doSomething">...</a>
```

修饰符
-----------------------------------

1. `v-model` 的修饰符
    - `.lazy` 将原本绑定在 input 事件的同步逻辑转变为绑定在 change 事件上
    - `.number` 将输入内容转为数值
    - `.trim` 自动过滤字符串前后的空格

2. `v-on` 的修饰符
    - `.stop` 阻止事件冒泡
    - `.prevent` 阻止当前事件的默认行为
    - `.self` 当触发事件的元素为绑定事件元素本身时处罚回调
    - `.once` 绑定的事件只会被触发一次

3. 按键修饰符
    监听键盘事件时，我们经常需要监测常见的键值
    ```html
    <!-- 例如 enter 键，keyCode 为13 -->
    <input @keyup.13="enterHandler">

    <!-- Vue 为部分按键提供了别名，13 == enter -->
    <input @keyup.enter="enterHandler">
    ```

可以通过全局 `config.keyCodes` 对象自定义按键修饰符别名：
```javascript
Vue.config.keyCodes.f2 = 113;
```

自定义指令
====================================

[官网描述](https://cn.vuejs.org/v2/guide/custom-directive.html), 其中较为重要的是钩子函数以及其参数。这里简单说一下指令可以注册全局指令，也可以组册局部指令。

> **PS:** 指令是一个特别重要的点，之前在使用 `angular` 时感觉特别明显，很多时候需要自定义指令去实现一些功能，我这里并没有仔仔细细的详述，还是遇到了再说吧！之后可能会补充一点。

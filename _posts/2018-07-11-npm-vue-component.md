---
layout: post
title:  封装 Vue 组件，并使用 npm 发布
date:   2018-07-11 11:45:30 +0800
categories: Vue
tag: note
---

* content
{:toc}

[源码地址](https://github.com/frankxjkuang/custom-ui)，如果对你有帮助的话希望不要吝啬你的 Star

本文主要记录一下如何基于 `Vue` 开发组件，并在 [npm](https://www.npmjs.com/) 上发布。废话不多说，进入正题

# Vue 开发插件

开发之前先看看官网的 [开发规范](https://cn.vuejs.org/v2/guide/plugins.html#%E5%BC%80%E5%8F%91%E6%8F%92%E4%BB%B6) 

我们开发的之后期望的结果是支持 import、require 或者直接使用 script 标签的形式引入，就像这样：

```js
// 这里注意一下包的名字前缀是 custom ，组件的名字前缀是 moor
// 这是因为那个名字发布包的时候被占用了（我做实验的时候叫 moor-ui）现在改成了custom-ui，但是组件的前缀懒得改
import CustomUI from 'custom-ui';

// 或者 const CustomUI = require('custom-ui');

// 或者 <script src="..."></script>

Vue.use(CustomUI);
```

# 构建一个 Vue 项目

开发组件我们使用 `webpack-simple` ：

```bash
vue init webpack-simple <project-name>
```

> **PS:** 这里我选择了 use sass 因为，之后开发组件会用到

开发组件的文件结构如下，参考了一下 [element](https://github.com/elemefe) 不过我们这个是简易版，仅供分享和自己使用

```bash
.
├── src/                           // 源码目录
│   ├── packages/                  // 组件目录
│   │   ├── switch/                // 组件（以switch为例）
│   │   ├── moor-switch.vue        // 组件代码
│   │   ├── index.js               // 挂载插件
│   ├── App.vue                    // 页面入口
│   ├── main.js                    // 程序入口
│   ├── index.js                   // （所有）插件入口
├── index.html                     // 入口html文件
.
```

好了，到这里准备工作做好了，我们可以开始开发组件了，接着上面的例子，下面开始开发一个 `switch` 组件。

# 开发单个组件

先看一下目标效果：

![switch.gif](https://upload-images.jianshu.io/upload_images/2669310-780d8221c1db6252.gif?imageMogr2/auto-orient/strip)

开始开发：在 packages 文件夹下新建一个 switch 文件夹用来存放 switch 组件的源码，继续在 switch 文件夹中新建 moor-switch.vue 和 index.js 文件

## moor-switch.vue

这个文件是组件源码，我这里就不放源码了，这里就说一下我个人认为最重要的点吧，这也是封装表单类组件最为重要的点：

自定义组件绑定 v-model，[官网地址](https://cn.vuejs.org/v2/guide/components-custom-events.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BB%84%E4%BB%B6%E7%9A%84-v-model) 

使用：
```html
<!-- 使用父组件的值绑定 -->
<!-- isSwitch = false -->
<moor-switch 
  v-model="isSwitch">开关:
</moor-switch>

<!-- 子组件必须要有 input 来处理对应的值 -->
<!-- 其中最重要的就是需要 :value="value" 用来绑定值 -->
<!-- @change="$emit('input', $event.target.value)" 事件触发改变值 -->
<input
    type="checkbox"
    @change="$emit('input', $event.target.value)"
    :true-value="activeValue"
    :false-value="inactiveValue"
    :disabled="disabled"
    :value="value">

<!-- 当然还需要使用 props 来接受这个 value -->
<script> 
// ... 此处省略代码    
props: {
  value: {
    type: [Boolean, String, Number],
    default: false
  }
}
// ... 此处省略代码    
</script>    
```

## index.js

这个文件没什么好说的就是将该组件作为 Vue 插件，代码就三行这里就放在这吧：
```js
// MoorSwitch 是对应组件的名字，要记得在 moor-switch.vue 文件中还是 name 属性哦
import MoorSwitch from './moor-switch';

MoorSwitch.install = Vue => Vue.component(MoorSwitch.name, MoorSwitch);

export default MoorSwitch;
```

好了基本完成了，但是为了将所有的组件集中起来比如我还有 `select`、 `input`、 `button` 等组件，那么我想要统一将他们放在一个文件这中便于管理

所以在 App.vue 同级目录我新建了一个 index.js 文件，内容也没啥好说的看看就懂了：
```js
import HelloWorld from './packages/hello-world/index.js';
import MoorSwitch from './packages/switch/index.js';
// ...如果还有的话继续添加

const components = [
  HelloWorld,
  MoorSwitch
  // ...如果还有的话继续添加
]

const install = function(Vue, opts = {}) {
  components.map(component => {
    Vue.component(component.name, component);
  })
}

/* 支持使用标签的方式引入 */
if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue);
}

export default {
  install,
  HelloWorld,
  MoorSwitch
  // ...如果还有的话继续添加
}
```

本地运行通过 `<script/>` 标签的方式使用，修改 `index.html` 文件：
```html
<!-- 省略部分代码 -->
<div id="app">
  <moor-hello-world :color="color" :msg="msg"></moor-hello-world>
  <moor-switch
  v-model="lightSwitch">开关:</moor-switch>
</div>
<script src="./node_modules/vue/dist/vue.js"></script>
<script src="/dist/custom-ui.js"></script>
<script>
new Vue({
  el: '#app',
  data() {
    return {
      color: 'red',
      msg: 'hello world!',
      lightSwitch: false
    }
  }
})
</script>
```

然后运行 `npm run dev` 你就可以看到效果了:

![preview.png](https://upload-images.jianshu.io/upload_images/2669310-de6fe94ad7885cde.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

好了到这里我们的组件就开发完成了；下面开始说怎么打包发布到 npm 上

# 发布到 npm 

打包之前，首先我们需要改一下 `webpack.config.js` 这个文件;

```js
// ... 此处省略代码 
// 执行环境
const NODE_ENV = process.env.NODE_ENV

module.exports = {
  // 根据不同的执行环境配置不同的入口
  entry: NODE_ENV == 'development' ? './src/main.js' : './src/index.js',
  output: {
    // 修改打包出口，在最外级目录打包出一个 index.js 文件，我们 import 默认会指向这个文件
    path: path.resolve(__dirname, './dist'),
    publicPath: '/dist/',
    filename: 'custom-ui.js',
    library: 'custom-ui', // 指定的就是你使用require时的模块名
    libraryTarget: 'umd', // libraryTarget会生成不同umd的代码,可以只是commonjs标准的，也可以是指amd标准的，也可以只是通过script标签引入的
    umdNamedDefine: true // 会对 UMD 的构建过程中的 AMD 模块进行命名。否则就使用匿名的 define
  },
  // ... 此处省略代码 
}
```

修改 `package.json` 文件：
```js
// 发布开源因此需要将这个字段改为 false
"private": false,

// 这个指 import custom-ui 的时候它会去检索的路径
"main": "dist/custom-ui.js",
```

发布命令其实就是两句话
```js
// 这里需要你有一个 npm 的账号，文章开头有官网链接
npm login   // 登陆 
npm publish // 发布
```

完成之后我们就可以在项目中安装使用了
```js
npm install custom-ui -S 
```

在 `main.js` 中引入插件
```js
import CustomUI from 'custom-ui'
Vue.use(CustomUI)
```

在组件中使用：
```html
<!-- 直接使用脚手架的HelloWorld组件 -->
<!-- 此处有省略代码，看对地方加入代码哦 -->
<div class="moor-item">
  <label>Input: </label>
  <moor-input
  v-model="input1"
  placeholder="请输入信息">
  </moor-input>

  <moor-input
    v-model="input2"
    placeholder="请输入信息">
  </moor-input>

  <moor-input
    placeholder="输入框禁用"
    :disabled="inputDisabled">
  </moor-input>
</div>

<div class="moor-item">
  <label>Switch: </label>

  <moor-switch
  v-model="lightSwitch">开关(开):</moor-switch>

  <moor-switch
  v-model="switchLight">开关(关):</moor-switch>
</div>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      // HelloWorld
      msg: 'Welcome to moor UI!',
      color: 'red',
      // input
      input1: '',
      input2: '这是默认值',
      inputDisabled: true,
      // switch
      lightSwitch: false,
      switchLight: true
    }
  },
  watch: {
    lightSwitch: newValue => console.log('开关：', newValue),
  }
}
</script>

<style scoped>
.moor-select, .moor-btn, .moor-switch, .moor-input {
  margin: 10px 6px;
}
.moor-item {
  display: flex;
  align-items: center;
}
.moor-item label {
  width: 100px;
  display: inline-block;
}
</style>
```

预览效果如下:

![test-preview.png](https://upload-images.jianshu.io/upload_images/2669310-6eeb20be5bf621cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**PS:** 修改 .gitignore 去掉忽略dist,因为我们打包的文件也需要提交；每次上到npm上需要更改版本号，package.json 里的 version 字段


写的比较简单，主要还是提供思路。用习惯了开源的组件自己总得了解一下嘛，有的时候在开发的过程中我们找不到合适的开源组件就需要自己开发了，这个时候我们把自己写的一些精致的小插件开源出来挺好的...

最后希望你给个 Star [源码地址](https://github.com/frankxjkuang/custom-ui)

哦，对了README，不想写了...哈哈
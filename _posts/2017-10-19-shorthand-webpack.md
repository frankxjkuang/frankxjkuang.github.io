---
layout: post
title:  速记一下快速入门webpack
date:   2017-10-03 12:54:00 +0800
categories: document
tag: course
---

* content
{:toc}


webpack作为第三方包的管理工具以及项目打包工具等，是前端必须掌握的必不可少的工具之一,下面速记一下如何快速入门webpack。

Webpack和Grunt以及Gulp相比
====================================

Grunt和Gulp的工作方式是：在一个配置文件中，指明对某些文件进行类似编译，组合，压缩等任务的具体步骤，工具之后可以自动替你完成这些任务，类似于处理文件的流水线。

Webpack的工作方式是：把你的项目当做一个整体，通过一个给定的主文件（如：index.js），Webpack将从这个文件开始找到你的项目的所有依赖文件，使用loaders处理它们，最后打包为一个（或多个）浏览器可识别的JavaScript文件。

总结一下：Gulp/Grunt是一种能够优化前端的开发流程的工具，而Webpack是一种模块化的解决方案，不过Webpack的优点使得Webpack在很多场景下可以替代Gulp/Grunt类的工具。

使用Webpack开始一个小的项目搭建
====================================

正式使用Webpack前的准备
------------------------------------

1. 创建一个项目文件夹，在其中创建一个package.json文件，这是一个标准的npm说明文件， 面蕴含了丰富的信息，包括当前项目的依赖模块，自定义的脚本任务等等。

```git
# 在终端中使用npm init命令可以自动创建package.json文件
npm init
```

> **PS:**输入这个命令后，终端会问你一系列诸如项目名称，项目描述，作者等信息，不过不用担心，如果你不准备在npm中发布你的模块，这些问题的答案都不重要，回车默认即可。但是项目名称不可以为webpack(Refusing to install package with name "webpack" under a package npm ERR! also called "webpack");

2. package.json文件已经就绪，我们在本项目中安装Webpack作为依赖包

```git
# 全局安装
npm install webpack -g 
# 安装到你的项目目录
npm install webpack --save-dev
```

> **PS:**--save-dev是指将包信息添加到package.json里的devDependencies节点，表示开发时依赖的包;--save是指将包信息添加到package.json里的dependencies节点，表示发布时依赖的包。

3. 回到项目根文件夹，并在里面创建两个文件夹,res文件夹和public文件夹，res文件夹用来存放原始数据和我们将写的JavaScript模块，public文件夹用来存放之后供浏览器读取的文件（包括使用webpack打包生成的js文件以及一个index.html文件）。接下来我们再创建三个文件:

- index.html --放在public文件夹中;

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
</body>
<script src="bundle.js"></script>
</html>
```

- Hello.js-- 放在res文件夹中;

```javascript
module.exports = function() {
  const hello = document.createElement('h1');
  hello.textContent = "Hello world!";
  return hello;
};
```

- main.js-- 放在res文件夹中;

```javascript
const hello = require('./Hello.js');
document.querySelector("body").appendChild(hello());
```

> **PS:**在index.html文件中引入了打包后的js文件（这里我们先把之后打包后的js文件命名为bundle.js，之后我们还会详细讲述）。

正式使用Webpack
------------------------------------

```git
# {extry file}出填写入口文件的路径，本文中就是上述main.js的路径，
# {destination for bundled file}处填写打包文件的存放路径
# 填写路径的时候不用添加{}
# webpack {entry file} {destination for bundled file}
webpack res/main.js public/bundle.js
```

> **PS:**webpack非全局安装的情况```node_modules/.bin/webpack app/main.js public/bundle.js```

好了现在已经实现了文件打包，但是这样使用命令对文件进行打包是非常不方便的，webpack提供了非常方便的操作方式，下面就来说一说如何使用配置文件进行更加方便的操作。

通过配置文件来使用Webpack
------------------------------------

在当前项目根目录下新建一个名为webpack.config.js的文件，我们在其中写入如下所示的简单配置代码，目前的配置主要涉及到的内容是入口文件路径和打包后文件的存放路径。

```javascript
module.exports = {
  entry:  __dirname + "/res/main.js",  // 项目的唯一入口文件
  output: {
    path: __dirname + "/public",  // 打包后的文件存放的路径
    filename: "bundle.js",  // 打包后输出文件的文件名
  },
}
```

> **PS:**__dirname"是node.js中的一个全局变量，它指向当前执行脚本所在的目录。

```git
# 现在你只需要运行以下命令就可以实现打包
webpack

# 非全局安装需使用
node_modules/.bin/webpack
```

如果连以上这条命令都可以不用，那种感觉会不会更爽~，别急还有更人性化的操作，在package.json中增加如下代码

```javascript
"scripts": {
    "start": "webpack"
}
```

> **PS:**json文件不支持注释的哦！package.json中的script会安装一定顺序寻找命令对应位置，本地的node_modules/.bin路径就在这个寻找清单中，所以无论是全局还是局部安装的Webpack，你都不需要写前面那指明详细的路径了。此外npm的start命令是一个特殊的脚本名称，其特殊性表现在，在命令行中使用npm start就可以执行其对于的命令，如果对应的此脚本名称不是start，想要在命令行中运行时，需要这样用npm run {script name}

devtool生成Source Maps（使调试更容易）
------------------------------------

| options       |      配置结果  |   
| ------------- | :--------------| 
| source-map | 在一个单独的文件中产生一个完整且功能完全的文件。这个文件具有最好的source map，但是它会减慢打包速度 | 
| cheap-module-source-map | 在一个单独的文件中生成一个不带列映射的map，不带列映射提高了打包速度，但是也使得浏览器开发者工具只能对应到具体的行，不能对应到具体的列（符号），会对调试造成不便 | 
| eval-source-map  | 使用eval打包源文件模块，在同一个文件中生成干净的完整的source map。这个选项可以在不影响构建速度的前提下生成完整的sourcemap，但是对打包后输出的JS文件的执行具有性能和安全的隐患。在开发阶段这是一个非常好的选项，在生产阶段则一定不要启用这个选项 | 
| cheap-module-eval-source-map | 这是在打包文件时最快的生成source map的方法，生成的Source Map 会和打包后的JavaScript文件同行显示，没有列映射，和eval-source-map选项具有相似的缺点 | 

上述选项由上到下打包速度越来越快，不过同时也具有越来越多的负面作用，较快的打包速度的后果就是对打包后的文件的的执行有一定影响。对小到中型的项目在开发阶段eval-source-map是一个很好的选项

```javascript
module.exports = {
  entry:  __dirname + "/app/main.js",
  output: {
    path: __dirname + "/public",
    filename: "bundle.js"
  }
  devtool: 'eval-source-map',
}
```

webpack构建本地服务器（devserver）
------------------------------------

> 让浏览器监听代码的修改，并自动刷新显示修改后的结果，其实Webpack提供一个可选的本地开发服务器，这个本地服务器基于node.js构建，可以实现你想要的这些功能，不过它是一个单独的组件，在webpack中进行配置之前需要单独安装它作为项目依赖

```git
npm install wabpack-dev-server --save-dev
```

devserver配置项[更多]: https://webpack.js.org/configuration/dev-server/
------------------------------------

| options      |      描述      |
| ------------- | :--------------| 
| contentBase  | 默认webpack-dev-server会为根文件夹提供本地服务器，如果想为另外一个目录下的文件提供本地服务器，应该在这里设置其所在目录（本例设置到“public"目录） | 
| port         | 设置默认监听端口，如果省略，默认为”8080“ | 
| inline       | 设置为true，当源文件改变时会自动刷新页面 | 
| historyApiFallback | 在开发单页应用时非常有用，它依赖于HTML5 history API，如果设置为true，所有的跳转将指向index.html | 

```javascript
// 在 webpack.config.js 中进行以下配置
devServer: {
  contentBase: "./public",  // 本地服务器所加载的页面所在的目录
  historyApiFallback: true,  // 不跳转
  inline: true,  // 实时刷新
  port: 8888  // 监听端口
}   

// 在 package.json 中进行以下配置
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "start": "webpack",
  "server": "webpack-dev-server --open"
}, 
```

Loaders
------------------------------------

Loaders是webpack提供的最激动人心的功能之一了。通过使用不同的loader，webpack有能力调用外部的脚本或工具，实现对不同格式的文件的处理，比如说分析转换scss为css，或者把下一代的JS文件（ES6，ES7)转换为现代浏览器兼容的JS文件，对React的开发而言，合适的Loaders可以把React的中用到的JSX文件转换为JS文件。

Loaders需要单独安装并且需要在webpack.config.js中的modules关键字下进行配置，Loaders的配置包括以下几方面：

- test：一个用以匹配loaders所处理文件的拓展名的正则表达式（必须）
- loader：loader的名称（必须）
- include/exclude:手动添加必须处理的文件（文件夹）或屏蔽不需要处理的文件（文件夹）（可选）；
- query：为loaders提供额外的设置选项（可选）

Babel
------------------------------------

Babel其实是一个编译JavaScript的平台，它的强大之处表现在可以通过编译帮你达到以下目的：
- 使用下一代的JavaScript代码（ES6，ES7...），即使这些标准目前并未被当前的浏览器完全的支持；
- 使用基于JavaScript进行了拓展的语言，比如React的JSX；

安装与配置：
Babel其实是几个模块化的包，其核心功能位于称为babel-core的npm包中，webpack可以把其不同的包整合在一起使用，对于每一个你需要的功能或拓展，你都需要安装单独的包（用得最多的是解析Es6的babel-preset-es2015包和解析JSX的babel-preset-react包）。

```git
# 安装
npm install --save-dev babel-core babel-loader babel-preset-es2015 babel-preset-react

# 继续安装 React 和 React-DOM进行以下实验
npm install --save react react-dom
```

```javascript
// 在webpack中配置Babel的方法如下:
module.exports = {
    entry: __dirname + "/app/main.js",
    output: {
        path: __dirname + "/public",
        filename: "bundle.js"
    },
    devtool: 'eval-source-map',
    devServer: {
        contentBase: "./public",
        historyApiFallback: true,
        inline: true,
        port: 8888
    },
    module: {
        rules: [
            {
                test: /(\.jsx|\.js)$/,
                use: {
                    loader: "babel-loader",
                    options: {
                        presets: [
                            "es2015", "react"
                        ]
                    }
                },
                exclude: /node_modules/
            }
        ]
    }
};
```

```javascript
// 在 res 文件中新建一个 config.json文件并写入
{
  "helloText": "Hello world!"
}

// Hello 文件如下
import config from './config.json';
import React, {Component} from 'react';

class Hello extends Component {
  render() {
    return (<h1>{config.helloText}</h1>);
  }
}
export default Hello;

// main 文件如下
import React from 'react';
import {render} from 'react-dom';
import Hello from './Hello';

render(<Hello />, document.getElementsByTagName('body')[0]);
```

之后再进行打包，启动服务就可以看到效果了。

一切皆模块
====================================

Webpack把所有的文件都都当做模块处理，JavaScript代码，CSS，fonts以及图片等等通过合适的loader都可以被处理。

CSS
------------------------------------

webpack提供两个工具处理样式表，`css-loader` 和 `style-loader`
,二者处理的任务不同，`css-loader`使你能够使用类似`@import`和`url(...)`的方法实现`require()`的功能,`style-loader`将所有的计算后的样式加入页面中，二者组合在一起使你能够把样式表嵌入webpack打包后的JS文件中。

```git
# 安装
npm install style-loader css-loader --save-dev
```

```javascript
// 使用
module.exports = {
   ...
    module: {
        rules: [
            {
                test: /(\.jsx|\.js)$/,
                use: {
                    loader: "babel-loader"
                },
                exclude: /node_modules/
            },
            {
                test: /\.css$/,
                use: [
                    {
                        loader: "style-loader"
                    }, {
                        loader: "css-loader"
                    }
                ]
            }
        ]
    }
};
```

之后在 mian.js 中引入
```javascript
import './main.css';
```

> **PS:**通常情况下，css会和js打包到同一个文件中，并不会打包为一个单独的css文件，不过通过合适的配置webpack也可以把css打包为单独的文件的。

CSS module(真正意义上的css模块)
------------------------------------

```javascript
// 使用
module.exports = {
   ...
    module: {
        rules: [
            {
                test: /(\.jsx|\.js)$/,
                use: {
                    loader: "babel-loader"
                },
                exclude: /node_modules/
            },
            {
                test: /\.css$/,
                use: [
                    {
                        loader: "style-loader"
                    }, {
                        loader: "css-loader"，
                        options: {
                          modules: true
                        }
                    }
                ]
            }
        ]
    }
};
```

```css
// 在 res 目录中新建一个 style.css 文件
.redfont {
  font-color: red;
}
```

```javascript
// 在 Hello.js 中使用 css 模块
import config from './config.json';
import React, {Component} from 'react';

import styles from './style.css'

class Hello extends Component {
  render() {
    return (<h1 className={styles.redfont}>{config.helloText}</h1>);
  }
}
export default Hello;
```

CSS 预处理器
------------------------------------

常用的CSS处理loaders：
- Less Loader
- Sass Loader
- PostCss Loader

下面举例来说如何使用PostCSS，我们使用PostCSS来为CSS代码自动添加适应不同浏览器的CSS前缀（顺便安装Scss）：

```git
# 安装
npm install postcss-loader autoprefixer sass-loader node-sass --save-dev 
```

```javascript
// webpack.config.js
module.exports = {
   ...
    module: {
        rules: [
            {
                test: /(\.jsx|\.js)$/,
                use: {
                    loader: "babel-loader"
                },
                exclude: /node_modules/
            },
            {
                test: /(\.scss|css)$/,
                use: [
                    {
                        loader: "style-loader"
                    }, {
                        loader: "css-loader",
                        options: {
                          modules: true
                       }
                    }, {
                        loader: "sass-loader"
                    }, {
                        loader: "postcss-loader"
                    }
                ]
            }
        ]
    }
};

// 在项目根目录下新建一个 postcss.config.js 内容如下
module.exports = {
    plugins: [
        require('autoprefixer')
    ]
}
```


// 下面我们就可以使用sass语法写css代码了，并且webpack会自动为你加上前缀
```css
// main.css做如下修改(现在也可以使用scss文件编写css代码)
html, body {
  width: 100%;
  height: 100%;
  position: relative;
  h1 {
    color: pink;
    padding-top: 50px;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
}
```

Plugins(插件)
====================================

插件（Plugins）是用来拓展Webpack功能的，它们会在整个构建过程中生效，执行相关的任务。
Loaders和Plugins常常被弄混，但是他们其实是完全不同的东西，可以这么来说，loaders是在打包构建过程中用来处理源文件的（JSX，Scss，Less..），一次处理一个，插件并不直接操作单个文件，它直接对整个构建过程其作用。

Webpack有很多内置插件，同时也有很多第三方插件，可以让我们完成更加丰富的功能。

使用插件
------------------------------------

要使用某个插件，我们需要通过`npm`安装它，然后要做的就是在webpack配置中的plugins关键字部分添加该插件的一个实例（plugins是一个数组）继续上面的例子,

HtmlWebpackPlugin
------------------------------------

这个插件的作用是依据一个简单的index.html模板，生成一个自动引用你打包后的JS文件的新index.html。这在每次生成的js文件名称不同时非常有用（比如添加`hash`值）

```git
# 安装
npm install html-webpack-plugin --save-dev 
```

这个插件自动完成了我们之前手动做的一些事情，在正式使用之前需要对一直以来的项目结构做一些更改：

1. 移除public文件夹中所有的文件，利用此插件，index.html文件会自动生成，此外CSS已经通过前面的操作打包到JS中了。

2. 在app目录下，创建一个index.tmpl.html文件模板，这个模板包含title等必须元素，在编译过程中，插件会依据此模板生成最终的html页面，会自动添加所依赖的 css, js，favicon等文件，index.tmpl.html中的模板源代码如下：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  
</body>
</html>```

3.更新webpack的配置文件，方法同上,新建一个build文件夹用来存放最终的输出文件

```javascript
// webpack.config.js 更改如下
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: __dirname + "/res/main.js",
    output: {
        path: __dirname + "/public",
        filename: "bundle.js"
    },
    devtool: 'eval-source-map',
    devServer: {
        ...
    },
    module: {
        ...
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: __dirname + "/res/index.tmpl.html" // new 一个这个插件的实例，并传入相关的参数
        })
    ],
};
```

再次执行npm start你会发现，public文件夹下面生成了bundle.js和index.html。

Hot Module Replacement
------------------------------------

Hot Module Replacement（HMR）也是webpack里很有用的一个插件，它允许你在修改组件代码后，自动刷新实时预览修改后的效果。

在webpack中实现HMR也很简单，只需要做两项配置:

```javascript
// webpack.config.js 更改如下
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    ...
    devServer: {
        ...
        // 1.在Webpack Dev Server中添加“hot”参数
        hot: true
    },
    module: {
        ...
    },
    plugins: [
        ...
        // 2.在webpack配置文件中添加HMR插件
        new webpack.HotModuleReplacementPlugin()  // 热加载插件
    ],
};
```

补充
====================================

在开发中还会用到很多其它的东西
比如以下插件:
- OccurenceOrderPlugin :为组件分配ID，通过这个插件webpack可以分析和优先考虑使用最多的模块，并为它们分配最小的ID
- UglifyJsPlugin：压缩JS代码；
- ExtractTextPlugin：分离CSS和JS文件

还有打包图片的 `url-loader`等。

为了避免缓存引起的问题，在打包文件时通常也会在文件名后加上`hash`值

就说这么多吧，基本的使用应该没问题了。
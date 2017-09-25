---
layout: post
title:  webpack配置多项目下多入口
date:   2017-09-16 11:29:00 +0800
categories: document
tag: log
---

* content
{:toc}


>最近做了一个项目在这里写速记一下

+ 该项目是多个官网项目
+ 使用一个仓库，多人联合开发
+ 需要webpack配置项目的多个入口
+ 提取webpack的公共部分

文件结构就不提了配置中的路径很明了
------------------------------------

> 之后也会写一些webpack相关的知识

公共webpack的配置
====================================

```javascript
const path = require('path');
const webpack = require('webpack');
const ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new ExtractTextPlugin('themes/mobile/css/[name].css?[contenthash:8]', {
      // allChunks: true
    }),
  ],
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /(node_modules)/,
        use: {
          loader: 'babel-loader',
          options: {
            cacheDirectory: true,
            presets: [['es2015', {modules: false}]],
            plugins: ['syntax-dynamic-import', 'transform-runtime'],
          },
        },
      },
      {
        test: /\.html$/,
        use: [{
          loader: 'html-loader',
          options: {
            minimize: false,
            interpolate: true,
          },
        }],
      },
      {
        test: /\.(scss|css)$/,
        use: ExtractTextPlugin.extract({
          fallback: 'style-loader',
          use: [
            {
              loader: 'css-loader',
              options: {
                minimize: true,
                import: true,
              },
            },
            {
              loader: 'sass-loader',
              options: {
                import: true,
              },
            },
          ],
        }),
      },
      {
        test: /\.(png|jpg|gif|svg|eot|ttf|woff|woff2)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10000,
              useRelativePath: false,
              name: '[name].[ext]?[hash:8]',
              outputPath: 'themes/mobile/assets/',
              publicPath: '/',
            },
          },
        ],
      },
    ],
  },
};
```

单个项目的webpack配置
====================================

```javascript
const path = require('path');
const merge = require('webpack-merge');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const commonWebpackConfig = require('../../../scripts/webpack.config.common');
const webpack = require('webpack');
const CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;

const webpackConfig = {
  context: __dirname,
  entry: {
    index: ['./js/index.js'],
    newsList: ['./js/newsList.js'],
    newsDetail: ['./js/newsDetail.js'],
    aboutUs: ['./js/aboutUs.js'],
    error404: ['./js/error404.js'],
  },
  output: {
    path: path.resolve(__dirname, '../../../dist/RavvChina'),
    filename: 'themes/mobile/js/[name].js?[hash:8]',
  },
  devServer: {
    contentBase: path.join(__dirname, '../../../dist/RavvChina'),
    port: 7001,
  },
  plugins: [
    new CommonsChunkPlugin({
      name: 'common',
      chunks: ['index', 'newsList', 'newsDetail', 'aboutUs', 'error404'],
      // minChunks是指一个文件至少被require几次才会被放到CommonChunk里，如果minChunks等于2，说明一个文件至少被require两次才能放在CommonChunk里
      minChunks: 5 // 提取所有chunks共同依赖的模块
    }),
    new HtmlWebpackPlugin({
      filename: 'error404.html',
      template: 'error404.html',
      chunks: ['common', 'error404'],
    }),
    new HtmlWebpackPlugin({
      filename: 'newsList.html',
      template: 'newsList.html',
      chunks: ['common', 'newsList'],
    }),
    new HtmlWebpackPlugin({
      filename: 'newsDetail.html',
      template: 'newsDetail.html',
      chunks: ['common', 'newsDetail'],
    }),
    new HtmlWebpackPlugin({
      filename: 'aboutUs.html',
      template: 'aboutUs.html',
      chunks: ['common', 'aboutUs'],
    }),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      chunks: ['common', 'index'],
    }),
  ],
};

module.exports = merge(commonWebpackConfig, webpackConfig);
```

依赖的包
====================================

```json
{
  "name": "xxxx",
  "version": "1.0.0",
  "description": "xxxx official website",
  "main": "",
  "scripts": {
    "build": "dar-clean-dist && webpack --config ./src/pc/darHolo/webpack.config.js && webpack --config ./src/pc/ravvchina/webpack.config.js && webpack --config ./src/pc/ravvar/webpack.config.js",
    "clean": "dar-clean-dist",
    "start-darholo": "webpack-dev-server --inline --open --config ./src/pc/darHolo/webpack.config.js",
    "start-ravv": "webpack-dev-server --inline --open --config ./src/pc/ravvchina/webpack.config.js",
    "start-ravvar": "webpack-dev-server --inline --open --config ./src/pc/ravvar/webpack.config.js"
  },
  "repository": {
    "type": "git",
  },
  "author": "xxxx Frontend Technical Team",
  "license": "proprietary",
  "homepage": "",
  "dependencies": {
    "babel-polyfill": "^6.23.0",
    "babel-runtime": "^6.23.0",
    "jquery": "~3.2.1",
    "jquery-validation": "^1.17.0",
    "swiper": "^2.7.6"
  },
  "devDependencies": {
    "babel-core": "^6.24.1",
    "babel-eslint": "~7.2.3",
    "babel-loader": "^7.0.0",
    "babel-plugin-syntax-dynamic-import": "^6.18.0",
    "babel-plugin-transform-runtime": "^6.23.0",
    "babel-preset-es2015": "^6.24.1",
    "css-loader": "~0.28.4",
    "eslint": "~3.19.0",
    "eslint-config-airbnb-base": "~11.2.0",
    "eslint-plugin-import": "^2.2.0",
    "extract-text-webpack-plugin": "^3.0.0",
    "file-loader": "~0.11.2",
    "html-loader": "~0.4.5",
    "html-webpack-plugin": "^2.30.1",
    "node-sass": "~4.5.3",
    "sass-loader": "~6.0.5",
    "style-loader": "~0.18.1",
    "url-loader": "~0.5.8",
    "webpack": "^3.5.0",
    "webpack-merge": "^4.1.0",
    "webpack-dev-server": "~2.7.0"
  }
}
```

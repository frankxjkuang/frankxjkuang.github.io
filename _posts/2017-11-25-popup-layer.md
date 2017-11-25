---
layout: post
title:  Popup-layer 自定义的弹出框
date:   2017-11-25 13:12:00 +0800
categories: document
tag: log
---

* content
{:toc}


写在前面
====================================

浏览器原生的提示框不满足需要，于是自己写了一个，发布在npm上...


# popup-layer

About
====================================

**Popup-layer** is a pop-up plugin,it exposes a base class outwards, which means that you can completely customize attributes and methods by instantiating them.**Popup-layer** is written through native JS, independent of the third party library.

Install
====================================

```git
npm install popup-layer
```

Import
====================================

```javascript
import popup from 'popup-layer';
```

Instantiation
====================================

**Popup-layer** provides a static method to customize configuration.

```javascript
// here is a simple example
popup.open({
  title: 'Title',
  content: 'Hello Popup-Layer!',
});
```

Properties
====================================

- title {string}: message header
- content {string}: message content
- time {number}: popup-layer auto shut down time(millisecond)
- popupHeader {object}: custom message header styles
- popupContent {object}: custom message content styles
- popupFooter {object}: custom message footer styles

> In addition, you can directly copy `popup-layer` styles, such as fontSize:'16px'

Methods
====================================

- btnCancel: callback method for cancel button
- btnClose: callback method for close button
- btnOk: callback method for ok button
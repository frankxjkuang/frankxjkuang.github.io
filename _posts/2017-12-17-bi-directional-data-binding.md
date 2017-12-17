---
layout: post
title:  "自定义实现双向数据绑定"
date:   2017-12-17 17:05:30 +0800
categories: document
tag: course
---

* content
{:toc}


> 今天一个社区的朋友问我如何自定义实现双向数据绑定！解答那位朋友疑惑的同时我也顺便写篇文章记录一下这个小的知识点，这里记录一种个人认为最容易想到的一种。


Bi-directional data binding（双向数据绑定）
====================================

> 即数据模型（Module）和视图（View）之间的双向绑定，我最开始接触到双向数据绑定绑定是从angualrJS开始的，...扯远了进入正题

准备工作
------------------------------------

先获取到 Modele 和 View 对象
```javascript
let data = {}; // 待绑定的数据模型 -> Module
let aEle = document.getElementsByTagName('*'); // 待绑定的元素 -> View
```

Module binding View
------------------------------------

> 元素增加的 module 的属性为双向数据绑定的变量 

```javascript
function apply() {
	Array.from(aEle).forEach(ele => {
		let model = ele.getAttribute('model');

		if (model) { // 该元素有 module 属性即进行了双向数据绑定
			if (data[model]) {
				ele.value = data[model];
			} else {
				ele.value = ''; // 未绑定或数据模型中没有该值即返回''
			}
		}
	});
}
```

View binding Module 
------------------------------------

```javascript
Array.from(aEle).forEach(ele => {
    let model = ele.getAttribute('model');

    if (model) {
        ele.oninput = function() {
            data[model] = this.value;
        	apply();
        };
    }
});
```

效果
====================================

本来打算弄一张gif图片的，肚子饿了吃饭去了先！放几个元素加装有效果，就到这吧！
```html
<input type="text" model="test">
<select model="test">
	<option value="1">select 1</option>
	<option value="2">select 2</option>
	<option value="3">select 3</option>
</select>
```



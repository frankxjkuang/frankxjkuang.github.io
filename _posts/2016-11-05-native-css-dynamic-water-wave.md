---
layout: post
title:  css实现动态波浪
date:   2016-11-05 02:16:00 +0800
categories: document
tag: 教程
---

* content
{:toc}


> 在迅雷，360等的小悬浮窗的动态波浪效果，最近在网上看到了一个用纯css动画实现的方法，非常的巧妙，这里分享出来记录一下。至于效果图嘛，懒得截图了自己脑补吧！

HTML 结构
====================================

```html
<div class="container">
	<div class="wave"></div>
</div>
```

css 样式
====================================

```css
.container { 
	overflow: hidden;
}
.wave {
	position: relative;
	height: 200px;
	width: 200px;
	background: #0ee251;
	border-radius: 50%;
}
.wave:before, .wave:after {
	content: '';
	position: absolute;
	width: 400px;
	height: 400px;
	top: 0;
	left: 50%;
	background: rgba(255, 255, 255, .4);
	border-radius: 46%;
	transform: translate(-50%, -70%) rotate(0);
	animation: rotate 6s linear infinite;
	z-index: 10;
}
.wave:after {  
    border-radius: 47%;  
    background-color: rgba(255, 255, 255, .8);  
    animation: rotate 8s linear -5s infinite;  
    z-index: 20;  
}  
@keyframes rotate {  
    50% {  
        transform: translate(-50%, -73%) rotate(180deg);  
    } 100% {  
        transform: translate(-50%, -70%) rotate(360deg);  
    }  
} 
```


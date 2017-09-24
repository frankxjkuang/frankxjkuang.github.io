---
layout: post
title:  回到顶部按钮的封装
date:   2017-09-16 12:35:00 +0800
categories: document
tag: course
---

* content
{:toc}


网页回到顶部按钮的封装
====================================

这是一个网页中常见的功能按钮，当网页内容过长时，用户滚动到一定高度时再想回到
顶部，显然再滚动回去就显得不够人性化，所以回到顶部这个功能就孕育而生了。下面
将实现该功能。

功能需求
------------------------------------

+ 圆形按钮，鼠标移入时背景改变
+ 滚动到超过`banner`的1.5倍时显示回到顶部按钮
+ 该按钮位置固定到窗口右下角`(5%, 5%)`的位置
+ 当出现`footer`的时候该按钮保持在其上方`5%`的位置
+ 回到顶部时间固定`800ms`

> 改插件使用了`jquery`主要是为了方便写JS代码，当然这个是完全没有必要的
> 根据需要使用原生JS替换jquery部分代码即可

js 部分
====================================

文字解释和原理尽量写在注释里了，就不单独写了。

```javascript
import $ from 'jquery';

export default {
  toTop: function () {
    // footer 到文档顶部的高度
    const footerTop = $('footer').offset().top;
    // footer 元素的高度
    const footerHeight = $('footer').height();
    // 窗口的高度
    const windowHeight = $(window).height();

    // 回到顶部的按钮，将会动态插入到 dom 中
    const $toTop = $('<a href="javascript:;" class="to-top"></a>');
    $('body').on('click', '.to-top', () => {
      // 此处使用同时写了body和html是为了兼容Firefox和Chrome
      $('body, html').animate({ scrollTop: 0 }, 800);
    });

    // 监听滚动事件
    $(window).scroll(() => {
      // 窗口滚动的高度
      const scrollTop = $(window).scrollTop();
      // banner 元素的高度
      const bannerHeight = $('.banner').height();

      if (scrollTop > bannerHeight * 1.5) { // 超过 banner 高度的1.5倍且回到顶部按钮不存在即插入该回到顶部按钮
        if (!$('.to-top').length) {
          $toTop.appendTo('body');
        }
      } else { // 反之移除
        $toTop.remove();
      }

      if ($('.to-top').length > 0) {
        // 滚动高度加窗口高度之和大于 footer 的高度让按钮保持在 footer 上方
        if (scrollTop + windowHeight > footerTop) {
          // 定位采用绝对定位，其高度为 footer 的高度加上窗口的 5% 
          // 视觉上与之前的位置没有变化，会随着滚动而移动
          $('.to-top').css({
            position: 'absolute',
            bottom: `${footerHeight + windowHeight * .05}px`,
          });
        } else {
          // 反之使用固定定位
          $('.to-top').css({
            position: 'fixed',
            bottom: '5%',
          });
        }
      }
    });
  },
}
```

css 部分
====================================

```css
// 这里是一些基础的样式部分 
.to-top {
  height: 50px;
  width: 50px;
  position: fixed;
  right: 5%;
  bottom: 5%;
  border-radius: 50%;
  box-shadow: 0 2px 18px rgba(0, 0, 0, .2);
  background: rgba(255, 255, 255, .8) center center 45% no-repeat url('./images/back_top.svg');
  z-index: 999;

  &:hover {
    background-color: rgba(0, 0, 0, .05);
  }
}
```

> **PS:**一套非常常规的思路一个简单的回到顶部功能就封装好了，当然这个还是很粗糙
> 的，只是进行了简单的功能实现。更近一步的优化，可以把按钮出现的滚动高度，回到
> 顶部的动画时间，定位的参数等等作为函数的参数这样就可以实现一个完全自定义的回
> 到顶部按钮，想要更高质量优美简洁的代码就赶紧动手优化吧


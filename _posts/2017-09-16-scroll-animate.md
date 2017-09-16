---
layout: post
title:  自定义动画的实现
date:   2017-09-16 12:35:00 +0800
categories: document
tag: 教程
---

* content
{:toc}


自定义动画
====================================

> 通过监听窗口的滚动，实现在进入页面的不同模块触发动画
> 当窗口高度 + 窗口滚动高度 > 待播放动画的元素offset().top时触发动画

动画参数
------------------------------------

| 元素属性      |      描述      |    备注    |
| ------------- | --------------:| :---------:|
| ani           | 动画元素的class| 必须       |
| data-animation| 动画的方式     | 可以自定义 |
| data-timeout  | 动画延迟       | 单位毫秒   |
| ...           | ...            | ...        |

js 部分
====================================

```javascript
import $ from 'jquery';

export default {
  onScrollAnimate: function() {
    const aniHeight = $(window).height();

    $(window).on('scroll', function () {
      const scrollTop = $(this).scrollTop();
      const scrollLeft = $(this).scrollLeft();
      
      // 向下滚动触发动画
      $('.ani:not(.animated)').each(function () {
        const $this = $(this);
        const offsetTop = $this.offset().top;
        if (scrollTop + aniHeight > offsetTop) {
          if ($this.data('timeout')) {
            setTimeout(() => {
              $this.addClass(`animated ${$this.data('animation')}`);
            }, parseInt($this.data('timeout'), 10));
          } else {
            $this.addClass(`animated ${$this.data('animation')}`);
          }
        }
      });
    });
  }
}
```

css 部分
====================================

```css
// 动画相关 
// 根据以下方式自定义动画方式
.ani {
  opacity: 0;
  opacity: 1\9\0;
}

.animated {
  animation-duration: 1.5s;
  animation-fill-mode: both;
  animation-timing-function: cubic-bezier(0.25, 0.1, 0.25, 0.1);
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translate3d(0, 100%, 0);
  }

  to {
    opacity: 1;
    transform: none;
  }
}

.fadeInUp {
  animation-name: fadeInUp;
}

@keyframes fadeInDown {
  from {
    opacity: 0;
    transform: translate3d(0, -100%, 0);
  }

  to {
    opacity: 1;
    transform: none;
  }
}

.fadeInDown {
  animation-name: fadeInDown;
}

@keyframes fadeInUp-quater {
  from {
    opacity: 0;
    transform: translate3d(0, 25%, 0);
  }

  to {
    opacity: 1;
    transform: none;
  }
}

.fadeInUp-quater {
  animation-name: fadeInUp-quater;
}

@keyframes fadeInDown-quater {
  from {
    opacity: 0;
    transform: translate3d(0, -25%, 0);
  }

  to {
    opacity: 1;
    transform: none;
  }
}

.fadeInDown-quater {
  animation-name: fadeInDown-quater;
}

@keyframes fadeInRight {
  from {
    opacity: 0;
    transform: translate3d(25%, 0, 0);
  }

  to {
    opacity: 1;
    transform: none;
  }
}

.fadeInRight {
  animation-name: fadeInRight;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

.fadeIn {
  animation-name: fadeIn;
}
```

html 部分
====================================

```html
<div class="box ani" data-animation="fadeIn">
  <div class="box-left fl ani" data-animation="fadeInUp" data-timeout="200"></div>

  <div class="box-right fr ani" data-animation="fadeInUp" data-timeout="400"></div>
</div>         
```

> **PS:**现在就可以使用该动画了，而且用户还可以根据需要自定义动画的样式,
> 该动画还存在不足的地方，当用户在某个动画元素处出刷新页面时，会无法正常播放动
> 画的情况，该bug是由于我们的动画实现建立在监听窗口的滚动事件的基础上，也就是只
> 刷新页面不窗口不会触发滚动也就不会触发动画，解决方法可以在刷新的时候手动触发
> 一下动画。



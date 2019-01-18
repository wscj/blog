### 发现问题

![Bug示意图](/wscj/static-resource/blob/master/images/bug.gif?raw=true)

### 问题描述

业务需求是：当鼠标进入一个子项时，这个子项周围出现阴影，并且出现左右切换的按钮，但子项的宽高时保持原来不变的
具体实现sass代码：

```scss
.item {
  position: relative;
  width: 20%;
  height: 15%;
  &:hover {
    box-shadow: 0 3px 16px 0 rgba(36, 39, 65, 0.4);
    .buttons {
      display: block;
    }
  }
  .buttons {
    position: absolute;
    top: 0;
    left: 0;
    display: none;
  }
}
```

这里增加`box-shadow`属性不会影响宽高，`.buttons`元素是脱离文档流的，不会影响其父元素的宽高，所以理论上当鼠标经过时，`.item`元素是不会改变宽高的。实际上也基本不会出现问题，只是当你调整浏览器的宽度到某些特定值时才出现，而这些值在不同的电脑上也是不同的。

期间也尝试过把`.buttons`的子元素以及css属性全部删掉，在`hover`时只改变`.buttons`的`opacity`属性，这样也会出现同样的问题。

### 原因

明显这是浏览器的锅

### 解决方法

chrome的bug我们还真没直接的办法应对，只能尝试用其他的方式来实现业务，避开这个bug。通过一些尝试后发现，发现`hover`的时候改变`visibility`属性就不会出现这问题，所以改成用`visibility`来实现业务即可。

### Chrome版本

70.0.3538.110

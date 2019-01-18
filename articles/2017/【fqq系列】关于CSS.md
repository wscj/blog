本项目作为一个移动端的app，为了适应各种大小不一的屏幕，CSS的长度单位采用`rem`。

> `rem`是一种相对的长度单位，其实际大小与`html`根节点的`font-size`保持相等。

自己折腾的项目，自然没有设计稿，我就假装有设计稿。

## 关于长度的设计

现在很多安卓手机的逻辑像素是360，高与宽比是`16:9`，所以我把设计稿的分辨率定为`1920*1080`，因为我的手机也是这个分辨率，这样在手机上直接截屏出来的图片也是这个分辨率，可直接作为设计稿。这种情况相当于拿9个设备物理像素作为一个`CSS`的`px`像素。很多UI设计师会把设计稿定为`1280*720`，这样是拿4个设备物理像素作为一个`CSS`的`px`像素。这方面相关的知识可以看看[这篇文章](https://github.com/jawil/blog/issues/21)。

由于`rem`是个相对的单位，所以最好在加载`CSS`之前就计算出`html`的`font-size`大小，请看`index.html`文件里的这段代码：

```javascript
window.onresize = function () {
    document.querySelector("html").style.fontSize = document.documentElement.clientWidth / 360 * 312.5 + "%";
}
window.onresize();
```
这里有以下要点：

* `html`默认的`font-size`为`16px`，`16px * 312.5% = 50px`，这是逻辑像素，对应设计稿的`150px`
* `document.documentElement.clientWidth`是设备的逻辑像素宽度，除以360得到的就是每个逻辑像素的宽度

## 单位换算

使用`rem`作为单位，需要对设计稿的`px`做换算，手动计算显得比较麻烦，网上有一种方案是引入`sass`或`less`，写一个函数来做换算，这方法用在本项目上也挺简单，便采用了。新建一个`sass`的文件`./src/assets/sass/function.scss`，在里面定义一个换算的函数：
```scss
@function px2rem($px, $base-font-size: 150px) {
  @return ($px / $base-font-size) * 1rem;
}
```
引用的时候，在组件的样式标签加入`lang="scss"`，然后导入`function.scss`文件，如下：

```html
<style lang="scss" scoped>
  @import '../../assets/sass/function';
  .test {
    width: px2rem(150px);
    height: px2rem(90px);
  }
</style>
```

*注意：px2rem是sass的函数，不能在calc()里使用*

## Sublime Text 高亮配置

习惯了使用`sublime`编辑器，要让`.vue`文件在`sublime`里显示高亮，首先得装个`Vue Syntax Highlight`插件，这样`.vue`文件里的`html`与`js`还有`css`代码都能显示高亮，但`sass`的代码还得再装个`SCSS`高亮插件

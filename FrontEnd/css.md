## IE盒模型和标准盒模型
### 标准盒模型
![](https://mdn.mozillademos.org/files/13647/box-model-standard-small.png)
width和height只包含content。

在IE中使用标准盒模型：
1. 添加文档声明 < !DOCTYPE html >
2. < meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" >

### IE盒模型
![](https://mdn.mozillademos.org/files/13649/box-model-alt-small.png)
width和height包含content、padding和border

在非IE中使用IE盒模型：在CSS中设置box-sizing属性为border-box（默认是content-box）

**最佳实践：**
```css
box-sizing: border-box;
-moz-box-sizing: border-box;
-webkit-box-sizing: border-box;
```
这样可以多浏览器兼容，保持一致。

----------
 
## 元素消失的四种CSS方法
```css
visibility: hidden;
opacity: 0;
position: absolute; left: -1000px;
display: none;
```
~~width: 0; height: 0;~~

~~z-index: -100;~~

----------
 
## 伪类和伪元素

## 长文本如何用CSS实现省略号
```css
p {
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
}
p:hover {
  overflow: visible;
}
```
解释：

`white-space`规定了如何处理元素中的空格。

| value | description |
|-------|-------------|
| normal | 默认方式：多余的空格被忽略，必要时换行 |
| nowrap | 多余的空格被忽略，遇见`<br>`换行 |
| pre | 保留空格，只在换行符换行，类似于`<pre>` |
| pre-line | 多余的空格被忽略，保留换行符，必要时换行 |
| pre-wrap | 保留空格和换行符，必要时换行 |

`text-overflow`规定了溢出不显示的内容以何种形式呈现给用户。`css3`新属性。

| value | description |
|-------|-------------|
| clip | 默认方式：截断 |
| ellipsis | 多余部分用省略号 |

-----------------------------

## BEM
BEM：**Block**，**Element**，**Modifier**。类似于OOP，BEM是一种用代码和一系列模式来描述现实情况的方法，只考虑程序实体，和程序语言无关。利用BEM原则可以指导网站的构建。遵循统一的原则和规范可以有助于团队协作和沟通。

![](/image/BEM.jpg)

### Block
一个块是一个独立的实体，如积木一般。块可以复合。下图是搜索块：

![](/image/block.jpg)

### Element
元素是块的一部分，具有一定的功能，依赖上下文。下图是搜索块中的输入域和按钮：

![](/image/element.jpg)

块和元素构成了页面内容，并且遵循着一定的顺序。为了描述页面布局，需要为每一个块和元素定义可识别的唯一关键字（一般是名字）。
### 块的独立性
随着项目发展，我们会在页面中添加、移动或者删除某些块，因此每个块必须是独立的，可以放置在页面的任何位置，包括嵌套在其他块中。
### CSS和BEM
从CSS的角度来看：
- 每一个块（或者元素）必须有一个唯一的名字（即CSS类名）。
- HTML元素不能作为CSS选择器，因为这样上下文相关，不独立。
- 尽量少用级联（cascade）选择器。

CSS命名方案（BEM原则）：
1. 块的CSS类名就是块的名字。
2. 一个元素的CSS类名是一个块名和元素名的组合，中间用两个下划线（B__E）。
3. 块名和元素名用连字符分割单词（block-name__element-name）。
### 块修饰符
我们经常需要创建一个和已存在的块相似的块，只是外观和行为有些许改变。为了避免重复开发，引入修饰符（***modifier***）的概念。修饰符作为块/元素的属性，代表块/元素在外观/行为上的改变。一个修饰符有一个名字和一个值。可以同时使用多个修饰符。

![](/image/modifier.jpg)
### 从HTML/CSS角度看修饰符
对于块/元素来说，修饰符就是附加的CSS类。我们用一个下划线来分隔块名（或元素名）和修饰符名，再用另一个下划线来分隔修饰符名和它对应的值。
```html
<ul class="menu menu_size_big menu_type_buttons">
  ...
</ul>
```
```css
.menu_size_big {  }
.menu_type_buttons .menu__item {  }
```
### 块的一致性
复用块不仅要实现相同的CSS，还要复用行为。

----------

## 🤜 BFC
### 👉 从margin合并开始说起
MDN官网上描述margin合并：
> 块的顶部外边距和底部外边距有时被组合(折叠)为单个外边距，其大小是组合到其中的最大外边距，这种行为称为外边距塌陷(margin collapsing)，有的地方翻译为外边距合并。

发生外边距合并的三种情况：
1. 相邻的兄弟姐妹元素

毗邻的两个兄弟元素之间的外边距会塌陷（除非后者兄弟姐妹需要清除过去的浮动）。例如：
```html
<p style="margin-bottom: 30px;">这个段落的下外边距被合并...</p>
<p style="margin-top: 20px;">...这个段落的上外边距被合并。</p>
```
可以发现这两个段落中间的距离，不是 ”上面段落的下边距“ 与 ”下面段落的上边距“ 的 求和 ，而是两者中的较大者（在此示例中为30px）。

2. 块级父元素与其第一个/最后一个子元素

如果块级父元素中，不存在上边框、上内边距、内联元素、 清除浮动 这四条属性（也可以说，当上边框宽度及上内边距距离为0时），那么这个块级元素和其第一个子元素的上边距就可以说”挨到了一起“。此时这个块级父元素和其第一个子元素就会发生上外边距合并现象，换句话说，此时这个父元素对外展现出来的外边距将直接变成这个父元素和其第一个子元素的margin-top的较大者。
类似的，若块级父元素的 margin-bottom 与它的最后一个子元素的margin-bottom 之间没有父元素的 border、padding、inline content、height、min-height、 max-height 分隔时，就会发生下外边距合并现象。

3. 空块元素

如果存在一个空的块级元素，其 border、padding、inline content、height、min-height 都不存在。那么此时它的上下边距中间将没有任何阻隔，此时它的上下外边距将会合并。例如：
```html
<p style="margin-bottom: 0px;">这个段落的和下面段落的距离将为20px</p>

<div style="margin-top: 20px; margin-bottom: 20px;"></div>

<p style="margin-top: 0px;">这个段落的和上面段落的距离将为20px</p>
```
### 👉 发生margin合并的原因
#### margin 合并的条件
1. 必须是处于常规文档流（非float和绝对定位）的块级盒子,并且处于同一个 BFC 当中。
2. 没有inline盒子，没有 padding 和 border 将他们分隔开。
3. 都属于垂直方向上相邻的外边距。
#### margin 合并的初衷
Collapsing margins 的初衷就是为了让段落显示的更加好看。以由几个段落组成的典型文本页面为例。第一个段落上面的空间等于段落的上外边距。如果没有外边距合并，后续所有段落之间的外边距都将是相邻上外边距和下外边距的和。这意味着段落之间的空间是页面顶部的两倍。如果发生外边距合并，段落之间的上外边距和下外边距就合并在一起，这样各处的距离就一致了。

![](/image/外边距合并初衷.png)

### 👉 如何解决margin合并
只要不满足合并条件即可。
#### 兄弟元素的margin合并
让两个元素不处于同一个BFC，比如float、inline-block、绝对定位会创建一个BFC。[测试效果](https://jsfiddle.net/22oacowm/3/)
#### 父子元素的margin合并
1. 使父元素形成一个BFC，比如overflow。
2. 父元素增加border或者padding。

[测试效果](https://jsfiddle.net/22oacowm/5/)
### 👉 什么是 BFC
在普通的文档流中，盒子会参与一种格式上下文，这个盒子可能是块盒也可能是行内盒，但不可能同时是块盒又是行内盒。块级盒参与块级格式上下文(***Block formatting contexts***)，行内级盒参与行级格式上下文(***Inline formatting contexts***)。
#### BFC 的形成
浮动，绝对定位元素，display属性为inline-boxs、table-cells、table-captions的不是块盒的块容器(除非这个值已经被传播到视口)，以及当overflow不为visible的块盒才会为它的内容创建新的BFC。总结来说就是：
1. float 的值不为 none
2. position 的值不为 static 或 relative(absolute 和 fixed)
3. display 的值为 table-cell、table-caption、inline-block、flex 或 inline-flex
4. overflow 的值不为 visible(hidden 和 auto)
#### BFC 的特性
1. 在 BFC 中，盒子都是从它的包含块的顶部一个一个的垂直放置的。两个相邻盒子的垂直间距决定于 margin 属性。在 BFC 中，两个相邻块级盒子之间垂直方向上的外边距是会折叠的。
2. 在 BFC 中，每个盒子的左外边界挨着包含块的左边界(对于从右到左的格式化，则为右边界互相挨着)。即使是存在浮动元素也是如此(即使一个盒子的行盒是因为浮动而收缩了的)，除非这个盒子建立了一个新的 BFC(在某些情况下这个盒子自身会因为浮动而变窄)。
#### 进一步理解
1. BFC 内部的盒子会在垂直方向上一个接一个的放置；
2. 垂直方向上的距离由 margin 决定，会发生折叠；
3. BFC 中的子元素不会超出它的包含块，而 position 为 absolute 的元素(因为建立了一个新的 BFC)可以超出它的包含块边界；
4. BFC 区域不会与 BFC 区域外的 float 元素重叠；
5. 计算 BFC 的高度时，浮动子元素也参与计算(包含一个浮动元素的盒子如果不是 BFC 的话，将无法被撑开，高度为0)；
6. BFC 就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素，反之亦然；
#### BFC 的应用
1. 防止 margin 折叠，前面已经提及，不再赘述。
2. 清除浮动。
清除浮动的三种方法：[测试效果](https://jsfiddle.net/eq5oomos/)
- 为浮动元素的父元素设置`overflow: auto`，形成BFC。
- 为浮动元素后面的元素设置`clear: both`，清除前面浮动元素带来的影响。
- 利用伪类 after 为浮动元素的父元素设置
```css
#container:after {
  clear: both;
  content: '';
  display: block;
  width: 0;
  height: 0;
  visibility: hidden;
}
```
3. 多栏布局。

**三栏布局(左右定宽，中间适应)** 
- 左右绝对定位，中间设置 margin。缺点：中间栏设置最小宽度无效；中间栏和左右栏的顶端可能需要调整对齐。[测试效果](https://jsfiddle.net/o2nhfhe5/)
- 左右浮动，中间设置 margin。缺点：中间栏设置最小宽度无效；中间栏必须放在左右栏后面，这样的话主要内容会比左右两栏后加载；缩小浏览器时，中间栏会覆盖右栏。[测试效果](https://jsfiddle.net/o2nhfhe5/1/)
- 左右浮动，中间 BFC。缺点：中间栏设置最小宽度无效；中间栏必须放在左右栏后面，这样的话主要内容会比左右两栏后加载。[测试效果](https://jsfiddle.net/o2nhfhe5/7/)
- float + 负 margin。缺点：由于三栏都是浮动的，所以中间栏的文字会被右栏覆盖。（ps：调试了一会儿没有找到合适的办法避免这个问题，希望大佬们可以在issue中提出更好的办法。）[测试效果](https://jsfiddle.net/o2nhfhe5/4/)
- flex 布局，设置容器`display: flex`，中间栏`width: 100%`即可。缺点：缩小浏览器时，会挤压左右两栏。[测试效果](https://jsfiddle.net/o2nhfhe5/5/)
- grid 布局，设置容器`display: grid; grid-template-columns: 固定宽度 auto 固定宽度;`即可。优点：非常简单。缺点：大多浏览器暂不支持。[测试效果](https://jsfiddle.net/o2nhfhe5/9/)

**两栏布局(左固定右适应)**

在实现两栏布局的七种办法中，有一种是通过 BFC 实现的，其原理就是利用上面`进一步理解`提到的第四点：`BFC 区域不会与 BFC 区域外的 float 元素重叠`实现的。关于实现左固定右适应布局的七种办法可以参考这篇文章：[七种实现左侧固定，右侧自适应两栏布局的方法](https://zhuqingguang.github.io/2017/08/16/adapting-two-layout/#双inline-block方案)。

1. 双 inline-block 方案
2. 双 float 方案
3. float + margin-left 方案
4. 使用 absolute + margin-left 方法
5. 使用 float + BFC 方法
6. flex 方案
7. grid 方案

ps：本来想自己好好总结，无意发现这篇文章，我觉得总结得很棒了，所以不再赘述。

> 参考资料：
> 
> [外边距合并](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing)
> 
> [你不知道的 CSS 秘密：margin篇](http://www.jianshu.com/p/851b7483f700)
> 
> [我对BFC的理解](http://www.jianshu.com/p/76484dff1cb5)
> 
> [七种实现左侧固定，右侧自适应两栏布局的方法](https://zhuqingguang.github.io/2017/08/16/adapting-two-layout/#双inline-block方案)

-------------

## Flex布局

## Grid布局
## 元素居中
### 水平居中
inline 元素：`text-align: center`

block 元素：`margin: 0 auto`

多个block元素并排：
```html
<main class="inline-block-center">
  <div>
    I'm an element that is block-like with my siblings and we're centered in a row.
  </div>
  <div>
    I'm an element that is block-like with my siblings and we're centered in a row.
  </div>
  <div>
    I'm an element that is block-like with my siblings and we're centered in a row.
  </div>
</main>
```
```css
.inline-block-center {
  text-align: center;
}
.inline-block-center div {
  display: inline-block;
}
```
[测试效果](https://jsfiddle.net/w8kte3k9/)

用flexbox：
```css
.flex-center {
  display: flex;
  justify-content: center;
}
```
[测试效果](https://jsfiddle.net/w8kte3k9/2/)
### 垂直居中
#### inline 元素
- 一行元素：`padding-top` = `padding-bottom` or `line-height` = `height`
- 多行元素：
```html
<div class="center-table">
    <p>I'm vertically centered multiple lines of text in a CSS-created table layout.</p>
</div>
```
```css
/* 伪装成table的cell，设置vertical-align */
.center-table {
  display: table;
}
.center-table p {
  display: table-cell;
  vertical-align: middle;
}
```
[测试效果](https://jsfiddle.net/w8kte3k9/3/)

用flexbox：
```css
.flex-center-vertically {
  display: flex;
  justify-content: center;
  flex-direction: column;
}
```
[测试效果](https://jsfiddle.net/w8kte3k9/4/)
#### block 元素
知道需要居中元素的height：
```css
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  height: 50px;
  margin-top: -25px; /* height/2, account for padding and border if not using box-sizing: border-box */
}
```
[测试效果](https://jsfiddle.net/1vc637z4/1/)

不知道需要居中元素的height：
```css
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}
```
[测试效果](https://jsfiddle.net/1vc637z4/2/)

用flexbox：
```css
.parent {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
```
[测试效果](https://jsfiddle.net/1vc637z4/4/)
### 水平和垂直居中
固定宽度和高度的元素：
```css
.parent {
  position: relative;
}
.child {
  width: 80px;
  height: 40px;
  padding: 10px;
  position: absolute;
  top: 50%;
  left: 50%;
  margin: -30px 0 0 -50px; /* 原理和之前的相似，一半的宽/高 + padding */
}
```
[测试效果](https://jsfiddle.net/1vc637z4/6/)

未知宽度和高度：
```css
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```
[测试效果](https://jsfiddle.net/1vc637z4/8/)

用flexbox：
```css
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}
```
[测试效果](https://jsfiddle.net/1vc637z4/10/)

----------

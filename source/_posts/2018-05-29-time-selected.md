---
title: 基于iScroll的移动端日期选择器
date: 2018-05-29 15:40:40
tags: [js, css, html]
---

最近博主在做一个项目的时候，需要一个移动端的日期选择器，在网上没有找到自己想要的插件，只能自己去撸了。

其实撸起来的时候发现也不是很麻烦，只是有几点需要注意一下。

废话不多说，直接上代码。

等等等等，上代码之前差点忘告诉大家，先去下个[iScroll.js](http://wiki.jikexueyuan.com/project/iscroll-5/gettingstart.html)

## HTML代码

```bash
<div class="wrapper_box">
	<p class="line"></p>
	<div id="year_wrapper">
		<ul>
			<li>&nbsp;</li>
			<li>2017年</li>
			<li>2017年</li>
			<li>2017年</li>
			<li>2017年</li>
			<li>&nbsp;</li>
		</ul>
	</div>
	<div id="month_wrapper">
		<ul>
			<li>&nbsp;</li>
			<li>1月</li>
			<li>1月</li>
			<li>1月</li>
			<li>1月</li>
			<li>&nbsp;</li>
		</ul>
	</div>
	<div id="day_wrapper">
		<ul>
			<li>&nbsp;</li>
			<li>1日</li>
			<li>1日</li>
			<li>1日</li>
			<li>1日</li>
			<li>1日</li>
			<li>&nbsp;</li>
		</ul>
	</div>
</div>
```

## CSS代码

```bash
.line{
	height: 40px;
	width: 100%;
	border-top: 1px solid #999999;
	border-bottom: 1px solid #999999;
	position: absolute;
	top: 40px;
}
.wrapper_box:before{
	content: "";
    width: 100%;
    height: 40px;
    background: -webkit-linear-gradient(top, rgba(255,255,255,1), rgba(255,255,255,0));
    position: absolute;
    left: 0;
    top: 0;
    z-index: 10;
}
.wrapper_box:after{
	content: "";
    width: 100%;
    height: 40px;
    background: -webkit-linear-gradient(top, rgba(255,255,255,0), rgba(255,255,255,1));
    position: absolute;
    left: 0;
    bottom: 0;
    z-index: 10;
}
.wrapper_box{
	display: flex;
	justify-content: space-between;
	position: relative;
	margin-top: 30%;
}
.wrapper_box>div{
	overflow: hidden;
	height: 120px;
	flex: auto;
	text-align: center;
}
.wrapper_box>div>ul>li{
	line-height: 40px;
}
```

## JS代码

```bash
let yearScroll = new IScroll('#year_wrapper')
let monthScroll = new IScroll('#month_wrapper')
let dayScroll = new IScroll('#day_wrapper')
```

看完上面代码以后，大家可能有个疑问，为啥ul列表有两个空值，这是因为我们的选择区域距离上下有40px的距离，如果不增加两个空值的话，列表的第一个值和第二个值就滚不到选择区域了。

上面代码撸完以后，浏览器运行，就可以看到如下截图了

{% asset_img 1.png %}

可以顺畅的滚动了，难道这就结束了？no，骚年你还是太年轻了

其实滚动的时候我们可以发现几个问题

> NO.1 滚动的时候没有对齐，上图....图..........图

{% asset_img 2.png %}

WTF！！！！这样怎么办，难道要通过一系列的计算来让它对齐？no..no..no,骚年看好了，出来吧，神龙

```bash
let yearScroll = new IScroll('#year_wrapper',{
	snap: 'li'
})
```

加上这个以后，我们发现问题解决了。

这个其实就是iScroll.js的功能，snap属性可以让我们对齐元素。代码中加上这个，我们就可以对齐到元素li了。

第一个问题完美解决。

> NO.2 半透明遮罩的地方不能滚动

为了让选择器看起来更好看，我们为选择器增加了两个半透明遮罩，就是wrapper_box的伪元素before和after。

就导致了只有我们的手指在选择区域触摸滚动才能滚动日期。显然这样用户体验是极差的。

经过多方查找，我终于找到了解决方案。

```bash
pointer-events: none
```
把上面的css加到before和after里面，你就会发现问题解决了。oh，yes，give me five。

至于这个属性的作用，嘿嘿嘿，自己百度吧！！！！


到此，一个移动端的日期选择器就完成了。

博主的分享就到这了，喜欢的话欢迎来star。

{% asset_img 3.png %}



title: css
author: tzq
tags:
  - Css
categories:
  - Css
  - ''
date: 2020-03-27 17:26:00
---
### CSS学习




<!--more-->

#### 四种引入css的方法

1. 内联样式：

```
<div class="banner" style="background-image:url(#)"></div>

```
写在标签内的为内联样式。

2. 外联样式

```
<head>
		<tittle>汤子晴的个人简历</tittle>
		<style>
		iframe{
			width: 100%;
			height: 400px;
		}			
		table{
			border-collapse: collapse;
		}
		body{
			background-color: grey;
		}
		h1{
			text-align:center;color:red
		}
		</style>
	</head>

```

3. 外部css文件：即将css样式写在新的css文件内，有html文件引入使用。

```
<head>
		<link rel="stylesheet" href="./a.css">
</head>

```

4. css文件内可引入其他css文件

```
@import url("./b.css");

```

#### CSS属性

1. float:浮动    浮动须用边框限制

```
.clearfix::after{
	content: '';
	display: block;
	clear: both;
}


```

2.list-style:

3.选择器

①  `>`  右边必须是左边的儿子，不能是孙子

```
.topNavBar>nav>ul{
	list-style: none;margin: 0;padding: 0;border: red 1px solid;"
}

```

4.页面默认字体大小是16px

5.iconfoot使用symbol方法引入图标

①在head中 插入iconfont生成的Js

		<script src="//at.alicdn.com/t/font_1754054_nj983jy1fp.js"></script>
		<style type="text/css">
			.icon{
				width: 1em;
				height: 1em;
				vertical-align: -0.15em;
				fill: currentcolor;
				overflow: hidden;
			}
		</style>


②href是iconfont中图标的名字

	<svg class="icon" aria-hidden="true">
			<use xlink:href="#icon-weibo"></use>
	</svg>

6、做三角形

	.userCard .welcome .triangle{
	display: block;
	border:10px solid transparent;
	width:0px;
	border-left-color:#e6686a;
	border-top-width:0px;
	position: absolute;
	left: 5px;
	top: 100%;
	}

7、**div高度由其内部文档流元素的高度总和决定**

文档流：文档内元素的流动方向

内联元素:从左往右流  长单词不换行 强行使单词换行 word-break:break-all;

块级元素:从上往下流  使块级元素连在一起可使用float

8、脱离文档流

	position：fixed；
    
9、相对定位

在父元素中写

	position：relative；

在子元素中写

	position；absolute；
    
10、**伪元素 ** 不是空的标签都有伪类 无法被触发

	div::before{
    content:'【';
    }
   	div::after{
    	content:'】';
    	}

11、旋转动画

	@keyframes slidein{
		from{
			transform: rotate(0deg);
		}
		to{
			transform: rotate(360deg);
		}
	}
    

	#yinyang{
		width: 200px;
		height: 200px;
		border-radius: 50%;
		background: linear-gradient(to bottom, #ffffff 0%,#ffffff 50%,#000000 50%,#000000 100%); 
		position: relative;
		margin: auto;
		animation-name: slidein;
		animation-duration: 3s;
        animation-iteration-count: infinite; /不停变化/
        animation-timing-function: linear; /线性变化/
	}
 
 
12、hover触发

	body > main .downloadResume:hover{
	box-shadow: 0px 5px 10px 0px rgba(0,0,0,0.5);//阴影	
	}
    
13、阴影出现速度

	transition: box-shadow 0.3s;
    
14、字体加粗

	font-weight:bold;
    
15、加圆角

	border-radius: 4px;
    
16、盒模型 将10像素算在50%里面

	width:50%;
    box-sizing:border-size;
    padding-right:10px;
    
17、选中第n个儿子（伪类）

	.skills >li >li:nth-child(even){ //even偶数，选中第偶数个儿子 odd奇数
    }
    
18、将块级元素变成内联元素需要写vertical-align,否则会出bug

	display: inline-block;
	vertical-align: top;
    
19、改变鼠标悬上去的手势

	cursor: pointer;
    
20、改变动画变化速度

	transition: all 0.3s;
 

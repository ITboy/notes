# Knowledge points about HTML

## 怎样组织一个HTML页面
多数情况下HTML的页面长这样：
![HTML layout](https://mdn.mozillademos.org/files/12417/sample-website.png)

``` HTML
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title>A Common Page Layout</title>
		<link rel="stylesheet" type="text/css" href="style.css">
	</head>
	<body>
		<header>Article heading</header>
		<nav>
	      <ul>
	        <li><a href="#">Home</a></li>
	        <li><a href="#">Our team</a></li>
	        <li><a href="#">Projects</a></li>
	        <li><a href="#">Contact</a></li>
	      </ul>
	
	       <!-- A Search form is another commmon non-linear way to navigate through a website. -->
	
	       <form>
	         <input type="search" name="q" placeholder="Search query">
	         <input type="submit" value="Go!">
	       </form>
	    </nav>
		<main>
			The main content.
			<aside>Related links</aside>
		</main>
		<footer>Page footer</footer>
	<body>
</html>
```

### 页面的主要组成部分说明：

* `<main>`包含页面的主要内容，一个页面应该只有一个`<main>`元素，并且是`<body>`的直接子元素
* `<header>`包含一些介绍性的内容，当作为页面的头的时候，介绍整个页面，也可以出现在`<section>`内部
* `<aside>`包含的内容与主要内容无关，但是提供一些额外的内容，比如一些相关的连接，作者的信息等，可以放在`<main`中
* `<footer>`包含一些结束性的内容，作为页面的结尾时，通常是地址信息，版权信息等
* `<nav>` 页面的导航，可以作为`<header>`的一部分，但不是必须的
* `<article>`包含一块独立的内容，即便没有页面的其他内容，他也有意义，可以被重用的单元
* `<section>`与`<article>`类似，但是他还可以把组织一些有单独功能的部分组织在一起，并且他一般有`<header>`,可以有`<footer>`

### 规划一个网站

怎么为一个网站做规划呢？

1. 思考一个页面的通用部分放些什么内容，比如footer的地址，版权，头部的logo和介绍信息，aside是否放一些超链接或作者信息
2. 粗略规划页面的布局，分几列，1中规划的信息分别如何放置
3. 头脑风暴整个网站有什么内容或功能
4. 对3中的内容进行分类
5. 对分类好的内容进行分配，要分几个页面，他们之间的跳转关系

### 使一个页面更有意义

#### 有意义的标签

#### 没有意义的标签

* div
* span

## 图片

### 普通用法
使用图片时注意以下几点：

* 一定设置图片的width和height属性
  这两个属性可以引起图片的拉伸和压缩，但是不要这么做，要设置为真正图片的大小，因为压缩会导致带宽浪费，拉伸会影响图片效果，如果一定要这么做，在css中做。这里保留这两个属性是在图片加载缓慢时，为页面预留一定的大小，提升页面的性能和体验。
* 对于有意义的图片要使用img标签，如果是装饰使用，使用css background-image
* 为图片加caption使用figure
```HTML
  <figure>
    <img src="img.jpg">
    <figcaption>图2-1</figcaption>
  </figure>
```

### Responsive图片

#### resolution switching problem

当不同大小的屏幕需要加载图片时，根据屏幕大小自动下载适合的图片，节约带宽

``` HTML
<img srcset="elva-fairy-320w.jpg 320w,
             elva-fairy-480w.jpg 480w,
             elva-fairy-800w.jpg 800w"
     sizes="(max-width: 320px) 280px,
            (max-width: 480px) 440px,
            800px"
     src="elva-fairy-800w.jpg" alt="Elva dressed as a fairy">
```

#### art direction problem

对于不同大小的屏幕可能采用不同的布局，使用不同的图片。

```HTML
<picture>
  <source media="(max-width: 799px)" srcset="elva-480w-close-portrait.jpg">
  <source media="(min-width: 800px)" srcset="elva-800w.jpg">
  <img src="elva-800w.jpg" alt="Chris standing up holding his daughter Elva">
</picture>
```

#### difference

这两种都是根据不同的屏幕大小，使用不同的图片，那么有什么不同？
在使用时，发现如果使用`resolution switching problem`来解决`art direction problem`的问题，将屏幕逐渐调大时，会下载另一个图片并正常显示，但是如果再缩小时，不会换成另一种布局的图片，所以使用`img`属性，在使用大的图片时，他会下载并显示大的图片，但是如果是缩小，他就认为无需下载小图片，而是把大图片缩小以节约带宽。

## Table
### caption

为表格添加一个标题，这是表里的第一项，对于screenreader的用户，来通过caption来决定是否要看表中的具体内容

### colgroup

在为表添加样式的时候，可以为table加样式，也可以为行tr加样式，还可以为单元格加样式，那么怎么为一列加样式呢？这就是colgroup的作用。
可以为colgroup内部定义多个col，为每一个col可以添加样式。这个应该放在table的仅次于caption的位置

### th

这是在一个复杂的表中最重要的元素，他有如下几个属性：
* colspan
  当前单元格占据几列
* rowspan
  当前单元格占据几行
* scope
  表示当前head的作用，是一行，还是一列，还是一个行group，或列group

### thead

  表头，一般是一些<th>组成的行

### tbody

  表的主体

### tfoot

  表的结尾行，比如统计信息之类

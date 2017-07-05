# CSS 学习笔记

## 术语

* CSS Declaration
  属性 + 冒号 + 值
  ![CSS Declaration](https://mdn.mozillademos.org/files/3665/css%20syntax%20-%20declaration.png)

* CSS Declaration block
  { + CSS Declarations + }
  ![CSS Declaration block](https://mdn.mozillademos.org/files/3667/css%20syntax%20-%20declarations%20block.png)

* CSS ruleset (or rule)
  selector + CSS Declaration block
  ![css rule](https://mdn.mozillademos.org/files/3668/css%20syntax%20-%20ruleset.png)

* CSS statments
  - CSS ruleset (or rule)
  - At-rules
  - Nested statements
  - Comments /\* This is a comment \*/

## Simple Selector
### Type selector

格式: `TAG { ... }`

``` css
/* All p elements are red */
p {
  color: red;
}

/* All div elements are blue */
div {
  color: blue;
}
```

### Class selector

格式: `.class-name { ... }`

``` css
.base-box {
  background-image: linear-gradient(to bottom, rgba(0,0,0,0.1), rgba(0,0,0,0.3));
  padding: 3px 3px 3px 7px;
}

.important {
  font-weight: bold;
}

.editor-note {
  background-color: #9999ff;
  border-left: 6px solid #333399;
}

.warning {
  background-color: #ff9999;
  border-left: 6px solid #993333;
}
```

### ID selector

格式： `#ID { ... }`

``` css
#polite {
  font-family: cursive;
}

#rude {
  font-family: monospace;
  text-transform: uppercase;
}
```

### Universal selector

格式：`* { ... }`

``` css
* {
  padding: 5px;
  border: 1px solid black;
  background: rgba(255,0,0,0.25)
}
```

### Combinators selector

* 空格
  子孙节点
* >
  儿子节点
* +
  下一个紧挨的兄弟节点
* ~
  所有兄弟节点

``` css
section p {
  color: blue;
}

section > p {
  background-color: yellow;
}

h2 + p {
  text-transform: uppercase;
}

h2 ~ p {
  border: 1px dashed black;
}
```

## Attribute selectors

### 是否存在或值比较的属性选择器

* [attr]
  选择节点存在attr这个属性，不论他的值是什么

* [attr=value]
  存在attr并且attr的值等于value的节点

* [attr~=value]
  存在attr，并且value在attr的值所指示的列表中

### 属性值子串选择器

以下选择器都是把value作为一个完整的字符串来匹配的选择器，不会考虑空格在值中的起把多个值分开的作用。

* [attr|=value]
  等于value或以`value-`开头，主要用于`lang`属性

* [attr^=value]
  以value开头的属性值

* [attr$=value]
  以value结尾的属性值

* [attr*=value]
  包含value的属性值

## 伪类和伪元素

### 伪类

伪类是以冒号开头的关键字，用来表示一个元素的当前状态，比如一个元素出于光标hover，或者checkbox是选中，或者一个元素的是他父元素的子元素中的第一个等等。

```
:active
:any
:checked
:default
:dir()
:disabled
:empty
:enabled
:first
:first-child
:first-of-type
:fullscreen
:focus
:hover
:indeterminate
:in-range
:invalid
:lang()
:last-child
:last-of-type
:left
:link
:not()
:nth-child()
:nth-last-child()
:nth-last-of-type()
:nth-of-type()
:only-child
:only-of-type
:optional
:out-of-range
:read-only
:read-write
:required
:right
:root
:scope
:target
:valid
:visited
```

#### Example

* 超链接的不同状态的样式

```HTML
<a href="https://developer.mozilla.org/" target="_blank">Mozilla Developer Network</a>
```

```CSS
/* These styles will style our link
   in all states */
a {
  color: blue;
  font-weight: bold;
}

/* We want visited links to be the same color
   as non visited links */
a:visited {
  color: blue;
}

/* We highlight the link when it is
   hovered (mouse), activated
   or focused (keyboard) */
a:hover,
a:active,
a:focus {
  color: darkred;
  text-decoration: none;
}
```

* 奇偶行不同样式
[完整例子](https://thimbleprojects.org/prepper/290470)

### 伪元素

伪元素与伪类类似但不同，他们是关键字以`::`开头，可以被添加到选择器的后面来选择元素的某些部分。

#### 在尾部添加内容

```HTML
<ul>
  <li><a href="https://developer.mozilla.org/en-US/docs/Glossary/CSS">CSS</a> defined in the MDN glossary.</li>
  <li><a href="https://developer.mozilla.org/en-US/docs/Glossary/HTML">HTML</a> defined in the MDN glossary.</li>
</ul>
```

```CSS
/* All elements with an attribute "href", which values
   start with "http", will be added an arrow after its
   content (to indicate it's an external link) */
[href^=http]::after {
  content: '⤴';
}
```

#### 为第一行和第一个单词添加样式

```HTML
<p>This is my very important paragraph.
 I am a distinguished gentleman of such renown that my paragraph
 needs to be styled in a manner befitting my majesty. Bow before
my splendour, dear students, and go forth and learn CSS!</p>
```

```CSS
p::first-line {
  font-weight: bold;
}

p::first-letter {
  font-size: 3em;
  border: 1px solid black;
  background: red;
  display: block;
  float: left;
  padding: 2px;
  margin-right: 4px;
}
```
## CSS 规则优先级

受三个因素影响：
1. Importance
2. Specificity
3. Source Order

### Importance

如果一个属性值后面有`!important`，那么这个属性值的优先级一定是最高的。

```CSS
.better {
  background-color: gray;
  border: none !important;
}
```

### Specificity

根据specificity的不同，有不同的优先级，使用千，百，十，个来作为加权的单位：
1. 千位得1分： 如果规则定义在`<style>`或元素上的`style`属性中
2. 百位得1分：ID选择器选择
3. 十位得1分：class，伪类，属性选择器
4. 个位得1分：元素或伪元素选择器

例子：
<table>
  <tr>
    <th>Selector</th>
    <th>Thousands</th>
    <th>Hundrends</th>
    <th>Tens</th>
    <th>Ones</th>
    <th>Total specificity</th>
  </tr>
  <tr>
    <td>h1</td>
    <td>0</td>
    <td>0</td>
    <td>0</td>
    <td>1</td>
    <td>0001</td>
  </tr>
  <tr>
    <td>#important</td>
    <td>0</td>
    <td>1</td>
    <td>0</td>
    <td>0</td>
    <td>0100</td>
  </tr>
  <tr>
    <td>h1 + p::first-letter</td>
    <td>0</td>
    <td>0</td>
    <td>0</td>
    <td>3</td>
    <td>0003</td>
  </tr>
  <tr>
    <td>li &gt; a[href=\*"en-US"] &gt; .inline-warning</td>
    <td>0</td>
    <td>0</td>
    <td>2</td>
    <td>2</td>
    <td>0022</td>
  </tr>
  <tr>
    <td>#important div &gt; div &gt; a:hover, inside a &lt;style&gt; element	</td>
    <td>1</td>
    <td>1</td>
    <td>1</td>
    <td>3</td>
    <td>1113</td>
  </tr>
</table>

## Box Model

![Default Box Model](https://mdn.mozillademos.org/files/13647/box-model-standard-small.png)

### Box Model的行为

* 如果没有设定box的width和height，box的大小由内部的内容决定
* 如果设置了box的width和height，box的大小就固定，如果box的内容溢出，默认内容就会显示并溢出到box之外。
可以设置overflow的属性（hidden，scroll，visiable）来修改该行为
* 如果要修改box的模型，可以设置box-sizing (center-box, boder-box)
* 默认背景图片和背景颜色会显示在content, padding, border中，可以设置background-clip (border-box, padding-box, content-box, text)

### display

* block
  元素以box模型显示，前面和后面都会强制换行。
* inline
  元素就像文本流，前后都不会换行，并且padding，border，margin会影响环绕的文本的位置，但不会影响外部的box。
  需要注意：margin只影响左右而不影响上下。
* inline-block
  像inline一样前后不会换行，但是padding，border，margin表现的像box。

## 常见属性

### text style

#### font

* font-family
  5 basic type: serif, sans-serif, monospace, cursive, fantasy
* font-size
* font-weight
* font-style
* color
  web safe color: 00, 33, 66, 99, cc, ff

#### text layout

* text-align
* text-transform
* text-decoration
* text-shadow
* letter-spacing
* word-spacing
* line-height
* text-indent
* overflow-wrap/word-wrap
* word-break

#### link style

5 pseudo-classes
1. link
2. visited
3. focus
4. hover
5. active


## 经验总结
### 如果发现外部的box的位置不是预期的，而他以及他上下元素的margin，border等都考虑进去了，那么很有可能是内部元素的marign溢出

### 为inline的元素设置margin，padding不会把外面的元素撑开

### overflow-wrap, word-wrap, word-break区别

word-wrap是overflow-wrap的历史名字，作用是：
当一个单词的长度过长时，默认浏览器会把这个单词放到下一行，但是如果他长到一整行都放不下，就会超出他的父元素，如果希望他在溢出时断行放到下一行，就可以使用overflow-wrap: break-word; 需要注意的是，配置了这个属性，当一个单词过长时仍然优先把他放到下一行，如果他的长度一行也放不下，就会把超出的部分放到下一行。这样的缺点是第一行会留下过多空白。

而word-break则更加简单粗暴，如果一个单词过长，直接断行，不需要考虑把整个单词放到下一行，因此多数情况下尽量不要使用。
优点是每一行不会浪费太多空间，缺点是单词会被随意打断，可读性差。

### 奇怪的float

#### 奇怪的float并不奇怪

float元素的行为：
1. float会使当前元素变为块元素，并且从normal流中出来，float元素从normal流出来后由前一个元素的位置决定
2. 如果前一个元素不是float，那么当前float元素就另起一个block，如果前一个是float，那么就根据前一个的位置来决定，如果不想受影响，虽然自己是float元素，也可以设置clear来不受前一个元素影响
3. float会使后面的元素的位置发生变化，因为前一个float元素从normal流中出来，所以后面元素的位置会根据前一个非float元素来定位
4. 如果只有前面两个行为，最终的结果会是后面的元素被重新定位，前一个float元素会出现在后面元素的上方，覆盖元素的内容，但实际上还会带来一个新的结果，就是后面的元素的内容会被挤压，挤压的方向有前一个float元素的float的值决定
5. 后面的元素可以通过设置clear的方向让自己不受前一个float元素的影响

#### 并不奇怪的float有点奇怪

float会见缝插针，哪怕有一点空隙，也会插进去。
以下浮动元素的高度不一样的问题：

浮动虽然看起来很简单，但是稍微不注意就会用错，而且如果不知道原理的话很难找到原因，例如：五个li元素浮动，我们要的效果应该是这样

![zhihu](https://pic4.zhimg.com/v2-0eea2c5944f3a8737ab095d2814aaadb_b.png)

可是现实问题确实这样：紫色的li调到下面去了。

![zhihu](https://pic4.zhimg.com/v2-b6c87bca46ee7621383ecf363f3d233b_b.png)

也许大部分人就像我一样只记得浮动会让父元素塌陷，无法撑开高度这个特性，可是我们却忘了还有一个重要的特性：

![zhihu](https://pic4.zhimg.com/v2-b6c87bca46ee7621383ecf363f3d233b_b.png)

因此，蓝色的li触碰到了蛋白质那个li，导致它被卡在那里，紫色自然被移到下一行

![zhihu](https://pic4.zhimg.com/v2-0d61f1926928fa232e7204bc914863cf_b.png)

解决办法，固定高度

![zhihu](https://pic3.zhimg.com/v2-0a2823acb61f9a21f984f7a5d1fff266_b.png)

## 良好习惯
### 为html节点定义`font-size: 10px`
  在文档中使用em和rem作为单位来定义长度，极个别属性使用px，比如text-shadow, border等
  如果需要设置长度的属性希望跟text的字体大小有关系，使用em，这样如果字体大小变大，不会影响他们的比例。
  如果需要设置的长度跟字体大小无关，是跟px相关，可以使用rem，如果rem计算除不尽，或很小数字的像素，就使用px

### 如果一个布局是responsive的，而他里面有图片，这时候由于图片的宽度默认是固定的，就是图片本身的宽度，这时如果布局被缩小的时候，图片就可能超出布局边界，解决办法是为图片配置: `max-width: 100%`，不要是用`width: 100%`，否则当布局太宽时，图片会被拉伸

## 浏览器的奇怪行为

### chrome

#### 最小字体是12px

当你设置字体大小小于12px，在computed style中会看到仍然显示12px。
可以在设置高级设置里设置最小字体大小。

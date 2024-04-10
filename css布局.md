css布局

### **行内元素和块级元素的区别：**

行内元素：

- 与其他行内元素并排；
- 不能设置宽、高。默认的宽度，就是文字的宽度。
- 可以通过` text-align: center; `使得居中显示

块级元素：

- 霸占一行，不能与其他任何元素并列；
- 能接受宽、高。如果不设置宽度，那么宽度将默认变为父亲的100%。
- 可以通过` margin: auto; `使得居中显示



**行内元素和块级元素的分类：**

在以前的HTML知识中，我们已经将标签分过类，当时分为了：文本级、容器级。

从HTML的角度来讲，标签分为：

- 文本级标签：p、span、a、b、i、u、em。
- 容器级标签：div、h系列、li、dt、dd。
-  行内块级元素：<button>  <input>    <textarea>  <select>  <img>

可以通过修改属性改变
```css
display: inline;//行内元素
display: block;//
display:inline-block;//既可以设置宽高，也可以并排
```





### position

**Position** 是 CSS 一个很重要的属性，设定值包括 static、absolute、relative、fixed 和 sticky 五个。

static：受到html的flow移动，对top left right bottom都不生效

relative：在static的基础上增加两个特性，第一是受top left right bottom影响，第二是可以让absolute子元素根据他的位置去定位

absolute：绝对位置，根据relative、absolute的父容器定位或整个页面的位置定位。

fixed：牛皮癣广告

sticky：设置top，卷轴变化到对应的top高度时，sticky跟着页面走





### Flex 布局

https://www.bilibili.com/video/BV1qJ411N7TA/?spm_id_from=333.999.0.0&vd_source=6f7d567a14610ad97533d7595995ec91

任何一个容器都可以指定为 Flex 布局。

> ```css
> .box{
>   display: flex;
> }
> ```

行内元素也可以使用 Flex 布局。

> ```css
> .box{
>   display: inline-flex;
> }
> ```

采用 Flex 布局的元素，称为 **Flex 容器（flex container）**，简称"容器"。它的所有子元素自动成为容器成员，称为 **Flex 项目（flex item**），简称"项目"。

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2015071004.png)

容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做`main start`，结束位置叫做`main end`；交叉轴的开始位置叫做`cross start`，结束位置叫做`cross end`。

项目默认沿主轴排列。单个项目占据的主轴空间叫做`main size`，占据的交叉轴空间叫做`cross size`。

#### 容器的属性

以下6个属性设置在容器上。

> - flex-direction
> - flex-wrap
> - flex-flow
> - justify-content
> - align-items
> - align-content

**flex-direction属性**

`flex-direction`属性决定主轴的方向（即项目的排列方向）。

> ```css
> .box {
>   flex-direction: row | row-reverse | column | column-reverse;
> }
> ```

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2015071005.png)

它可能有4个值。

> - `row`（默认值）：主轴为水平方向，起点在左端。
> - `row-reverse`：主轴为水平方向，起点在右端。
> - `column`：主轴为垂直方向，起点在上沿。
> - `column-reverse`：主轴为垂直方向，起点在下沿。

**flex-wrap属性**

默认情况下，项目都排在一条线（又称"轴线"）上。`flex-wrap`属性定义，如果一条轴线排不下，如何换行。

![img](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015071006.png)

> ```css
> .box{
>   flex-wrap: nowrap | wrap | wrap-reverse;
> }
> ```

它可能取三个值。

（1）`nowrap`（默认）：不换行。

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2015071007.png)

（2）`wrap`：换行，第一行在上方。

![img](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015071008.jpg)

（3）`wrap-reverse`：换行，第一行在下方。

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2015071009.jpg)

**flex-flow**

`flex-flow`属性是`flex-direction`属性和`flex-wrap`属性的简写形式，默认值为`row nowrap`。

> ```css
> .box {
>   flex-flow: <flex-direction> || <flex-wrap>;
> }
> ```

**justify-content属性**

`justify-content`属性定义了项目在主轴上的对齐方式。

> ```css
> .box {
>   justify-content: flex-start | flex-end | center | space-between | space-around;
> }
> ```

![img](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015071010.png)

它可能取5个值，具体对齐方式与轴的方向有关。下面假设主轴为从左到右。

> - `flex-start`（默认值）：左对齐
> - `flex-end`：右对齐
> - `center`： 居中
> - `space-between`：两端对齐，项目之间的间隔都相等。
> - `space-around`：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

**align-items属性**

`align-items`属性定义项目在交叉轴上如何对齐。

> ```css
> .box {
>   align-items: flex-start | flex-end | center | baseline | stretch;
> }
> ```

![img](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015071011.png)

它可能取5个值。具体的对齐方式与交叉轴的方向有关，下面假设交叉轴从上到下。

> - `flex-start`：交叉轴的起点对齐。
> - `flex-end`：交叉轴的终点对齐。
> - `center`：交叉轴的中点对齐。
> - `baseline`: 项目的第一行文字的基线对齐。
> - `stretch`（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。

**align-content属性**

`align-content`属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。

> ```css
> .box {
>   align-content: flex-start | flex-end | center | space-between | space-around | stretch;
> }
> ```

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2015071012.png)

该属性可能取6个值。

> - `flex-start`：与交叉轴的起点对齐。
> - `flex-end`：与交叉轴的终点对齐。
> - `center`：与交叉轴的中点对齐。
> - `space-between`：与交叉轴两端对齐，轴线之间的间隔平均分布。
> - `space-around`：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
> - `stretch`（默认值）：轴线占满整个交叉轴。



#### 项目的属性

以下6个属性设置在项目上。

> - `order`
> - `flex-grow`
> - `flex-shrink`
> - `flex-basis`
> - `flex`
> - `align-self`

**order属性**

`order`属性定义项目的排列顺序。数值越小，排列越靠前，默认为0。

> ```css
> .item {
>   order: <integer>;
> }
> ```

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2015071013.png)

**flex-grow属性**

`flex-grow`属性定义项目的放大比例，默认为`0`，即如果存在剩余空间，也不放大。

> ```css
> .item {
>   flex-grow: <number>; /* default 0 */
> }
> ```

![img](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015071014.png)

如果所有项目的`flex-grow`属性都为1，则它们将等分剩余空间（如果有的话）。如果一个项目的`flex-grow`属性为2，其他项目都为1，则前者占据的剩余空间将比其他项多一倍。

**flex-shrink属性**

`flex-shrink`属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。

> ```css
> .item {
>   flex-shrink: <number>; /* default 1 */
> }
> ```

![img](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015071015.jpg)

如果所有项目的`flex-shrink`属性都为1，当空间不足时，都将等比例缩小。如果一个项目的`flex-shrink`属性为0，其他项目都为1，则空间不足时，前者不缩小。

负值对该属性无效。

**flex-basis属性**

`flex-basis`属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为`auto`，即项目的本来大小。

> ```css
> .item {
>   flex-basis: <length> | auto; /* default auto */
> }
> ```

它可以设为跟`width`或`height`属性一样的值（比如350px），则项目将占据固定空间。

**flex属性**

`flex`属性是`flex-grow`, `flex-shrink` 和 `flex-basis`的简写，默认值为`0 1 auto`。后两个属性可选。

> ```css
> .item {
>   flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
> }
> ```

该属性有两个快捷值：`auto` (`1 1 auto`) 和 none (`0 0 auto`)。

建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会推算相关值。

**align-self属性**

`align-self`属性允许单个项目有与其他项目不一样的对齐方式，可覆盖`align-items`属性。默认值为`auto`，表示继承父元素的`align-items`属性，如果没有父元素，则等同于`stretch`。

> ```css
> .item {
>   align-self: auto | flex-start | flex-end | center | baseline | stretch;
> }
> ```

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2015071016.png)

该属性可能取6个值，除了auto，其他都与align-items属性完全一致。



### Grid布局

https://www.bilibili.com/video/BV1XE41177oN/?spm_id_from=333.999.0.0

Flex 布局是轴线布局，只能指定"项目"针对轴线的位置，可以看作是**一维布局**。Grid 布局则是将容器划分成"行"和"列"，产生单元格，然后指定"项目所在"的单元格，可以看作是**二维布局**。Grid 布局远比 Flex 布局强大。

#### **容器和项目**

采用网格布局的区域，称为"容器"（container）。容器内部采用网格定位的子元素，称为"项目"（item）。

> ```markup
> <div>
>   <div><p>1</p></div>
>   <div><p>2</p></div>
>   <div><p>3</p></div>
> </div>
> ```

上面代码中，最外层的`<div>`元素就是容器，内层的三个`<div>`元素就是项目。

注意：项目只能是容器的顶层子元素，不包含项目的子元素，比如上面代码的`<p>`元素就不是项目。Grid 布局只对项目生效。

行和列

容器里面的水平区域称为"行"（row），垂直区域称为"列"（column）。

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/1_bg2019032502.png)

上图中，水平的深色区域就是"行"，垂直的深色区域就是"列"。

单元格

行和列的交叉区域，称为"单元格"（cell）。

正常情况下，`n`行和`m`列会产生`n x m`个单元格。比如，3行3列会产生9个单元格。

网格线

划分网格的线，称为"网格线"（grid line）。水平网格线划分出行，垂直网格线划分出列。

正常情况下，`n`行有`n + 1`根水平网格线，`m`列有`m + 1`根垂直网格线，比如三行就有四根水平网格线。

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/1_bg2019032503.png)

上图是一个 4 x 4 的网格，共有5根水平网格线和5根垂直网格线。



#### 容器属性

Grid 布局的属性分成两类。一类定义在容器上面，称为容器属性；另一类定义在项目上面，称为项目属性。这部分先介绍容器属性。

3.1 display 属性

`display: grid`指定一个容器采用网格布局。

> ```css
> div {
>   display: grid;
> }
> ```

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2019032504.png)

上图是`display: grid`的效果。

默认情况下，容器元素都是块级元素，但也可以设成行内元素。

> ```css
> div {
>   display: inline-grid;
> }
> ```

上面代码指定`div`是一个行内元素，该元素内部采用网格布局。

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2019032505.png)

上图是`display: inline-grid`的[效果](https://jsbin.com/qatitav/edit?html,css,output)。

> 注意，设为网格布局以后，容器子元素（项目）的`float`、`display: inline-block`、`display: table-cell`、`vertical-align`和`column-*`等设置都将失效。

3.2 grid-template-columns 属性， grid-template-rows 属性

容器指定了网格布局以后，接着就要划分行和列。`grid-template-columns`属性定义每一列的列宽，`grid-template-rows`属性定义每一行的行高。

> ```css
> .container {
>   display: grid;
>   grid-template-columns: 100px 100px 100px;
>   grid-template-rows: 100px 100px 100px;
> }
> ```

上面代码指定了一个三行三列的网格，列宽和行高都是`100px`。

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2019032506.png)

除了使用绝对单位，也可以使用百分比。

> ```css
> .container {
>   display: grid;
>   grid-template-columns: 33.33% 33.33% 33.33%;
>   grid-template-rows: 33.33% 33.33% 33.33%;
> }
> ```

**（1）repeat()**

有时候，重复写同样的值非常麻烦，尤其网格很多时。这时，可以使用`repeat()`函数，简化重复的值。上面的代码用`repeat()`改写如下。

> ```css
> .container {
>   display: grid;
>   grid-template-columns: repeat(3, 33.33%);
>   grid-template-rows: repeat(3, 33.33%);
> }
> ```

`repeat()`接受两个参数，第一个参数是重复的次数（上例是3），第二个参数是所要重复的值。

`repeat()`重复某种模式也是可以的。

> ```css
> grid-template-columns: repeat(2, 100px 20px 80px);
> ```

上面代码定义了6列，第一列和第四列的宽度为`100px`，第二列和第五列为`20px`，第三列和第六列为`80px`。

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2019032507.png)

**（2）auto-fill 关键字**

有时，单元格的大小是固定的，但是容器的大小不确定。如果希望每一行（或每一列）容纳尽可能多的单元格，这时可以使用`auto-fill`关键字表示自动填充。

> ```css
> .container {
>   display: grid;
>   grid-template-columns: repeat(auto-fill, 100px);
> }
> ```

上面代码表示每列宽度`100px`，然后自动填充，直到容器不能放置更多的列。

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2019032508.png)

除了`auto-fill`，还有一个关键字`auto-fit`，两者的行为基本是相同的。只有当容器足够宽，可以在一行容纳所有单元格，并且单元格宽度不固定的时候，才会有行为差异：`auto-fill`会用空格子填满剩余宽度，`auto-fit`则会尽量扩大单元格的宽度。

（3）fr 关键字

为了方便表示比例关系，网格布局提供了fr关键字（fraction 的缩写，意为"片段"）。如果两列的宽度分别为1fr和2fr，就表示后者是前者的两倍。

.container {
  display: grid;
  grid-template-columns: 1fr 1fr;
}
上面代码表示两个相同宽度的列。

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/1_bg2019032509.png)

fr可以与绝对长度的单位结合使用，这时会非常方便。

.container {
  display: grid;
  grid-template-columns: 150px 1fr 2fr;
}
上面代码表示，第一列的宽度为150像素，第二列的宽度是第三列的一半。

![img](https://cdn.jsdelivr.net/gh/kintong3000/Kintong-Image-Hosting@main/img/bg2019032510.png)

**（4）minmax()**

`minmax()`函数产生一个长度范围，表示长度就在这个范围之中。它接受两个参数，分别为最小值和最大值。

> ```css
> grid-template-columns: 1fr 1fr minmax(100px, 1fr);
> ```

上面代码中，`minmax(100px, 1fr)`表示列宽不小于`100px`，不大于`1fr`。

**（5）auto 关键字**

`auto`关键字表示由浏览器自己决定长度。

> ```css
> grid-template-columns: 100px auto 100px;
> ```

上面代码中，第二列的宽度，基本上等于该列单元格的最大宽度，除非单元格内容设置了`min-width`，且这个值大于最大宽度。

**（6）网格线的名称**

`grid-template-columns`属性和`grid-template-rows`属性里面，还可以使用方括号，指定每一根网格线的名字，方便以后的引用。

> ```css
> .container {
>   display: grid;
>   grid-template-columns: [c1] 100px [c2] 100px [c3] auto [c4];
>   grid-template-rows: [r1] 100px [r2] 100px [r3] auto [r4];
> }
> ```

上面代码指定网格布局为3行 x 3列，因此有4根垂直网格线和4根水平网格线。方括号里面依次是这八根线的名字。

网格布局允许同一根线有多个名字，比如`[fifth-line row-5]`。

**（7）布局实例**

`grid-template-columns`属性对于网页布局非常有用。两栏式布局只需要一行代码。

> ```css
> .wrapper {
>   display: grid;
>   grid-template-columns: 70% 30%;
> }
> ```

上面代码将左边栏设为70%，右边栏设为30%。

传统的十二网格布局，写起来也很容易。

> ```css
> grid-template-columns: repeat(12, 1fr);
> ```





### HTML元素居中方法总结 

通常我们将一个元素左右居中的方式是一件很容易的事：

当元素的`display`属性是`inline`或者`inline-block`，即使成为行内（块）元素的时候，用`text-align:center`就可以将它左右居中了。

如果元素的`display`属性是`block`的时候，将`margin-left`和`margin-right`设定为`auto`就可以将该元素左右居中了。

上下居中就没有那么容易了，本文将总结三种常用的上下左右居中的方法。

**方式1：**Position Absolute

``` img{
    position: absolute;
    top:50%;
    left:50%;
    transform: translate(-50%,-50%);
/* 等同于transform: translateX(-50%) translateY(-50%); */
}
```

**方式2：**flex布局

``` 
.background{
    display: flex;
    justify-content: center;/*水平轴*/
    align-items: center;/*垂直轴*/
}
```

**方式3：**Display Table

略





### 盒子模型

- 在 **标准盒子模型**中，**width 和 height 指的是内容区域**的宽度和高度。增加内边距、边框和外边距不会影响内容区域的尺寸，但是会增加元素框的总尺寸。
- 当两个元素的margin-top和margin-bottom发生重叠的时候，会保留较大的margin，不会叠加
- box-sizing：预设为content-box，即width为内容宽度。可以设为border-box，则最后的padding+content+border=width
- inline 属性并不会影响垂直方向的版面布局










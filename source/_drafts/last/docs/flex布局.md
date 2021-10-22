## flex布局概念  
> `flex`布局通常被叫做`flex box`，也叫`弹性盒子`或`容器`，是一种一维的`布局模型`，它能子元素(`flex item`)之间提供了强大的空间分布和对齐能力。   
   
![v2-54a0fc96ef4f455aefb8ee4bc133291b_720w.png](./img/flex-layout/WBQ8RSuH7norCs4.png)    

`flex容器`中存在两个轴，一个`水平方向`main axis，一个`竖直方向`cross axis，这是`flex`布局最重要的点，内部元素`flex item`都是根据这个轴来排列对齐的。  

## flex布局好处    
在一个`html元素`和内部`多个子元`的组合中。要实现子元素有序的，高度自定义的布局，在传统布局方式中，会使用`position`，`float`方式实现，比如多个子元素在父元素中`垂直水平居中`，比较困难。这个时候就可以用`flex`布局，只需要简单几句代码就可以优雅的实现。    

## flex容器属性    
> 设置一个盒子为`flex`容器，首先设置`display: flex`;  

 - **flex-direction**
 - **flex-wrap**
 - **justify-content**
 - **align-items**
 - **align-content**  
 
### flex-direction   
`flex-direction`属性规定所有`flex item`在`flex`容器从 **第一个元素** 开始的排列方式
  
```css  
flex-direction: row(默认) | row-reverse | column | column-reverse
```  
**row**： 从左往右水平展示    
**row-reverse**： 从右向左(反向)水平展示    
**column**： 从上往下垂直展示  
**column-reverse**： 从下往上(反向)垂直展示

### flex-wrap   
`flex-wrap`属性规定`flex`容器内所有`flex item`总宽度超出`flex`容器时换行方式    

```css  
flex-wrap: nowrap(默认) | wrap | wrap-reverse
```  
**nowrap**： 不换行，只展示一行。如果容器不够宽，平均缩小每个`item`的宽度  
**wrap**： 换行，向下换行  
**wrap-reverse**： 以相反方向换行，向上换行
   
### justify-content  
`justify-content`属性规定所有`flex item`作为一个整体在`flex`容器在主轴(默认为水平轴，通过`flex-direcation`设置)上的排列方式

```css  
justify-content: flex-start(默认值) | flex-end | center | space-between | space-around
```  
**flex-start**： 整体位于容器开头  
**flex-end**： 整体位于容器结尾  
**center**： 整体位于容器中心  
**space-between**： 每个`flex item`平均分布在容器内； `第一个item`和`容器开头`，`最后一个item`和`容器结尾`都无间距  
**space-around**： 每个`flex item`平均分布在容器内， `第一个item`和`容器开头`，`最后一个item`和`容器结尾`的间距都为`每个item`间距的一半    

### align-items 
`align-items`属性规定所有`flex item`作为一个整体在`flex`容器在副轴(非主轴，和`justify-content`相反)上的排列方式

```css  
align-items: stretch(默认值) | center | flex-start | flex-end | baseline
```  
**stretch**： 整体占满整个主轴的高度, 前提必须是你没有设置item的高度(宽度)，否则不生效  
**center**： 整体位于主轴中心   
**flex-start**： 整体位于主轴开头    
**flex-end**： 整体位于主轴结尾  
**baseline**： 主轴位于`item`中第一行文字 *基线* 对齐    

> 基线的概念：英文中大概就是一个小写字母 *x* 的最底部，*f* 的中下部；而中文大概就汉字中下部，不太准确，我没深入了解，有兴趣可以网上了解。
   
### align-content  
`align-content`属性规定了多条主轴的情况下(只有`item`换行了，才会产生多条主轴)，这些主轴作为一个整体在他们的排列方向中的排列方式；注意：只有一条主轴(`item`不换行)不生效

```css  
align-content: stretch(默认) | center | flex-start | flex-end | space-between | space-around
```  
**stretch**： 整体被拉伸以适应容器  
**center**： 整体内各个轴紧靠，并且位于容器中心，头尾留白   
**flex-start**： 整体内各个轴紧靠，并且位于容器头部    
**flex-end**： 整体内各个轴紧靠，并且位于容器尾部  
**space-between**： 整体平分，其中头尾和容器头尾紧靠，中间间隙平分  
**space-around**：  整体平分，其中头尾和容器头尾距离为中间item的空隙的一半，中间间隙平分 

## flex item属性  
 - order
 - flex-basis
 - flex-grow
 - flex-shrink
 - align-self  

### order 
`order `属性规定了当前`flex item`排列前后顺序，越小排列越靠前，可以为负数

```css  
order: 0(默认值) | <number>  
```      

### flex-basis  
`flex-basis `属性规定了当前`flex item`的高度(主轴为水平轴则是高度，如果主轴为竖直轴则是宽度，主轴通过`flex-direction`设置)的伸缩比例或大小

```css  
flex-basis: auto(默认) | number
```  
**number**： 可以是具体的像素值，如30px；也可以是auto大小百分比
  
### flex-grow    

`flex-grow`属性 前提在所有项目以`flex-basis`的值排列完如有剩余空间，则规定当前`flex item`的高度(主轴为水平轴则是高度，如果主轴为竖直轴则是宽度，主轴通过`flex-direction`设置)的剩余空间份比例  

```css  
flex-grow: 0(默认值) | <number>;
```  
**number**：当剩余空间足够时，如果值为`1`那么高度(宽度)`放大剩余空间1份`，如果只有一个元素设置了此属性，那么剩余全部全部分给他；如果多个元素设置了此属性，比如`a:1` `b:2` `c:3`则会把剩余空间分成`6份`，a在本身的大小基础上增加剩余空间的`1/6份` a则是`2/6份` c则是`3/6份`，如果剩余空间不足，则取`剩余空间最大值`，不支持负数
 
### flex-shrink    

`flex-shrink`属性规定了当剩余空间不足时候，当前`flex item`的高度(主轴为水平轴则是高度，如果主轴为竖直轴则是宽度，主轴通过`flex-direction`设置)的缩小超出空间份比例，和上面的`flex-grow`是相反的，一个放大，一个缩小

```css  
flex-shrink: 1(默认值) | <number>;
``` 
**number**：当剩余空间不足时，默认按照比例1缩小，如果值为`2`那么高度(宽度)`缩小超出空间2份`
  
### align-self     

`flex-shrink`属性规定当前`flex item`作为一个整体在`flex`容器在副轴(非主轴，和`justify-content`相反)上的排列方式    

>注意：这个属性和`align-item`的区别是`align-item`影响全部`flex item`，而`align-self`只影响当前`flex item`  

```css  
align-self: auto(默认值) | stretch | center | flex-start | flex-end | baseline
``` 
**auto**：继承父亲`align-item`的`value`  
**stretch**： 占满整个主轴的高度, 前提必须是你没有设置item的高度(宽度)，否则不生效  
**center**： 位于主轴中心   
**flex-start**： 位于主轴开头    
**flex-end**： 位于主轴结尾  
**baseline**： 位于`item`中第一行文字 *基线* 对齐  
     
## 扩展一道面试题  
#### 一个元素包含内部两个元素。当flex水平轴为主轴，实现左固定，右自适应：  
  
```html 
<div class='flex2'>
	<p>1</p>
	<p>2</p>
</div>
```  

```css
div.flex2{
	display:flex;
	border:1px solid red;
	width:100%;
	height:300px;
}
.flex2 p{
	width:100px;
	height:100px;
	background-color: skyblue;
	line-height: 100px;
	text-align: center;
	font-size: 80px;
	border:1px solid blue;
}
.flex2 p:nth-child(1){
	width:200px;// 左侧元素固定为200px宽
}
.flex2 p:nth-child(2){
	flex-grow:1; // 右侧元素自适应
}
```
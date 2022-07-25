# 双飞与圣杯布局本质

<!--more-->
>经典的双飞翼与圣杯布局，主要利用了CSS盒子模型的特性，以float布局为主要方法，结合Position属性。
### 前置知识了解
#### [CSS盒子模型](https://www.runoob.com/css/css-boxmodel.html)
* 我们需要先了解CSS的盒子模型，详细的参考标题链接内容，关键点在于当我们指定一个CSS元素的宽度和高度属性时，只是设置内容区域的宽度和高度，要知道，完整大小的元素，还必须添加内边距，边框和外边距。
* 当然我们也可以添加一个属性设置整个盒子的总宽度和高度
{{< highlight html >}}
/* 加上这个css3的新特性，可以自动计算，保持总的宽高是设置的值 */
            box-sizing: border-box;
{{< /highlight >}}
#### 负值的margin带来的影响
一般来说，我们都是给一个元素设置margin正值来调整与其他元素的距离，如果我们给margin设置负值会发生什么呢？
我们分两种情况讨论：
##### 水平方向
* 1、margin-left设置负值，会使自身以及后面的元素一起往左移动，移动的距离可以给具体值，也可以设置百分比(相对父元素的宽度content的百分比)
* 2、margin-right设置负值，自身不发生改变，但会令后面的元素往左移动，移动的距离原理同上。
##### 垂直方向
* 1、margin-top设置负值，会使自身以及后面的元素一起往上移动，移动的距离可以给具体值，也可以设置百分比(相对父元素的高度content的百分比)
* 2、margin-bottom设置负值，自身不发生改变，但会令后面的元素往上移动，移动的距离原理同上。     
-------------------------**可以在下方CSS指定位置中测试**
<p class="codepen" data-height="614" data-default-tab="html,result" data-slug-hash="mdxWYgQ" data-editable="true" data-user="liaozhijie8" style="height: 614px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/liaozhijie8/pen/mdxWYgQ">
  负值的margin带来的影响</a> by liaozhijie8 (<a href="https://codepen.io/liaozhijie8">@liaozhijie8</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

#### [CSS Float(浮动)](https://www.runoob.com/css/css-float.html)
* 在网页流中，所有元素都是一个盒子，类似于玩俄罗斯方块一样，如果一行宽度不够放下一个完整的积木，那么这块积木将会放到下一行，慢慢地充满整个网页。这是一个二维的平面，如果转变为三维模型，把每一个元素再加上一个厚度。假设在排列过程中每一块元素的厚度一致，一行排不下同样需要换行，但是有些元素我们还是希望他们在这一行排列，那么就把他们排列在第二层中，这就是CSS float特性，具体参考标题链接。

#### [CSS Position(定位)](https://www.runoob.com/css/css-positioning.html)
* 有时候我们希望元素能按照给定的坐标定位，例如**坐标(x,y)**,那么我们只需设置一个参考的原点即可，那就是position属性，详情点击链接。
### 双飞翼布局
#### 原理
* 双飞翼布局左中右三列布局，渲染顺序中间列书写在前保证提前渲染，左右两列定宽，中间列自适应剩余宽度。
* 双飞翼布局与圣杯布局的不同之处，圣杯布局的的左中右三列容器，中间middle多了一个子容器存在，通过控制 middle 的子容器的 margin或者padding空出左右两列的宽度。
<p class="codepen" data-height="399" data-default-tab="html,result" data-slug-hash="XWERJGv" data-editable="true" data-user="liaozhijie8" style="height: 399px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/liaozhijie8/pen/XWERJGv">
  双飞布局</a> by liaozhijie8 (<a href="https://codepen.io/liaozhijie8">@liaozhijie8</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

* 1、左中右三列都设置float左浮动
* 2、中间列设置100%的宽度(达到自适应剩余宽度的效果)，并包含一个子元素(用于显示内容)，设置子元素的左右padding或者margin(防止显示内容被左右两列覆盖)
* 3、左右列设置固定的宽度
* 4、浮动的元素与浮动的元素同样会受盒子模型的影响而正常元素一样排列，这个时候，我们给浮动的元素设置margin负值调整具体的位置
* 5、左列元素我们希望它移动到最左边，设置margin-left:100%，这个100%是相对父元素的，因为它与中间列同属一个父元素，所以刚好移动到最左边
* 6、右列元素因为设置了浮动，并没有紧挨着左列元素，而是遇到中间列的元素边框排在了其身后，设置margin-left：自身宽度，使他排列在最右边
* **具体设置可以修改上图参数观看变化过程**

### 圣杯布局
* 圣杯布局与双飞翼布局的不同点在于，左中右三列包含在同一个父元素下，中间列设置padding或者margin留出位置给左右两列
* 左列元素尽管设置了margin-left:-100%,仍然无法移动到最右边，因为它与中间列同属一个父元素，所以需要利用position属性往左移动自身宽度
* 右列元素有两种方法设置，1)设置margin-right:-自身宽度，其元素后面的元素会往前移动 2)设置position:相对定位，margin-left：-自身宽度，right:-自身宽度;
<p class="codepen" data-height="410" data-default-tab="html,result" data-slug-hash="qBomBgg" data-editable="true" data-user="liaozhijie8" style="height: 410px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/liaozhijie8/pen/qBomBgg">
  圣杯布局</a> by liaozhijie8 (<a href="https://codepen.io/liaozhijie8">@liaozhijie8</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

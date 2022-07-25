# 各种居中

<!--more-->
### 水平居中
根据元素的类型不同可以分为两类情况
* 一类使行内元素，它不能设置宽高，所以需要给其父元素设置一个text-align: center;
* 一类使块级元素，其可以设置宽高，
   * 在自身已经设置宽高具体参数的情况下，可以给margin：0 auto
   * 利用position属性，相对父元素，移动左边或右边50%，然后再设置margin调整位置
  
<p class="codepen" data-height="409" data-default-tab="html,result" data-slug-hash="rNdmJMN" data-editable="true" data-user="liaozhijie8" style="height: 409px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/liaozhijie8/pen/rNdmJMN">
  各种水平居中</a> by liaozhijie8 (<a href="https://codepen.io/liaozhijie8">@liaozhijie8</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

### 垂直居中
<p class="codepen" data-height="415" data-default-tab="html,result" data-slug-hash="LYdyQzW" data-editable="true" data-user="liaozhijie8" style="height: 415px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/liaozhijie8/pen/LYdyQzW">
  垂直居中</a> by liaozhijie8 (<a href="https://codepen.io/liaozhijie8">@liaozhijie8</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

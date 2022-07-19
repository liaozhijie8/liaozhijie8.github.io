# ES6变量

### var、let、const的几点区别
### 1、重复声明
let和const都不允许，var可以。为了规范编程减少使用var，其实在vue**API**中也大多数情况下用**const**去初始化数据，主要是要获得响应式的数据，我们都不会直接去重新赋值
```
<!-- 基本数据类型 -->
const name = 'tom'
name = 'jack' //error
<!-- 引用数据类型 -->
const person = {
  name:'tom',
  age:22
}
person = {
  name:'leon',
  age:33
} //error
//采用另一种修改方式
person.name = 'leon' //成功修改
```
### 2、变量提升：
var会提升变量声明到当前作用域的前面
```
function test() {
  var i = 22
}
console.log(i) //22
```
### 3、暂时性死区
只要作用域内存在let、const，他们所声明的变量或常量就会自动绑定这个区域，不再受外部作用域的影响
```
function test() {
  let i = 22
}
console.log(i) //error
```
### 4、window对象的属性和方法
全局作用域下，var声明的变量，通过function声明的函数，会自动变成window对象的属性或方法
```
var age = 18
console.log(window.age) ==18
function add(){
  console.log('你好')
}
window.add === add //全等
```
但是使用const\let声明的变量或function都不会变为window中的
### 5、作用域链
```
function func(){
  for(let i = 0; i < 4;i ++){
    console.log(i)
  }
}
func() //当函数被调用即产生了一个函数作用域
console.log(i) //error i 未定义
```
### 原理图
{{< mermaid >}}
stateDiagram-v2
    state window全局作用域 {
        [*] --> func函数作用域

        state func函数作用域 {
            [*] --> for和let构成块作用域

            state for和let构成块作用域 {
                [*] --> i
                i --> 往外作用域寻找
            }
        }
    }

{{< /mermaid >}}
_一些块级作用域_
```
{} //花括号以及带有{}的一些方法
for(let){} // 需要和const let 配合
while(){}
if(){}
// 等等
function fuc(){} //函数作用域
```
### 应用
### 绑定事件案例
```
<button>按钮1</button>
<button>按钮2</button>
<button>按钮3</button>
<!-- 绑定点击事件 -->
var btn = document.querSelectorAll('button')
//给每一个按钮绑定点击事件
for(var i = 0;i < btn.length;i ++) {
  btn[i].addEventListener('click',function(){
    console.log(i)
  })
}
```
当我们点击分别点击按钮时，期望得到的时相应的i，即0，1，2(实际全是2)。但是由于绑定事件属于{块级作用域}，而var定义的i是具有函数级作用域的，所有在每一个绑定事件的{块级作用域}中是找不到i的值，故会出到外层查找，外层循环结束后i的值为2。

### 解决方法一
在ES6之前我们可以用闭包的办法解决
```
<!-- 绑定点击事件 -->
var btn = document.querSelectorAll('button')
//给每一个按钮绑定点击事件
for(var i = 0;i < btn.length;i ++) {
  //创建一个即时函数
  (function(index){
    btn[i].addEventListener('click',function(){
    console.log(index)
  })
  })(i)
}
```
### 原理图
{{< mermaid >}}
stateDiagram-v2
    state window全局作用域 {

        state func(i=0)函数作用域 {
          
            [*] --> (index=0)构成块作用域 index=0
            (index=0)构成块作用域 --> 外层func中查找
            [*] --> index=0
            外层func中查找 --> index=0
        }

        state func(i=1)函数作用域 {
          
            [*] --> (index=1)构成块作用域 index=1
            (index=1)构成块作用域 --> 外层func1中查找
            [*] --> index=1
            外层func1中查找 --> index=1
        }

        state func(i=2)函数作用域 {
          
            [*] --> (index=2)构成块作用域 index=2
            (index=2)构成块作用域 --> 外层func2中查找
            [*] --> index=2
            外层func2中查找 --> index=2
        }
    }

{{< /mermaid >}}

### 解决方法二
ES6以后，我们采用let 或 const 锁定作用域范围，达到和上面采用闭包一样的效果
```
<!-- 绑定点击事件 -->
let btn = document.querSelectorAll('button')
//给每一个按钮绑定点击事件
for(let i = 0;i < btn.length;i ++) {
  btn[i].addEventListener('click',function(){
    console.log(i)
  })
}

//0 1 2
```
### 原理图
{{< mermaid >}}
stateDiagram-v2
    state window全局作用域 {

        state func(i=0)函数作用域 {
          
            [*] --> (let=0)构成块作用域 i=0
            (let=0)构成块作用域 --> 外层func0中查找
            [*] --> i=0
            外层func0中查找 --> i=0
        }

        state func(i=1)函数作用域 {
          
            [*] --> (let=1)构成块作用域 i=1
            (let=1)构成块作用域 --> 外层func1中查找
            [*] --> i=1
            外层func1中查找 --> i=1
        }

        state func(i=2)函数作用域 {
          
            [*] --> (let=2)构成块作用域 i=2
            (let=2)构成块作用域 --> 外层func2中查找
            [*] --> i=2
            外层func2中查找 --> i=2
        }
    }

{{< /mermaid >}}


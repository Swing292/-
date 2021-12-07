[TOC]



# jQuery基础

## 使用

1. 下载jQuery库

   https://jquery.com/

2. 引入下载的库：

   在head标签里面引入：`<script src="jquery-migrate-1.4.1.js"></script>`

3. 编写jQuery程序“

   ```js
   <script>
           // 原生js
           window.onload=function(ev){}
           // jquery的入口函数
           $(document).ready(function(){})
   </script>
   ```

## 加载模式

- 获取页面图片：

  ```js
  <script>
          // 原生js
          window.onload=function(ev){
              var img=document.getElementsByTagName("img")[0];
              console.log(img);
  
              // 可以拿到dom元素的宽高
              var width=window.getComputedStyle(img).width;
              console.log(width);
          }
          // jquery
          $(document).ready(function(){
              // 获取dom元素
              var $img=$("img")[0];
              console.log(img);
  
              // 不可以拿到dom元素的宽高
              var $width=$img.width();
          })
      </script>
  ```

- 元素js和jQuery的入口函数的加载模式不同

  - js：等dom元素和图片加载完毕才会执行
  - jQuery：等dom元素加载完毕，但不等图片加载完毕就执行

- jQuery中编写多个入口函数，会一个个执行，后面的不会覆盖前面的

## 不同写法和冲突问题

1. 入口函数的写法：

   ```js
   <script>
           // 第一种写法
           $(document).ready(function(){
   
           })
           
           // 第二种写法
           jQuery(document).ready(function(){
   
           })
   
           // 第三种写法（推荐）
           $(function(){
   
           })
   
           // 第四种写法
           jQuery(function(){
   
           })
       </script>
   ```

2. 冲突问题

   - 原因：$符号可能会引起冲突问题
   - 解决：
     1. 在jQuery代码前释放$的使用权：`jQuery.noConflict();`
     2. 自定义访问符号（可以用其他代替$）：`var ng=jQuery.noConflict();`

# 核心函数

## 传输函数

1. 调用jQuery的核心函数：`$();`

2. 接收的参数：

   ```js
       <script>
           // 1. 函数
           $(function(){
   
           // 字符串
           // 2. 字符串选择器(返回一个jQuery对象，对象中保存了创建的DOM元素)
            var $box1=$("#box1");
   
           // 3. 代码片段(返回一个jQuery对象，对象中保存了创建的DOM元素)
           var $p=$("<p>创建的段落</p>");
           $box1.append($p);
   
           // 4. DOM元素(传过去的元素会被包装成一个jQuery对象返回)
           var span=document.getElementsByTagName("span"[0]);
           var $span=$(span);
           })
   
       </script>
   ```

## jQuery对象

1. 伪数组：有0到length-1的属性，并且有length属性
2. 是一个伪数组

## 方法

1. 方法的创建和调用

   ```js
       <script>
           // 定义一个类
           function AClass(){
   
           }
           // 1.添加一个静态方法（直接添加的）
           AClass.staticMethod=function(){
               alert("静态方法");
           }
           // 通过类名调用静态方法
           AClass.staticMethod();
   
           // 2.添加一个实例方法
           AClass.prototype.instanceMethod=function(){
               alert("实例方法");
           }
           // 通过类的实例调用
           var a=new AClass();
           a.instanceMethod();
       </script>
   ```

2. each静态方法

   ```js
       <script>
           // 原生的forEach遍历（不能遍历伪数组）
           var arr=[1,2,3,4,5];
           // 参数一：遍历到的元素  参数二：遍历到的索引
           arr.forEach(function(value,index){
               console.log(index,value);
           })
   
           // 使用jQuery（可以遍历伪数组）
           // 参数一：遍历到的索引  参数二：遍历到的元素
           $.each(arr,function(index,value){
               console.log(index,value);
           })
       </script>
   ```

3. map方法
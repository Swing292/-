[TOC]

# HTML

略

# CSS

## 基本语法

### css编写位置

1. 在标签内通过style属性设置

   ```css
       <a href="" style="color: aquamarine;">网页</a>
   ```

2. 编写到head中的style标签中

   ```css
   <head>
       <meta charset="UTF-8">
       <title>Document</title>
   
       <style>
           p{
               color: aquamarine;
           }
       </style>
   </head>
   ```

3. 在head中引入外部样式表

   ```css
   <head>
       <meta charset="UTF-8">
       <title>Document</title>
   
       <link 
   el="stylesheet" href="style.css">
   </head>
   ```

### css选择器

#### 选择器的分类

1. **元素选择器**：
   
   - 根据标签名选中元素
   
   - 语法：标签名{}
   
2. **id选择器**：

   - 根据id属性选中
   - 语法：#属性值{}

3. **类选择器**：根据class属性选中元素

   - 语法：. class属性值{}

4. **通配选择器**：选中所有元素

   - 语法：*{}

5. **复合选择器**：

   1. 交集选择器：
      - 满足多个条件的元素
      - 语法：选择器1选择器2…{}
   2. 并集选择器：
      - 选择满足选择器的所有元素
      - 语法：选择器1，选择器2，…{}

6. **关系选择器**：

   1. 子元素选择器：
      - 选中指定父元素的子元素
      - 语法：父元素 > 子元素 {}
   2. 后代元素选择器：
      - 选中指定元素内的指定后代元素
      - 语法：祖先 后代{}
   3. 兄弟元素选择器：
      1. 选中下一个兄弟：
         - 语法：前一个＋后一个{}
      2. 选中后面所有兄弟：
         - 语法：兄~弟{}

7. **属性选择器**：

   1. [属性名]：选择含有指定属性的元素

      ```css
      p[title]{
          color:red;
      }
      ```

   2. [属性名=属性值]

      ```css
      p[title=animal]{
          color:red;
      }
      ```

   3. [属性名^=属性值]：  选择属性值以指定值开头的元素

   4. [属性名￥=属性值]：选择属性值以指定值结尾的元素

   5. [属性名*=属性值]：选择属性值中含有某值的元素

8. **伪类选择器**：

   - 全部类型下排序：
     - :first-child 选中第一个子元素
     - :last-child 选中最后一个子元素
     - :nth-child()  选中该第n个子元素
       - 特殊值：n（选中第n个）、2n/even（偶数位）、2n+1/odd（奇数位）
   - 同一类型下排序：
     - :first-of-type
     - :last-of-type
     - :nth-of-type()
   - :not() 否定伪类：将符合条件的元素从选择器中去除

9. **超链接的伪类**：

   - :link 表示没访问过的链接
   - :visited 表示访问过的链接（只能改颜色）
   - :hover 表示鼠标移入的状态
   - :active 表示鼠标点击

10. **伪元素**：

    - 使用::开头

    - ::first-letter 表示第一个字母

    - ::first-line 表示第一行

    - ::selection 表示选中的内容

    - ::before 元素的开始（需要结合content属性使用）

    - ::after 元素的最后（需要结合content属性使用）

      ```css
      div::before{
          content:'在前面插入这几个字';
          color:red;
      }
      ```

#### 选择器的权重

- !important 最高优先级（慎用）

  ```css
  .red{
      background-color:red !important;
  }
  ```

- 内联样式 1000

- id选择器 100

- 类和伪类选择器 10

- 元素选择器 1

- 通配选择器 0

- 继承的样式 无

  **权重相同时，优先使用靠下的**

### 长度单位

1. 像素

2. 百分比：

   - 相对于父元素属性的百分比

3. em：

   - 相对于元素的字体大小计算

   - 1em=1font-size（根据字体大小的改变而改变）

     ```css
     .box{
         font-size:30px;
         width:10em;/*大小：300px */ 
     }
     ```

4. rem：

   - 相对于根元素的字体大小计算

     ```css
     html{
         font-size:20px;
     }
     .box{
         width:10rem;/*大小：200p */
     }
     ```


### 文档流

1. 元素在文档流中的特点：
   - 块元素：
     - 在页面中独占一行（垂直排列）
     - 默认宽度是父元素的全部
     - 默认高度是内容的高度
   - 行内元素：
     - 不独占一行，只占自身大小（自左向右排列）
     - 默认高度和高度都是内容的高度

### 盒子模型

1. 盒子组成：

   - 内容区（content）
   - 内边距（padding）
   - 边框（border）
   - 外边距（margin）

2. 内容区：

   - 元素中所有的子元素和文本内容都在内容区排列
   - 内容区的大小设置：
     - width
     - height

3. 边框：

   - **边框的宽度 border-width**
     - border-width:10px;（默认值：3个像素）
     - 值的设置：

   - **边框的颜色 border-color**
   - **边框的样式 border-style**








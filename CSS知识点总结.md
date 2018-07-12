### margin: 0 auto
实现块状元素的水平居中

### 伪类&伪元素
伪类：（:active,:focus,:hover,:link,:visited,:first-child,:lang）

伪元素：（::first, ::first-line, ::before, ::after）

区别：伪类是元素的一个状态，伪元素需要创建一个新的元素才能完成样式。

如清除浮动时，提供了content属性，可以添加内容。(参考清除浮动方法2)

### 清除浮动
浮动（http://www.cnblogs.com/iyangyuan/archive/2013/03/27/2983813.html）

    <div class="outer">
        <div class="div1">1</div>
        <div class="div2">2</div>
        <div class="div3">3</div>
    </div>
    
假设1,2,3都是左浮动。

##### 方法1
添加新元素，利用clear:both

    <div class="outer">
        <div class="div1">1</div>
        <div class="div2">2</div>
        <div class="div3">3</div>
        <div class="clear"></div>
    </div>
    
    .clear{clear:both; height: 0;line-height: 0;font-size: 0}
    
##### 方法2
::after添加内容块，完成清除，方法1的升级版

    .outer :after::{clear:both; content: '.';display:block;width:0;height:0;visibility:hidden}
    .outer{zoom:1}

### 水平垂直居中

##### 方法1
flex水平垂直居中

    <div class="parent">
      <div class="child">Demo</div>
    </div>
    
    <style>
      .parent {
        display: flex;
        justify-content: center; /* 水平居中 */
        align-items: center; /*垂直居中*/
      }
    </style>
    
##### 方法2
table-cell水平垂直居中

    <div class="parent">
      <div class="child">Demo</div>
    </div>
    
    <style>
      .parent {
        text-align: center;//水平居中
        display: table-cell;
        vertical-align: middle;//垂直居中
      }
      .child {
        display: inline-block;//防止块级元素宽度独占一行，内联元素可不设置
      }
    </style>
    
##### 方法3
absolute+transform水平垂直居中
    
    <div class="parent">
      <div class="child">Demo</div>
    </div>
    
    <style>
      .parent {
        position: relative;
      }
      .child {
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
      }
    </style>

##### 方法4
absolute+margin

    .parent{
        position:relative;
    }
    .child{
         width: 100px;
         height: 100px;
         position: absolute;
         top: 50%;
         left: 50%;
         margin: -50px 0 0 -50px;
    }
    
##### 水平居中

- 对于行内元素(inline)：text-align: center;
- 对于块级元素(block)：设置宽度且 marigin-left 和 margin-right 是设成 auto
- 对于多个块级元素：对父元素设置 text-align: center;，对子元素设置 display: inline-block;；或者使用 flex 布局

##### 垂直居中

对于行内元素(inline)

- 单行：设置上下 pandding 相等；或者设置 line-height 和 height 相等
- 多行：设置上下 pandding 相等；或者设置 display: table-cell; 和 vertical-align: middle;；或者使用 flex 布局；或者使用伪元素

对于块级元素(block)：下面前两种方案，父元素需使用相对布局

- 已知高度：子元素使用绝对布局 top: 50%;，再用负的 margin-top 把子元素往上拉一半的高度
- 未知高度：子元素使用绝对布局 position: absolute; top: 50%; transform: translateY(-50%);
使用 Flexbox：选择方向，justify-content: center;


### 自适应布局

    <div class="parent">
        <div class="left"><p>left</p></div>
        <div class="right-fix">
            <div class="right">
                <p>right</p><p>right</p>
            </div>
        </div>
    </div>

##### 左列定宽，右列自适应
方法1

margin + float

     .left{
        float: left;     //向左浮动
        width: 100px;    //固定宽度
        position: relative;//由于.left与.right-fix重合，且.right-fix在DOM树上的位置比.left要后，因此.right-fix会遮挡住.left，设置.left为relative可以让其冒出来。
    }
    .right-fix{
        float: right;     //向右浮动
        width: 100%;    //为了自适应设为100%
        margin-left: -100px;//由于宽度设为100%，.right-fix遭到浏览器换行处理；因此通过设置负的margin值，在左侧制造出100px的空白，使.right-fix与.left重合（即处于同一行）
    }
    .right{
        margin-left: 120px;    //由于.left和.right-fix重合了，因此给.right设置一个margin-left，避免内容区（.right）与.left重合。另外，120px - 100px = 多出来的20px实际上就相当于.left和.right之间的间隔了。
    }
    
方法2

absolute

    <div class="parent">
        <div class="left">
            <p>left</p>
        </div>
        <div class="right">
            <p>right</p>
            <p>right</p>
        </div>
    </div>
    .parent{
        position: relative;
    }
    .left{
        position: absolute;
        left: 0;
        width: 100px;
    }
    .right{
        position: absolute;
        left: 120px;    //比.left的left多出20px，相当于间隔
        right: 0;
    }
    
##### 左列不定宽，右列自适应

方法1

float + BFC
    
    <div class="parent">
        <div class="left">
            <p>left</p>
        </div>
        <div class="right">
            <p>right</p>
            <p>right</p>
        </div>
    </div>
    
    .left{
        float: left;
        width: 100px;
        margin-right: 20px;    //形成20px的间隔
    }
    .right{
        overflow: hidden; //通过设置overflow: hidden来触发BFC特性
    }
    
方法2

table（比较老旧）

    .parent{
        display: table; width: 100%;
        table-layout: fixed;
    }
    .left,.right{
        display: table-cell;
    }
    .left{
        width: 100px;
        padding-right: 20px;
    }
    
方法3

flex

    .parent{
        display: flex;
    }
    .left{
        margin-right: 20px;
    }
    .right{
        flex: 1;
    }
    .left p{width: 200px;}
    
### BFC
块格式化上下文（Block Formatting Context，BFC） 是Web页面的可视化CSS渲染的一部分，是布局过程中生成块级盒子的区域，也是浮动元素与其他元素的交互限定区域。

- 根元素或包含根元素的元素
- 浮动元素（元素的 float 不是 none）
- 绝对定位元素（元素的 position 为 absolute 或 fixed）
- 行内块元素（元素的 display 为 inline-block）
- 表格单元格（元素的 display为 table-cell，HTML表格单元格默认为该值）
- 表格标题（元素的 display 为 table-caption，HTML表格标题默认为该值）
匿名表格单元格元素（元素的 display为 table、table-row、 table-row-group、table-header-group、table-footer-group（分别是HTML table、row、tbody、thead、tfoot的默认属性）或 inline-table）
- overflow 值不为 visible 的块元素
- display 值为 flow-root 的元素
- contain 值为 layout、content或 strict 的元素
- 弹性元素（display为 flex 或 inline-flex元素的直接子元素）
- 网格元素（display为 grid 或 inline-grid 元素的直接子元素）


##### BFC作用

- 利用BFC避免外边距折叠
- BFC包含浮动
- 使用BFC避免文字环绕
- 在多列布局中使用BFC






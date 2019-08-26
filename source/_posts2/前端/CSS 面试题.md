---
title: CSS 面试题

categories:
- 前端

date: 2018-04-17
---

CSS 面试题

## 常见面试题

1. 哪个属性可用来改变背景颜色？
    - `bgcolor`
    - `color`
    - `background`
    - `backgroundColor`

1. 如何去掉文本超级链接的下划线？
    - `text-decoration:no underline`
    - `text-decoration:none`
    - `decoration:no underline`
    - `decoration:no underline`

1. 哪些HTML标记可以在页面中加入一个层？
    - `<floor>`
    - `<div>`
    - `<span>`
    - `<level>`

1. CSS中父元素的最后一个子节点如何选择？
    - `last`
    - `lastChild`
    - `last-child`
    - `children[children.length]`

1. CSS选择符有哪些？哪些属性可以继承？
    - id选择器（`#myid`）
    - 类选择器（`.myclassname`）
    - 标签选择器（`p, h1, p`）
    - 相邻选择器（`h1 + p`）
    - 子选择器（`ul > li`）
    - 后代选择器（`li a`）
    - 通配符选择器（`*`）
    - 属性选择器（`a[rel = "external"]`）
    - 伪类选择器（`a:hover, li:nth-child`）
    - 可继承的样式：`font-size font-family color, UL LI DL DD DT`
    - 不可继承的样式：`border padding margin width height`

1. CSS优先级算法如何计算？
    - 优先级就近原则，同权重情况下样式定义最近者为准
    - 载入样式以最后载入的定位为准
    - 优先级为:
        `!important >  id > class > tag`
        `important`比内联优先级高

1. CSS3新增伪类有那些？
    - `p:first-of-type`选择属于其父元素的首个元素的每个元素
    - `p:last-of-type`选择属于其父元素的最后元素的每个元素
    - `p:only-of-type`选择属于其父元素唯一的元素的每个元素
    - `p:only-child`选择属于其父元素的唯一子元素的每个元素
    - `p:nth-child(2)`选择属于其父元素的第二个子元素的每个元素
    - `:before`在元素之前添加内容,也可以用来做清除浮动
    - `:after` 在元素之后添加内容
    - `:enabled :disabled`控制表单控件的禁用状态
    - `:checked`单选框或复选框被选中

1. 如何居中`p`？
    - 水平居中
        给`p`设置一个宽度，然后添加`margin:0 auto`属性。
        ```css
        p{
            width:200px;
            margin:0 auto;
        }
        ```
    - 让绝对定位的`p`居中
        ```css
        p {
            position: absolute;
            width: 300px;
            height: 300px;
            margin: auto;
            top: 0;
            left: 0;
            bottom: 0;
            right: 0;
            background-color: pink; /* 方便看效果 */
        }
        ```
    - 水平垂直居中一
        确定容器的宽高，设置层的外边距。
        ```css
        p {
            position: relative;     /* 相对定位或绝对定位均可 */
            width:500px; 
            height:300px;
            top: 50%;
            left: 50%;
            margin: -150px 0 0 -250px;      /* 外边距为自身宽高的一半 */
            background-color: pink;     /* 方便看效果 */
        
        }
        ```
    - 水平垂直居中二
        未知容器的宽高，利用`transform`属性
        ```css
        p {
            position: absolute;     /* 相对定位或绝对定位均可 */
            width:500px; 
            height:300px;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: pink;     /* 方便看效果 */
        
        }
        ```
    - 水平垂直居中三
        利用`flex`布局，实际使用时应考虑兼容性。
        ```css
        .container {
            display: flex; 
            align-items: center;        /* 垂直居中 */
            justify-content: center;    /* 水平居中 */
        
        }
        .container p {
            width: 100px;
            height: 100px;
            background-color: pink;     /* 方便看效果 */
        }  
        ```
1. `display`有哪些值？说明他们的作用
    - `block` 块类型。默认宽度为父元素宽度，可设置宽高，换行显示
    - `none` 缺省值。象行内元素类型一样显示
    - `inline` 行内元素类型。默认宽度为内容宽度，不可设置宽高，同行显示
    - `inline-block` 默认宽度为内容宽度，可以设置宽高，同行显示
    - `list-item` 象块类型元素一样显示，并添加样式列表标记
    - `table` 此元素会作为块级表格来显示
    - `inherit` 规定应该从父元素继承`display`属性的值

1. `position`的值`relative`和`absolute`定位原点是？
    - `absolute` 生成绝对定位的元素，相对于值不为 static的第一个父元素进行定位。
    - `fixed` （老IE不支持）生成绝对定位的元素，相对于浏览器窗口进行定位。
    - `relative` 生成相对定位的元素，相对于其正常位置进行定位。
    - `static` 默认值。没有定位，元素出现在正常的流中（忽略`top, bottom, left, right z-index`声明）。
    - `inherit` 规定从父元素继承`position`属性的值。

1. CSS3有哪些新特性？
    - 新增各种CSS选择器（`: not(.input)`：所有`class`不是`input`的节点）
    - 圆角（`border-radius:8px`）
    - 多列布局（`multi-column layout`）
    - 阴影和反射（`Shadow\Reflect`）
    - 文字特效（`text-shadow`）
    - 文字渲染（`Text-decoration`）
    - 线性渐变（`gradient`）
    - 旋转（`transform`）
    - 缩放,定位,倾斜,动画,多背景 `transform: scale(0.85,0.90) translate(0px,-30px) skew(-9deg,0deg)`

1. 请解释一下CSS3的Flexbox（弹性盒布局模型）,以及适用场景？
    一个用于页面布局的全新CSS3功能，Flexbox可以把列表放在同一个方向（从上到下排列，从左到右），并让列表能延伸到占用可用的空间。
    较为复杂的布局还可以通过嵌套一个伸缩容器（flex container）来实现。
    采用Flex布局的元素，称为Flex容器（flex container），简称“容器”。
    它的所有子元素自动成为容器成员，称为Flex项目（flex item），简称“项目”。
    常规布局是基于块和内联流方向，而Flex布局是基于`flex-flow`流可以很方便的用来做局中，能对不同屏幕大小自适应。
    在布局上有了比以前更加灵活的空间。

1. 用纯CSS创建一个三角形的原理是什么？
    把上、左、右三条边隐藏掉（颜色设为`transparent`）
    ```css
    #demo {
    width: 0;
    height: 0;
    border-width: 20px;
    border-style: solid;
    border-color: transparent transparent red transparent;
    }
    ```

1. 一个满屏 “品” 字布局 如何设计?
    - 上面的`p`宽`100%`，
    - 下面的两个`p`分别宽`50%`，
    - 然后用`float`或者`inline`使其不换行即可

1. CSS多列等高如何实现？
    利用`padding-bottom|margin-bottom`正负值相抵。
    设置父容器设置超出隐藏（`overflow:hidden`），这样子父容器的高度就还是它里面的列没有设定`padding-bottom`时的高度，
    当它里面的任 一列高度增加了，则父容器的高度被撑到里面最高那列的高度，
    其他比这列矮的列会用它们的`padding-bottom`补偿这部分高度差。

1. `li`与`li`之间有看不见的空白间隔是什么原因引起的？有什么解决办法？
    行框的排列会受到中间空白（回车\空格）等的影响，因为空格也属于字符,这些空白也会被应用样式，占据空间，所以会有间隔，把字符大小设为0，就没有空格了。

1. 为什么要初始化CSS样式？
    因为浏览器的兼容问题，不同浏览器对有些标签的默认值是不同的，如果没对CSS初始化往往会出现浏览器之间的页面显示差异。   
    当然，初始化样式会对SEO有一定的影响，但鱼和熊掌不可兼得，但力求影响最小的情况下初始化。
    最简单的初始化方法：`* {padding: 0; margin: 0;}`（强烈不建议）
    淘宝的样式初始化代码：
    ```css
    body, h1, h2, h3, h4, h5, h6, hr, p, blockquote, dl, dt, dd, ul, ol, li, pre, form, fieldset, legend, button, input, textarea, th, td { margin:0; padding:0; }
    body, button, input, select, textarea { font:12px/1.5tahoma, arial, \5b8b\4f53; }
    h1, h2, h3, h4, h5, h6{ font-size:100%; }
    address, cite, dfn, em, var { font-style:normal; }
    code, kbd, pre, samp { font-family:couriernew, courier, monospace; }
    small{ font-size:12px; }
    ul, ol { list-style:none; }
    a { text-decoration:none; }
    a:hover { text-decoration:underline; }
    sup { vertical-align:text-top; }
    sub{ vertical-align:text-bottom; }
    legend { color:#000; }
    fieldset, img { border:0; }
    button, input, select, textarea { font-size:100%; }
    table { border-collapse:collapse; border-spacing:0; }
    ```

1. `absolute`的containing block(容器块)计算方式跟正常流有什么不同？
    无论属于哪种，都要先找到其祖先元素中最近的`position`值不为`static`的元素，然后再判断：
    - 若此元素为`inline`元素，则containing block为能够包含这个元素生成的第一个和最后一个inline box的padding box（除`margin`，`border`外的区域）的最小矩形
    - 若此元素不为`inline`元素,则由这个祖先元素的padding box构成。
    - 如果都找不到，则为initial containing block

1. CSS里的`visibility`属性有个`collapse`属性值是干嘛用的？在不同浏览器下以后什么区别？

1. `position`跟`display`、`margin collapse`、`overflow`、`float`这些特性相互叠加后会怎么样？

1. 对BFC规范(块级格式化上下文：block formatting context)的理解？

1. css定义的权重
    以下是权重的规则：标签的权重为1，class的权重为10，id的权重为100，以下例子是演示各种定义的权重值：
    ```css
    /*权重为1*/
    p{
    }
    /*权重为10*/
    .class1{
    }
    /*权重为100*/
    #id1{
    }
    /*权重为100+1=101*/
    #id1 p{
    }
    /*权重为10+1=11*/
    .class1 p{
    }
    /*权重为10+10+1=21*/
    .class1 .class2 p{
    }
    ```
    如果权重相同，则最后定义的样式会起作用，但是应该避免这种情况出现

1. 请解释一下为什么需要清除浮动？清除浮动的方式
    清除浮动是为了清除使用浮动元素产生的影响。浮动的元素，高度会塌陷，而高度的塌陷使我们页面后面的布局不能正常显示。
    - 父级`p`定义`height`
    - 父级`p`也一起浮动
    - 常规的使用一个class
        ```css
        .clearfix:before, .clearfix:after {
                content: " ";
                display: table;
            }
            .clearfix:after {
                clear: both;
            }
            .clearfix {
                *zoom: 1;
            }
        
        ```
    - SASS编译的时候，浮动元素的父级`p`定义伪类`:after`
        ```css
        &:after,&:before{
                content: " ";
                visibility: hidden;
                display: block;
                height: 0;
                clear: both;
            }
        ```
    解析原理：
        -  `display:block`使生成的元素以块级元素显示,占满剩余空间;
        -  `height:0`避免生成内容破坏原有布局的高度。
        -  `visibility:hidden`使生成的内容不可见，并允许可能被生成内容盖住的内容可以进行点击和交互;
        - 通过 `content:"."`生成内容作为最后一个元素，至于`content`里面是点还是其他都是可以的，例如oocss里面就有经典的`content:"."`,有些版本可能`content`里面内容为空,一丝冰凉是不推荐这样做的,firefox直到7.0`content:""`仍然会产生额外的空隙；
        - `zoom：1` 触发IE hasLayout。

1. `zoom: 1`的清楚浮动原理?
    - 清楚浮动，触发hasLayout
        Zoom属性是IE浏览器的专有属性，它可以设置或检索对象的缩放比例。解决ie下比较奇葩的bug。
        譬如外边距（margin）的重叠，浮动清除，触发ie的haslayout属性等。
    - 来龙去脉大概如下
        当设置了zoom的值之后，所设置的元素就会就会扩大或者缩小，高度宽度就会重新计算了，这里一旦改变zoom值时其实也会发生重新渲染，运用这个原理，也就解决了ie下子元素浮动时候父元素不随着自动扩大的问题。
    - Zoom属是IE浏览器的专有属性，火狐和老版本的webkit核心的浏览器都不支持这个属性。然而，zoom现在已经被逐步标准化，出现在 CSS 3.0 规范草案中。
    - 目前非ie由于不支持这个属性，它们又是通过什么属性来实现元素的缩放呢？
        可以通过css3里面的动画属性`scale`进行缩放。

1. 移动端的布局用过媒体查询吗？
    假设你现在正用一台显示设备来阅读这篇文章，同时你也想把它投影到屏幕上，或者打印出来，而显示设备、屏幕投影和打印等这些媒介都有自己的特点，CSS就是为文档提供在不同媒介上展示的适配方法。
    当媒体查询为真时，相关的样式表或样式规则会按照正常的级联规被应用。
    当媒体查询返回假， 标签上带有媒体查询的样式表 仍将被下载 （只不过不会被应用）。
    包含了一个媒体类型和至少一个使用 宽度、高度和颜色等媒体属性来限制样式表范围的表达式。
    CSS3加入的媒体查询使得无需修改内容便可以使样式应用于某些特定的设备范围。
    ```css
    @media (min-width: 700px) and (orientation: landscape){
        .sidebar {
            display: none;
        }
    }
    ```

1. 使用CSS预处理器吗？喜欢那个？
    SASS (SASS、LESS没有本质区别，只因为团队前端都是用的SASS)

1. CSS优化、提高性能的方法有哪些？
    - 关键选择器（key selector）。选择器的最后面的部分为关键选择器（即用来匹配目标元素的部分）；
    - 如果规则拥有 ID 选择器作为其关键选择器，则不要为规则增加标签。过滤掉无关的规则（这样样式系统就不会浪费时间去匹配它们了）；
    - 提取项目的通用公有样式，增强可复用性，按模块编写组件；增强项目的协同开发性、可维护性和可扩展性;
    - 使用预处理工具或构建工具（gulp对css进行语法检查、自动补前缀、打包压缩、自动优雅降级）；

1. 浏览器是怎样解析CSS选择器的？
    样式系统从关键选择器开始匹配，然后左移查找规则选择器的祖先元素。
    只要选择器的子树一直在工作，样式系统就会持续左移，直到和规则匹配，或者是因为不匹配而放弃该规则。    

1. 在网页中的应该使用奇数还是偶数的字体？为什么呢？

1. `margin`和`padding`分别适合什么场景使用？
    - `margin`是用来隔开元素与元素的间距；
    - `padding`是用来隔开元素与内容的间隔。
    - `margin`用于布局分开元素使元素与元素互不相干；
    - `padding`用于元素与内容之间的间隔，让内容（文字）与（包裹）元素之间有一段

1. 抽离样式模块怎么写，说出思路，有无实践经验？[阿里航旅的面试题]

1. 元素竖向的百分比设定是相对于容器的高度吗？

1. 全屏滚动的原理是什么？用到了CSS的那些属性？

1. 什么是响应式设计？响应式设计的基本原理是什么？如何兼容低版本的IE？

1. 视差滚动效果，如何给每页做不同的动画？（回到顶部，向下滑动要再次出现，和只出现一次分别怎么做？）

1. `::before`和`:after`中双冒号和单冒号 有什么区别？解释一下这2个伪元素的作用。
    - 单冒号(`:`)用于CSS3伪类，双冒号(`::`)用于CSS3伪元素。（伪元素由双冒号和伪元素名称组成）
    - 双冒号是在当前规范中引入的，用于区分伪类和伪元素。不过浏览器需要同时支持旧的已经存在的伪元素写法，比如`:first-line`、`:first-letter`、`:before`、`:after`等，而新的在CSS3中引入的伪元素则不允许再支持旧的单冒号的写法。
    - 想让插入的内容出现在其它内容前，使用`::before`，否者，使用`::after`；
    - 在代码顺序上，`::after`生成的内容也比`::before`生成的内容靠后。
    - 如果按堆栈视角，`::after`生成的内容会在`::before`生成的内容之上

1. 如何修改chrome记住密码后自动填充表单的黄色背景 ？
    ```css
    input:-webkit-autofill, textarea:-webkit-autofill, select:-webkit-autofill {
    background-color: rgb(250, 255, 189); /* #FAFFBD; */
    background-image: none;
    color: rgb(0, 0, 0);
    }
    ```

1. 你对`line-height`是如何理解的？

1. 设置元素浮动后，该元素的`display`值是多少？
    自动变成了`display:block`

1. 怎么让Chrome支持小于12px 的文字？
    - 用图片：如果是内容固定不变情况下，使用将小于12px文字内容切出做图片，这样不影响兼容也不影响美观。
    - 使用12px及12px以上字体大小：为了兼容各大主流浏览器，建议设计美工图时候设置大于或等于12px的字体大小，如果是接单的这个时候就需要给客户讲解小于12px浏览器不兼容等事宜。
    - 继续使用小于12px字体大小样式设置：如果不考虑chrome可以不用考虑兼容，同时在设置小于12px对象设置-webkit-text-size-adjust:none，做到最大兼容考虑。
    - 使用12px以上字体：为了兼容、为了代码更简单 从新考虑权重下兼容性。   

1. 让页面里的字体变清晰，变细用CSS怎么做？
    `-webkit-font-smoothing: antialiased;`

1. `font-style`属性可以让它赋值为`oblique`，`oblique`是什么意思？
    倾斜的字体样式

1. `position:fixed`在android下无效怎么处理？
    fixed的元素是相对整个页面固定位置的，你在屏幕上滑动只是在移动这个所谓的viewport，
    原来的网页还好好的在那，fixed的内容也没有变过位置，
    所以说并不是iOS不支持fixed，只是fixed的元素不是相对手机屏幕固定的。

1. 如果需要手动写动画，你认为最小时间间隔是多久，为什么？（阿里）
    多数显示器默认频率是60Hz，即1秒刷新60次，所以理论上最小间隔为`1/60＊1000ms＝16.7ms`

1. `display:inline-block`什么时候会显示间隙？(携程)
    移除空格、使用`margin`负值、使用`font-size:0`、`letter-spacing`、`word-spacing`。

1. `overflow: scroll`时不能平滑滚动的问题怎么处理？

1. 有一个高度自适应的`p`，里面有两个`p`，一个高度`100px`，怎么让另一个填满剩下的高度？

1. png、jpg、gif 这些图片格式解释一下，分别什么时候用。有没有了解过webp？

1. 什么是Cookie 隔离？（或者说：请求资源的时候不要让它带cookie怎么做）
    如果静态文件都放在主域名下，那静态文件请求的时候都带有的cookie的数据提交给server的，非常浪费流量，所以不如隔离开。
    因为cookie有域的限制，因此不能跨域提交请求，故使用非主要域名的时候，请求头中就不会带有cookie数据，这样可以降低请求头的大小，降低请求时间，从而达到降低整体请求延时的目的。
    同时这种方式不会将cookie传入Web Server，也减少了Web Server对cookie的处理分析环节，提高了webserver的http请求的解析速度。

1. style标签写在`body`后与`body`前有什么区别？

1. 什么是CSS预处理器/后处理器？
    - 预处理器例如：LESS、Sass、Stylus，用来预编译Sass或less，增强了css代码的复用性，还有层级、mixin、变量、循环、函数等，具有很方便的UI组件模块化开发能力，极大的提高工作效率。
    - 后处理器例如：PostCSS，通常被视为在完成的样式表中根据CSS规范处理CSS，让其更有效；目前最常做的是给CSS属性添加浏览器私有前缀，实现跨浏览器兼容性的问题。
# 性能优化 ---- CSS

## 提高性能的方法

- 合并 css 文件
  - 减少请求
- 去除无用的 css
- 减少 css 嵌套，最好不超过三层
- 不要在 ID 选择器前进行嵌套
  - ID 唯一且权限值大，嵌套是浪费性能
- 提取公共样式类，把相同样式提取出来作为公共类使用
- 减少通配符 `*` 或者类似 `[hidden=true]` 选择器的使用
- 巧妙运用 css 的继承机制
- 拆分出公共 css 文件，对于比较大的项目可以将大部分页面的公共结构样式提取出来放到单独 css 文件里，这样一次下载 后就放到缓存里，当然这种做法会增加请求，具体做法应以实际情况而定。
- 不用 css 表达式，表达式只是让你的代码显得更加酷炫，但是对性能的浪费可能是超乎你想象的。_（已经弃用了吧）_
- 少用 css rest，可能会觉得重置样式是规范，但是其实其中有很多操作是不必要不友好的，有需求有兴趣，可以选择 `normalize.css`。
- cssSprite，合成所有 icon 图片，用宽高加上 background-position 的背景图方式显现 icon 图，这样很实用，减少了 http 请求。
- 善后工作，css 压缩(在线压缩工具 YUI Compressor)
- GZIP 压缩，是一种流行的文件压缩算法。

## 性能优化

- 避免使用 `@import` ，外部的 css 文件中使用会使页面在加载时增加额外的延迟
  - 使用 `@import` 引用的 css 文件只有在引用它的那个 css 文件被下载，解析之后，浏览器才会知道还有另外一个 css 需要下载，这时才去下载，然后下载后开始解析，构建 render tree 等一系列操作，这就导致浏览器无法并行下载所需的样式文件
  - 多个 `@import` 会导致下载顺序紊乱，**IE**中，`@import`后面的 js 文件优先于 `@import` 下载，并且打乱甚至破坏 `@import` 自身的并行下载
  - 使用 `<link>` 标签加载就好了
- 避免**回流**
  - 会导致**回流**的样式
    1. 大小有关的 width,height,padding,margin,border-width,border,min-height
    2. 布局有关的 display,top,position,float,left,right,bottom
    3. 字体有关的 font-size,text-align,font-weight,font-family,line-height,white-space,vertical-align
    4. 隐藏有关的 overflow,overflow-x,overflow-y
  - 减少**回流**的操作建议
    1. 减少回流的次数----不要一条条的修改 dom 的样式，预先定义好 class，然后修改 dom 的 classname
    2. 减少回流范围----不要修改影响范围较大的 dom
    3. 为动画元素使用绝对定位
    4. 不要 table 布局，因为一个很小的改动会造成整个 table 重新布局
    5. 避免设置大量的 style 属性，通过设置 style 属性改变节点样式的话，每一次设置都会触发一次 reflow，所以最好使用 class 属性
    6. 如果 css 里面有计算表达式，每次都会重新计算一遍，触发一次 reflow
- **重绘**
  - 颜色 color,background
  - 边框样式 border-style,outline-color,outline,outline-style,border-radius,box-shadow,outline-width
  - 背景有关 background,backgound-image,background-position,background-repeat,background-size
- CSS 动画
  - css 动画启用 GPU 加速，应用 GPU 的图形性能对浏览器中的一些图形操作交给 GPU 完成。canvas2D，布局合成，css3 转换，css3d 变换，webGL，视频
  - 2d 加速
  - 3d 加速，`translate3D`/`scaleZ`
  - `will-change`
- 减少使用昂贵的属性
  - box-shadow, border-radius, filter, 透明度, :nth-child 等

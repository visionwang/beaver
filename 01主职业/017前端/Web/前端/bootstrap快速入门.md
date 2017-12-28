BootStrap有两个重点，一个是概念的理解，理解bootstrap是如何通过div来代替过去的table布局的；一个是常用结构的熟悉，做到需要的组件马上就能找到，小修修改改可用就行。其最帅的一点就是：即使是审美能力几乎为空的我也可以整出个美美哒的页面。
![Markdown](http://i.imgur.com/Bj65n7O.jpg)
# 概念
BootStrap是由Twitter的两位员工Mark Otto和Jacob Thornton于2010年8月创建，距今已有7年，不过其仍然是最流行的前端CSS框架。它基于Less前端开发库，提供了常见的CSS和Javascript代码，然开发快速上手。其具有以下特性：完整的基础CSS插件；丰富的预定义样式表；**基于jQuery**的js插件集；非常灵活的响应式栅格系统，而且崇尚移动先行的思想。随着bootstrap的火爆，出现了很多优秀插件，比如Font Awesome字体。
**[Bootstrap官网地址](http://getbootstrap.com/)**，demo和文档非常丰富。
其下载后的源码结构为：
![Markdown](http://i.imgur.com/yrWIDWg.png)
这儿值得的一说的就是fonts中通过字体文件代替了过去的.png，其通过**@font-face语法**，将安全的Web字体实时下载到客户端，便于任意缩放、改变颜色。
Html标准模板如下所示

	<!DOCTYPE html>
	<html>
	<head>
	    <meta charset="utf-8">
	    <meta http-equiv="X-UA-Compatible" content="IE=edge">
	    <meta name="viewport" content="width=device-width, initial-scale=1">
	    <title>bootstrap基础模板</title>
	    <link href="../public/css/bootstrap.css" rel="stylesheet">
	    <link href="../public/css/bootstrap-theme.css" rel="stylesheet">
	</head>
	<body>
	    <header></header>
	    <article></article>
	    <footer></footer>
	    <script type="text/javascript" src="../public/js/jquery-2.1.4.js"></script>
	    <script type="text/javascript" src="../public/js/bootstrap.js"></script>
	</body>
	</html>

可以看到viewport的媒体查询，此外如果需要更多的模板可以访问[getbootstrap](http://getbootstrap.com/getting-started/)下载模板。
### CSS基本回顾
**优先级**：（过去有一些误区）如何确定CSS的优先级，需要引入一个机制，分别用数字(a,b,c,d)表示优先组合，a表示style属性（行内样式），优先级最高，但由于一般使用class样式，该值为0；b是该css选择器上id数量的总和，一般为1个；c是用在该css选择器上的其他属性css选择器和伪类的总和，包括class（.btn）和属性css选择器li[id=red];d计算元素div和伪元素first-child；通用css选择器`*`的0优先级，即最低；如果2个css具有相同优先级，在样式表中后面的起作用。比如下面连个选择器的优先，第一个比第二个高。
leftbar li#first{color: red}: 0,2,0,1 
leftbar li:first-child{color: blue}: 0,1,0,2

**选择器**
属性选择器
- [att=value]	该属性有确定的值
- [att~=value]	该属性值必须是多个空格隔开的值，比如`class="title featured home"`，其中需要有value
- [att|=value]	属性值时value或`value-`
- [att^=value]	属性值以什么开始
- [att$=value]	属性值以什么结束
- [att*=value]	属性值包含特定值

子选择器：用`>`表示，例如`table>thread>tr>th`
兄弟选择器：临近兄弟用`+`,普通兄弟用`~`

**伪类**：bootstrap支持的伪类包括:hover鼠标划过时的状态, `:focus`元素有焦点的状态, `:first-child`, `:last-child`, `:nth-child()`指定元素的一个或多个特定子元素(可以是数字，也可以是`even`,`odd`)。
例如，设置按钮组，除第一个、最后一个按钮和带dropdown-toggle样式的元素外，其他btn样式都不设置圆角

	.btn-group > .btn:not(:first-child):not(:last-child):not(.dropdown-toggle){border-radius:0}

**display属性**:用于定义建立布局时元素生成的框的类型，如果不谨慎使用容易出错
- none	此元素不会显示
- block	此元素显示为块级元素，前后会带换行符
- inline	默认，此元素会被显示为内联元素，没有换行符
- inline-block	行内块元素
- list-item	此元素会以列表显示
- run-in	此元素会根据上下问作为块级元素和内联元素显示
- inherit	规定从父元素集成display属性
- table-xxx	各种table显示

**媒体查询**:响应式设计的核心元素，常用的有min-width,max-width,and，详情可访问[Mediaqueries官方网站](https://www.w3.org/TR/css3-mediaqueries/ "mediaqueries")
### JavaScript语法回顾
**||和&&运算符**：a&&b返回第一个可转化为false的元素值，a||b返回第一个可转换为true的元素值。
**立即调用的函数表达式**：在JS中，function定义后通过加小括号，完成立即调动。(function(){}())
	
	+function($){
		"user strict";
	}(jQuery);//这儿+和;一样，起到分割的作用

**原型**：`Alert.prototype.close=function(e){}`
**jQuery事件绑定**：jQuery使用on和off来绑定和禁用时间，但bootstrap中稍有变化

	$('td').on('click', function(event){});//原有格式
	$('document').on('click.bjock', 'td', function(event){});//这儿增加了参数'td'，而且选择器变成了document，
	//好处是在document上绑定了一个单击事件，利用冒泡机制，在单机的时候检查是否为td元素，如果是才处理
	//而把td作为选择器，一个页面有多少td都会被绑定，性能下降，这三个参数的名字呗称为享元模式(回顾一下)。
**事件命名空间**：可以看到上例中的事件`click.bjork`，`bjork`被称命名空间，当需要触发自己的时间时，命名空间就变得很有用，比如`$('#first').trigger('click.bjork');`，这是触发自己定义的事件，而不影响他人，这在前端复杂化的今天变得非常重要。
**$.data()**:在很多js插件中都是用`$(selector).data()`方法，意思是收集指定元素上所有以`data-`开头的自定义属性，并合并成一个对象字面量。比如`<div id='aaa' data-role='bbb' data-toggle='ccc'></div>`，那么`$(#'aaa').data();`的值为:`var value={role:'bbb', toggle:'ccc'};`，当然也可以选定指定值`$(#'aaa').data('role');`，结果就是`'aaa'`。 
Tip：
BootStrap包含aria-xxx开头的属性，被称为辅助属性，用于支持有阅读障碍的人士。
# 整体结构
首先介绍一下栅格系统的工作原理
- 一行数据必须包含在一个`.container`中，一遍为其赋予合适的对齐方式和内边距padding。
- 使用行在水平方向上创建一组列
- 具体内容放在列中，只有列可以作为行的直接子元素

接下来看一下`.container`样式的源码，可以看出其核心就是`.container`和`@media`的设置


	.container {
		padding-right: 15px;
		padding-left: 15px;
		margin-left: auto;
		margin-right: auto;
	}
	@media ( min-width :768px) {
		.container {
			width: 750px;
		}
	}


栅系统的用法，其实就是列的组合，主要涉及4个特性：**列组合、列偏移、列嵌套、列排序**，首先介绍**列组合**的例子。

	<div class="container">
		<div class="row">
			<div class="col-md-1">.col-md-1</div>
			<div class="col-md-11">.col-md-11</div>
		</div>
	</div>

实际上列组合的实现非常简单，只涉及2个CSS特性，左浮动和宽度百分比，例如

	.col-md-1, ..., col-md-12{float:left;}
	.col-md-12{width:100%;}
	.col-md-1{width:8.33%;}

列偏移的例子，其原理就是margin-left的使用，有多少个1/12的margin-left，就有多少offset。

		<div class="row">
			<div class="col-md-4">.col-md-1</div>
			<div class="col-md-4 col-md-offset-4">.col-md-4 .col-md-offset-4</div>
		</div>

同样，栅格系统支持嵌套，即在列中再声明多个行，内部嵌套的row宽度为100%时，就是当前外部列的宽度。突然有个思路，就是最外围的`.container`是根据@media设置像素的，其中的内容均是使用的相对大小。

		<div class="row">
			<div class="col-md-8">
				<div class='row'>
				<div class="col-md-6">
				<div class="col-md-6">
				</div>
			</div>
		</div>

列排序其实就该改变列的方向，也就是改变左右浮动，并设置浮动的距离，其通过`.col-md-push-*`和`.col-md-pull-*`实现。系统为了方便，统一在左浮动的基础上，通过设置left和right的值来实现定位显示。

	.col-md-pull-6{right:50%;}
	.col-md-push-6{left:50%;}

**响应式栅格**：针对不同的设配使用不同的样式前缀，比如中型的`.col-md-xx`，大型的`.col-lg-xx`，有意思的是，我们可以跨设备的设置样式，例如示例既支持md中型的设备也支持sm小型的设备。
	
		<div class="col-sm-12 col-md-8">.col-sm-12 .col-md-8</div>
清除浮动问题：理想很丰满，现实很骨感，有时会出现不同设备显示时出现意外，比如因为高度的原因造成错位，这时需要使用

		<div class="clearfix visible-xs"></div>
# CSS布局
在BootStrap中，布局部分主要包括基础排版Typography、代码Code、表格Table、表单Forms、按钮Buttons、图片Images等内容。
- 基础排版：在type.less文件中设置，包括标题`h1`；页面主题`<body>`；强调文本`<small>`，`<em>`,`<cite>`;对齐方式,`<p class-"text-left/right/center/justify">`等4种；缩略语`<abbr title="xionger">HTML</abbr>`，有点意思；地址元素`<address>Shanghai</address>`;引用`<blockquote><p>帅的1B</p></blockquote>`；列表，增加了去点列表`<ul class="list-unstyled"></ul>`，内联列表`class='list-inline'`,定义列表`<dl><dt></dt><dd></dd></dl>`,水平定义列表`class="dl-horizontal"`；
- 代码：在code.less文件中设置，`<code>`显示单行内联代码；`<kbd>`显示用户输入代码；`<pre>`元素新生多行代码块。
- 表格：在table.less文件设置，基础样式`<table class="table">`；带背景条纹的表格`class='table table-striped'`；带边框的表格`class='table table-bordered`;带鼠标悬停高亮的`class='table table-hover'`；紧凑型的`class='table table-condensed'`；行级元素样式，即`<tr>`样式，包括`.active`,`.warning`,`.success`,`.danger`,`.info`;响应式表格`class='table table-responsive`。
- 表单：在form.less文件中设置，比较简单，基础的如`<div class='form-group'>`，`<input class='form-control' placeholder='请输入Emal'>`,`class='checkbox'`;内联表单`<form class='form-inline'>`;横向表单`<form class='form-horizontal'>`;横向的表单内元素，如`<label class='checkbox-inline'><input ...></label>`;控件状态，通过属性`focus`,`disabled`;验证状态，`class='form-group has-warning/error/success'`;控件大小，`class='input-lg/sm'`;帮助，`<span class='help-block'>`
- 按钮：在button.less文件配置，样式有`btn btn-default`,`btn btn-primary`,`btn btn-success`,`btn btn-info`,`btn btn-warning`,`btn btn-danger`,`btn btn-link`链接样式；大小`btn-lg/sm/xs`；活动/禁用状态，`active/disabled`。
- 图像：在scaffolding.less中配置，包括`img-rounded`,`img-circle`,`img-thumbnail`。
- 辅助样式：文本样式，柔和灰`text-muted`,主要蓝`text-primary`,成功绿`text-success`,信息栏`text-info`,警告黄`text-warning`,危险红`text-danger`；辅助图标，关闭`class="close"`,向下箭头`class="caret"`；内容浮动，`pull-right/left`,清除浮动`clearfix`;隐藏显示`show/invisible/hidden`

**Tip**:这样大体能够理解less等意思了，就是动态的CSS语言，编译后就是.css文件，其不仅仅是给bootstrap用的，任何的css框架均可使用。此外就是，如果说我们自己写样式是做填空题的话，bootstrap其实就是让我们变成做选择题，帅，boss的感觉有木有。

# 常用组件
在bootstrap中，CSS组件都是通过AO模式进行架构的：Append附加；Override重写。CSS组件包括8种基本类型，应用中对其进行组合即可。
![](http://i.imgur.com/Ow2LC3L.png)
**Bootstrap常用的布局组件的应用**,均可在该知道网页找到，[runboot.com](http://www.runoob.com/bootstrap/bootstrap-tutorial.html)，需要时copy-paste即可，其中还有一个菜鸟工具（最下方，叫做客户化布局），非常方便，常见组件结构如下图所示。
![](http://i.imgur.com/ShUgew1.png)
**Tip**：这部分的目标就是需要什么组件，能查得到即可，对于**分页**等复杂需求，还需要选择合适的bootstrap插件进行增强。

# 常用js插件
在之前CSS的基础上，BootStrap自带12种jQuery插件，其利用bootstrap数据API，大部分插件可以在不编写任何代码的情况下触发。
在bootstrap中，js插件遵循以下3个规则。
- Html布局规则：基于元素自定义属性的布局规则，比如使用类似于data-target的自定义属性
- javascript实现步骤：所有插件都遵循jQuery插件开发的标准步骤，所有的事件保持统一IDE标准
- 插件调用方法：所有插件的使用，都可以是HTML声明式，也可以是js调用式，且支持多种回调和可选参数。

具体示例如下所示，只需在button上添加`data-dismiss='alert'`属性，即可在单机该button时关闭警告框。

    <div class="alert">
        <button type="button" class="close" data-dismiss="alert">&times;</button>
        <strong>警告！</strong>
    </div>
BootStrap中的js都遵循同样的步骤来实现js插件，如下所示
1. 声明立即调用函数，如`+function($){"use strict";}(jQuery);`
2. 定义插件类及相关原型方法，如`Alert.prototype.close`
3. 在jQuery上定义插件并重设插件构造函数，如`$.fn.alert.Constructor=Alert`
4. 防冲突处理，如`$.fn.alert.noConflict`
5. 绑定各种触发事件(data-api)

常见的js插件如下图所示
![](http://i.imgur.com/PaYNcIn.png)

**通用技术**
- 禁用插件默认行为： `$(document).off('.alert.data-api')`
- 可编程性: `$('.btn.btn-danger').button('toggle').addClass('fat');`, `$('#myModel').modal({keyboard: false});`
- 自定义事件: `$('#myModel').on('show.bs.modal', function(e){if(!data) return e.preventDefault();})`

Tip:此外还可以禁止响应式布局，即删除名为viewport的meta元素，并未.container设置一个默认值。

# 补充知识
在实际应用汇总，bootstrap提供的组件基本满足了一般项目的需求，但比如分页控件的支持就显得比较弱，这时需要选用网上现有的插件，也可以自己编写相关扩展。这部分最重要的是思路，**在自定义样式时，为了避免覆盖BootStrap默认的样式或行为，建议通过附加样式的形式来实现**。

	.pagination.square > li > a {
		margin 0 5px;
	}
	<ul class='pagination square'></ul>

**第三方扩展**
- [Font Awesome](http://fontawesome.io/):是一款强大的icon图标集，可以进行矢量缩放，支持任意CSS对大小、颜色、阴影的操作。使用非常简单，只需下载组件包并引用即可，Font Awesome`icon-`与bootstrap`glyphicon-`完全兼容。


	<link rel='stylesheet' href='/css/font-awesome.min.css'>//常规用法
	<p><i class='icon-camera-retro icon-2x'>放大两倍</i></p>

- [Buttons组件](http://www.bootcss.com/p/buttons/)：依赖Font Awesome，其提供非常强大的按钮功能
- [DateTime Picker](http://www.bootcss.com/p/bootstrap-datetimepicker/index.htm)：非常强大的日期插件，功能和Jquery版的类似，但其风格和bootstrap更统一。
- 其他：Cikonss响应式icon；Flat UI扁平化风格；Metrao UI CSS，Win8效果；Messager弹框组件等。

**tip:第一次markdown体验，给自己点个赞。**
**参考资料**
1.徐涛. 深入理解bootstrap[M]. 北京:机械工业出版社, 2015.
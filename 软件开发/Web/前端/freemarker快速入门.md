虽然当前比较推荐使用thymeleaf替代jsp作为java网页开发的模板语言，不过公司推荐使用freemarker，那就顺势而为，速度学一发，然后迅速开始新项目了。
![](http://i.imgur.com/TIhHyrQ.png)
# 简介 #
FreeMarker第一个版本出现在1999年，哇，都18年了，2015年该项目导入到Apache软件基金会，应该还是有一些的自己的特色的，其官方手册还是比较详细的，[freemarker官方文档](http://freemarker.org/docs/index.html)
**常见指令**有很多，具体请见[directive](http://freemarker.org/docs/ref_directive_include.html)，接下来做个最基本的展示。
判断

	<#if student.name == 'xionger'>
	 	xionger is 2B!
	<#elseif student.name == 'xiongda'>
	 	xiongda is genius!
	<#else>
		others are handsome!
	</#if>
列表

	<p>We have these students:
	<table border=1>
	  <#list students as student>
	    <tr><td>${student.name}</td><td>${student.score}</td>
	  </#list>
	</table>
包含

	<#include "/copyright_footer.html">
**常见内置方法**

	student.name?upper_case //xionger->XIONGER
	student.name?length //7
	students?size //2,xionger,xiongda
	//在<#list students as student>中
	student?index //以0开始的索引
	student.graduated?string("Y", "N") 
	students?join(", ") //xionger.xiongda
类似的build-ins还有很多，请见[build-ins](http://freemarker.org/docs/ref_builtins.html)
# 扩展知识 #
**自定义宏**

	<#macro hello name>
		<h1>Hello ${name}!</h1>
	</#macro>
	<@hello name='xionger'/> //使用时直接调用即可
其功能就是把常用的模板做成宏的形式，便于复用。
**格式化输出**

	${'<span>test</span>'}  //输出：&lt;span&gt;test&lt;/span&gt;
	${'<span>test</span>'?no_esc}  //输出: <span>test</span>



# 补充内容 #
- Eclipse插件：Help->Install New Software，输入http://download.jboss.org/jbosstools/updates/development/indigo/，在JBoss Application Development 下找到 FreeMarker IDE，安装后重启Eclipse即可。

Tip:
对这部分的学习要求就是了解就好，不值得花很多的时间，项目中有问题再查阅。

**参考文献**


1. Apache, FreeMarker. Apache FreeMarker Manual[EB/OL]. http://freemarker.org/docs/index.html.
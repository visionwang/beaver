终于进入国内当前最火的前端框架Vue.js的学习了，最近周边的哥们也开始用该框架做线上项目，闲暇之余，做个快速的了解，重在基础部分。
![vue](http://i.imgur.com/XwVAUpE.png)

# 基础概念 #
目前在国内使用vue框架比较出名的团队包括饿了么、滴滴等，总的来说，vue框架目前有一个比较不错的发展，其主要聚焦在视图层，非常轻量、支持数据绑定、指令。安装使用非常简单，即可以使用`<script>`标签应用[js下载地址](http://www.bootcdn.cn/vue/)，也可以使用`npm i vue`安装。常见的参考资料包括[Vue官网](https://cn.vuejs.org/)，[基础demo博文](http://www.cnblogs.com/keepfool/p/5619070.html)等。
- **数据绑定**：包括`{{}}`，`{{ true? 1 : 0}}`,`{{example | toUpperCase}}`过滤器，`<div v-if="show">VUE</div>`指令。
- **指令**：常见指令包括`v-if`, `v-show`, `v-else`,  , `v-text`等同于`{{}}`, `v-html`， `v-ref`, `v-el`, `v-pre`, `v-cloak`。此外还可以添加自定义指令，例如`Vue.directive('custom-direct', {bind:function(){}, update:function(){}})`。
`v-model`创建双向数据绑定，可以添加number, lazy, debounce属性。
`v-for`循环，支持filterBy、orderBy。
`v-bind`用于响应的html特性，将一个或多个attribute动态绑定到表达式，比如给标价添加id, data-xxx。
`v-on`绑定事件监听器，之后可以添加参数，也可以填加修饰符，比如`<button v-on:click='doThis'>或<button @client='doThis'>`。
- **计算属性**：计算属性会在其依赖属性变化时，自动计算更新，与之相关的DOM部分也会更新。需要注意的是，计算属性默认是缓存的，详情参加之后基础示例一节。
- **过滤器**：其本质就是函数，可以在指令中用类似管道的方法处理数据，例如字母操作capitalize&uppercase&lowercase； json过滤器；限制过滤器，用在`v-for`中, limitBy, filterBy, orderBy; currency过滤器；debounce过滤器；自定义过滤器`Vue.filter('test', function(){})`，支持双向过滤，如`{read:function(){}, write:xxx}`。
- **Class与Style的绑定**：`<div id='xxx', class='testA' v-bind:class="{'cus-blue':isOk}">`
- **动画**：`<div v-if="show" transaction="expand">`，之后在CSS文件中添加`.expand-transaction{xxx}, .expend-enter, .expand-leave{xxx}`即可。
- **Method**：可以通过`methods: {methodA:function(){}}`的方式将所有相关方法放在一个ViewModel中，此外可以通过添加$event对象来访问原生事件对象，其常见修饰符包括`.prevent`, `.stop`, `.capture`, `.self`，键盘的事件如`@keyup.enter`。
- **Vue实例属性与方法**：实例属性包括组件树访问，$parent, $root, $children, $refs；DOM访问包括$el, $els；数据访问$data, $options。实例方法包括DOM方法，$appendTo(), $before(), $after(), $remove, $nextTick；Event方法包括$on(), $once(), $emit(), $dispatch(), $broadcase(), $off()等。

# 基础示例 #

	<!DOCTYPE html>
	<html>
	<head>
	    <meta charset="utf-8" />
	    <title>vue demo</title>
	</head>
	<body>
	    <div id="didi-navigator">
	        <ul>
	            <li v-for="tab in tabs">
	                {{tab.text}}
	            </li>
	        </ul>
	        <p>{{message}}</p>
	        <input type="text" v-model="message" />
	    </div>
	    <script src="vue.min.js"></script>
	    <script type="text/javascript">
	    new Vue({
	        el: '#didi-navigator',
	        data: {
	            tabs: [{
	                text: '火车'
	            }, {
	                text: '飞机'
	            }, {
	                text: '汽车'
	            }],
	            student: {
	                name: 'xionger',
	                desc: '多帅哟',
	                age:28,
	                workage:7
	            },
	            message:''
	        },
	        computed:{
	        	cache:false,
	        	totalAge:function(){return this.age+this.workage}
	        }
	    });
	    </script>
	</body>
	</html>

# 进阶概念 #
- **组件**：Vue推荐把组件代码按照template, style, script的方式拆分，放置到对应.vue文件中。组件可以理解为预先定义好行为的ViewModel类，一个组件可以定义很多选项，但最核心的选项包括**模板template**声明了数据和最终展现给用户的DOM之间的映射关系；**初始数据data**；**接受的外部参数props**，默认单项绑定，可以显示声明为双向绑定；**方法methods**操作数据，可以通过`v-on`将用户输入时间和组件方法进行绑定；**生命周期钩子lifecycle hools**，一个组件会触发多个生命周期钩子，比如`create`,`attached`,`destroyed`等，相对于传统的MVC，可以理解为Controller的逻辑被分散到这些钩子函数中。



- **路由与视图**：通过官方插件vue-router支持路由功能，支持开发大型单页应用程序，其支持嵌套路由、组件惰性加载、视图切换动画等功能，通过bower或npm安装，`npm i vue-router -g`。


		 <div id="app">
		        <p>
		            <a v-link="{path:'/foo'}">Go to Foo</a>
		            <a v-link="{path:'/bar'}">Go to Bar</a>
		        </p>
		        <router-view></router-view>
		    </div>
		    <script src="vue.min.js"></script>
		    <script src="vue-router.min.js"></script>
		    <script type="text/javascript">
		   var Vue = require('vue');
		   var VueRouter = require('vue-router');
		   //Vue.use(VueRouter)
		   var Foo = Vue.extend({template:'<p>This is foo!</p>'});
		   var App = Vue.extend({});
		   var router = new VueRouter();
		   router.map({
		   	'/foo': {component: Foo},
		   	'/bar': {component: Bar}
		   });
		   router.start(App, '#app');
		    </script>


- **表单校验**：可以通过`vue-validator`插件对表单进行校验，示例为`<input type='text' v-validate:username-"['required']"/>`，和jQuery等组件的验证组件逻辑一致。常见的验证规则包括`required`,`pattern`,`minlength`,`min`等，此外可以通过validator-errors错处信息输出组件进行输出（包括Component、Partial模板）。


- **与服务端通讯**：通过`vue-resource`插件，Vue.js可以构建一个完全不依赖后端服务的应用，也可以与服务端进行数据交互来通过更新界面，其基于AJAX、JSONP等技术与服务端通信，其实就是对ajax的简化封装了。

- **webpack**:webpack是一个模块化加载器，同时支持AMD、CMD等加载规范，支持通过和异步两种依赖加载方式，安装非常简单`npm i webpack -g`，最简单的示例如下所示(需要注意安装npm安装的module的path环境变量)。


		//1.cat.js
		var cats=['xionger', 'xiongda', 'guangtouqiang'];
		module.exports = cats;
		//2.app.js入口文件
		cats = require('./cats.js');
		console.log(cats);
		//3.打包
		$ webpack ./app.js app.bundle.js
		//4.运行
		$ node app.bundle.js
		//5.常见命令形式
		$ webpack <enrty> <output>
		 -p: 对打包后代码进行压缩
		 --watch:文件变化时，重新打包
		 --config:指定Webpack打包配置文件
		 --progress:在终端显示打包过程
	


- 辅助功能
**vue-cli脚手架工具**可以快速的构建项目，提供了指定项目模板，比如`vue init webpack my-project`直接生成基于webpack构建的项目，非常的赞，之后使用命令`npm install`和`npm run dev`就直接可以在浏览器看到结果了。
**测试开发与调试**常见工具包括Sublime, WebStorm和VSCode，支持视图安装，chrome中还有一个Vue.js devtools可用于调试。
**Scrat与vuejs的组合**可以很好的解决前端工程化的问题（如下图所示），毕竟现在前端的应用场景越来越复杂，比如新闻聚合网站、电商平台、直播互动社区等。
![](http://i.imgur.com/BfKxYc2.png)
Scrat是UC团队在百度FIS基础上二次开发的webapp模块化开发框架，[github地址](http://scrat-team.github.io/#!/index)，有些过时了，但思路确实很棒。
**vue-load**是基于Webpack的loader，在Vue组件化中起到决定性作用;

Tip：
[Vue2.0新手填坑攻略](http://www.jianshu.com/p/5ba253651c3b)
之后需要对ECMA6要有一个相应的了解。
JSONP：&callback=flightHandler"
生命周期钩子：其实就是面向切面的一些暴露出来的filter点，可以织入自身所需的代码。

**参考资料**
1. 张耀春. Vue.js权威指南[M]. 北京:电子工业出版社, 2016.
正好旁边前端的兄弟最近在学习React，为了更深入的了解前端的业态，也果断来学习一发，目标是有个基础的了解，需要时能快速上手就OK，说实话，个人并不是很喜欢它的这种推翻MVC的思路，这个思路和原来的微软的WebForm基本上是一个路子，只是组件的代码可控，可维护。
![](http://i.imgur.com/gC22fUM.png)

# 前端发展 #
ECMA6已发布两年，相关的配套环境已慢慢发展起来(比如Babel可以将最新代码翻译成老版的JS代码提供兼容性)，javascript这门语言也发展的越来越完善，和传统的Java，C#越来越像。
- **ECMA6**
**`const`,`let`**:前者定义常量，后者定义块级作用域。
**lambda表达式**： `let add = (a, b) => {return a+b;}`，可以解决this在对象方法中嵌套函数的问题。
**函数默认参数，参数数组**: `function desc(name='Xionger', age=29, ...args){}。`
**展开操作符**：可以展开数组、对象，`let arr1=[1,2]; let arr2=[...arr1,3]`
**模板字符串**：
		let a =`My name is ${name}!`
**解构赋值**: 
		let foo = ['one', 'two', 'three'];
		let [one, two, three] = foo;
		console.log(`${one}`, `${two}`, `${three}`);
**类**： `class Animal{  constructor(name, age){xxx} }`
**模块化**：最开始随着Require.js的流行，出现AND格式，之后随着Nodejs的出现有了**CommonJS**格式，再之后的browserify,让浏览器端也支持该格式，现在ES6的退出也有了相应的标准`import`, `export`。
**Babel**:可以将ES6代码编译成ES5代码，`npm install babel-cli -g`, `babel es6.js -o compiled.js`
Tip:
此外可以参看[30分钟掌握ES6](http://www.jianshu.com/p/ebfeb687eb70)或者[阮一峰大神](http://www.ruanyifeng.com/)的相关文章。

# React #
**特点**：一切基于组件；JSX，可以将类似HTML的结构编译成Javascript，说实话，个人不是很顶这个思路。**Flux**是React推出的一个前端架构思路，而Redux是对该思路的一个很好的实践。

**Virtual DOM**：在React中，用户不用操作真正的DOM节点，每个React组件都是用VirtualDOM渲染的，它是一种对于HTML DOM节点的抽象描述。它与DOM的一大区别就是它采用了更高效的渲染方式，组件的DOM结构映射奥VirtualDOM上，当需要重新渲染组件时，React在VirtualDOM上实现了一个Diff算法，通过这个算法寻找需要变更的节点，再把里面的修改更新到实际需要修改的DOM节点上，这样就避免了整个渲染DOM带来的巨大成本。简单来讲，就是其通过Diff算法（比较html实际的DOM和javascript代码中的虚拟DOM），将原来的DOM的**全量更新变为了增量更新**。

# webpack #
**特点**
代码拆分(code splitting)方案:`require.ensure(["module-a"], function(require){var a = require("module-a");});`
智能的静态分析:`require("./template/" + name + ".jade");`
模块热切换：`webpack-dev-server --hot`
**安装使用**

	npm install webpack -g
	webpack main.js bundle.js
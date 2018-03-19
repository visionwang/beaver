望借着换工作的东风，好好的俊一波IDEA，之前始终习惯于Eclipse的使用，不过在MAC系统环境，可以明显发现Eclipse无法满足Retina屏幕的要求，此外在idea中可以更容易的整合各种插件。

#使用事项
**界面**
1. IntelliJ IDEA 默认界面是隐藏掉 Toolbar 和 Tool Buttons，在`View菜单`中点选。注意通过`Window->Save Current Layout`避免每次都需要重新设置界面。 
2. 注意整体主题和编辑主题是分开的，代码字体和Console字体都要选择适用中文的
3. IDE编码默认是UTF-8，还需要将Project Encoding设置为UTF-8，Properties中勾选`Transparent native-to-ascii conversion`
4. 如果你的 Tomcat 控制台输出乱码，并且你已经保证了本文上面的控制台字体设置你设置的字体包含中文，那你还可以尝试下在 Tomcat 的 VM 参数上加上：`-Dfile.encoding=UTF-8`
如果你是 Mac 系统，很有可能是需要的，通过`工具集->字体册->添加字体`添加。
![](http://images2017.cnblogs.com/blog/636325/201712/636325-20171227172241394-399864608.png)
5. IntelliJ IDEA 缓存和索引介绍和清理方法， `File->Invalidate Caches / Restart..`
6. 相比于Eclipse的实时自动编译，IDEA更习惯手动编译降低资源消耗。其编译方式包括：`Compile`编译指定类文件，不管是够修购;`Rebuild`编译Project所有文件，效率最低;**`Make`**推荐使用，只编译修改过的文件提高效率，适合大型项目。
7. Maven配置,一图胜千言吧，一定主要注意的是，给maven建立好良好的结构，便于管理使用。对于不同的远程maven仓库，一定要设置做好相应的配置，包括不同的环境。
![](http://images2017.cnblogs.com/blog/636325/201712/636325-20171227180504269-1225323913.png)
8. 注意配置JDK1.7和1.8，包括Project级别和Platform级别，不同的Module可以选用不同的JDK，比如client一般使用1.7便于兼容，其他使用1.8。
9. 配置`Build -> Compiler`，取消自动Build，还可以将Build process heap size增大为1500。（如果编译出现`OutOfMemoryError `）
10. IDEA中没有类似Eclipse工作区的概念，而是通过Project和Module来管理项目代码。
11. 在创建包时，需要去除`齿轮图标中的Compact Empty Middle Packages`，不然空包被隐藏很尴尬，过去深受其困扰。
12. 代码管理工具集成，对GIT的支持很棒，具体的分支方案根据各自团队要求即可。
13. 实时代码模板`Live Template`，和Eclipse有些差别，`sysout->soutp`，可以根据习惯自定义。文件代码模板`file and code template`，预设模板内容很多，需要时再仔细处理即可。此外还有更加方便的`Postfix Completion`来快速完成代码，比如notnull来自动生成判空语句。
14. 插件安装，常见的包括[lombok](http://www.blogjava.net/fancydeepin/archive/2012/07/12/lombok.html)去除冗长代码 , Junit Generator, Alibaba Java Coding Guidelines, [Sonar](https://mp.neixin.cn/cms/content/article/19235902113272Bq4Ma9a5zgMKa1psTo)等。
15. 如果打开maven项目看不到Package包图的情况，直接删除.idea目录后重新打开即可。
16. IDEA目录类型，包括Sources, Tests, Resources, Test Resources和Excluded（排除项目）。
17. 在Web相关项目时，需要注意一个Artifacts概念，Java Web项目必备一个配置就是war包展开的方式，一般选择`war exploded`。
18. Tomcat VM参数设置，`-Xms550m -Xmx1250m -XX:PermSize=550m -XX:MaxPermSize=1250m`
19. 如下图在`Auto import`中设置自动导包和自动去除无用包，之前深受其困扰。
![](http://images2017.cnblogs.com/blog/636325/201712/636325-20171228131038535-384339921.png)
20. 设置方法分割线，在`Editor->Apperance->勾选Show method seperator`。
21. 文件可以通过`localHistory`查找本地更改， 避免信息丢失。
22. 修改``Editor Tabs的show tabs in single row`选项来显示多个tab页面。
23. 注释配置，`CodeStyle->Java->取消勾选Line comment at first column`
24. 设置`System settings->open project in new window`，避免每次都需要选择。
25. 长语句可以通过在行号上右键，选择软分行增强可读性。

[官方文档](http://www.jetbrains.com/idea/webhelp/getting-help.html)

#快捷键
|  Editing（编辑） |
|:--|
|Command + ,  系统首选项|
|Command + Y 删除一行|
|command+alt+B 可以导航到一个抽象方法的实现代码|
|command+N 查找指定类名的类|
|control+N generate生成方法或其他|
|Command + Option + L 格式化代码|
|Command + / 注释/取消注释与行注释|
|Ctrl + Shift + / 注释/取消注释与块注释|
|Command + Option + 左右 前进，后退，跳到之前处理页面|
|Command + F 文件内查找|
|Command + Shift + F 全局查找（根据路径）|
|Command + R 替换|
|Control + Option + O 优化import，alt+enter 导入包，自动修改|
|Command + Shift + U 大小写切换|

|Control + Space 基本的代码补全（补全任何类、方法、变量）|
|Control + Shift + Space 智能代码补全（过滤器方法列表和变量的预期类型）|
|Command + Shift + Enter 自动结束代码，行末自动添加分号|
|Control + O 覆盖方法（重写父类方法）|
|Control + I 实现方法（实现接口中的方法）|
|Command + Option + T 包围代码（使用if..else, try..catch, for, synchronized等包围选中的代码）|



|Option + 方向键上 连续选中代码块|
|Option + Delete 删除到单词的开头|
|Shift + Enter 开始新的一行|

|Command + Delete 删除当前行或选定的块的行|
|**Search/Replace（查询/替换）**|
|Double Shift 查询任何东西|

|**Live Templates（动态代码模板）**|
|Command + Option + J 弹出模板选择窗口，将选定的代码使用动态模板包住|
|Command + J 插入自定义动态代码模板|


https://www.cnblogs.com/pinganjiankang/p/6950684.html

**通过Diagram查看类结构**
在 IntelliJ IDEA 中这个查看一个类也就是当前类的所有继承关系，包括实现的所有的接口和继承的类，
这个继承，不仅仅是一级的继承关系，包括好几层的继承。父类的父类的父类。直到最后。
可以很清楚明了的了解一个类的实现关系。
diagram 英[ˈdaɪəgræm] 美[ˈdaɪəˌɡræm]

[windows版本参考](http://www.cnblogs.com/xiong2ge/p/idea_windows_fast.html)


IntelliJ IDEA for Mac 设置代码提示（Alt+/）或自动补全的快捷键
左键点击屏幕左上角: IntelliJ IDEA
点击选项菜单：Preferences 打开设置对话框
在左侧的导航框中点击: KeyMap
移除原来的Cycle Expand Word 的 Alt+/ 快捷键绑定。 
在 Basic 上点击右键,去除原来的 Ctrl+空格 绑定,然后添加 Alt+/ 快捷键。 

单元测试时可以选择method, class, package, open directory等不同维度

**参考资料**
<mark>推荐极客学院的相关教程</mark>[IntelliJ IDEA使用教程](http://wiki.jikexueyuan.com/project/intellij-idea-tutorial/)
[intellij idea如何学习？](https://www.zhihu.com/question/53659760)
[IntelliJ IDEA使用教程](http://www.phperz.com/special/83.html)
[为何 IntelliJ IDEA 比 Eclipse 更适合于专业java开发者](http://www.cnblogs.com/wangzhongqiu/p/6698880.html)
[IDEA Community(社区版) 使用Maven创建Web工程 并部署tomcat](http://blog.csdn.net/u012364631/article/details/47682011)
[Intellij IDEA 提交代码到远程GitHub仓库]
(https://my.oschina.net/lujianing/blog/180728)

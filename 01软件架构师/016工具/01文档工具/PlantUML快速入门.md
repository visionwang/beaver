在Ctrip时学会使用的PlantUML工具非常的棒，可以通过脚本来生成所需的图表，减少了大量通过拖动调整图形位置的时间，很好的提高了工作效率。
###概述
该工具目前支持顺序图，用例图，类图，活动图，部署图等，基本可以满足绝大部分的需求。不支持ER图和思维导图，其中ER图可以通过类图代替。
[官方网站](http://plantuml.com/)
[官方文档](http://plantuml.com/PlantUML_Language_Reference_Guide.pdf)
**顺序图**

* 常见参与者包括`**actor行动者**，boundary边界，control控制，entity实体，database数据库，collections集合等`。
* 给参与者通过关键字起别名
* 自动添加消息请求的顺序`autonumber`
* 分页`newpage`
* 分组信息`alt/else, opt, loop, par, break, critical, group`
* 添加注释`note left`
* 分割线`== Initialization ==`
* 延时消息`...5 minutes latter...`
* 生命周期的开始与结束`activate XXX  deactivate XXX`
* 标题`title __Simple__ **communication** example`
* 模块`box "Internal Service" #LightBlue`

**用例图**

    @startuml
    left to right direction
    skinparam packageStyle rectangle
    actor customer
    actor clerk
    rectangle checkout {
    customer -- (checkout)
    (checkout) .> (payment) : include
    (help) .> (checkout) : extends
    (checkout) -- clerk
    }
    @enduml

###依赖组件
**Graphviz**
除了顺序图，其他类型的图形生成都需要依赖Graphviz组件，[Graphviz官方网站](http://www.graphviz.org/)，可以通过Homebrew安装，`brew intstall graphviz`。
**homebrew**
MacOS中有一个很棒的包管理工具Homebrew，通过`/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`安装，类似Linux系统下的yum,apt等，[Homebrew](https://brew.sh/)。



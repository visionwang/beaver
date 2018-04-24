安装scala需要如下步骤

下载压缩包并解压
配置环境变量
检验安装结果

http://www.scala-lang.org/download/

在Terminal中，使用
brew install scala

IntelliJ的下载和配置
下载
官网下载地址
如果简单学习，使用社区版即可，如果专业开发，请购买专业版。如果资金充裕，建议购买专业版，不想花钱的话，网上也有很多破解方法，自行查找。
配置
下载完成后，打开IntelliJ，在菜单栏的Preference中，选择Plugins(插件)，然后搜索Scala


IntelliJ的下载和配置
下载
官网下载地址
如果简单学习，使用社区版即可，如果专业开发，请购买专业版。如果资金充裕，建议购买专业版，不想花钱的话，网上也有很多破解方法，自行查找。
配置
下载完成后，打开IntelliJ，在菜单栏的Preference中，选择Plugins(插件)，然后搜索Scala

Hello World!

scala> 1 + 1

res2: Int = 2

scala> 


object LoopTest {
    def main(args: Array[String]) {
      var a = 10;
      // 无限循环
      while( true ){
        println( "a 的值为 : " + a );
      }
  }
}





http://www.runoob.com/scala/scala-regular-expressions.html










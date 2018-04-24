https://www.cnblogs.com/Alandre/p/4456719.html

泥瓦匠初次遇见 navicat 是在 java.lang.Object 源码中的一个hashCode方法：

1
public native int hashCode();
    为什么有个navicat呢？这是我所要学习的地方。所以今天泥瓦匠想要总结下navicat。


凡是一种语言，都希望是纯。比如解决某一个方案都喜欢就单单这个语言来写即可。Java平台有个用户和本地C代码进行互操作的API，称为Java Native Interface (Java本地接口)。
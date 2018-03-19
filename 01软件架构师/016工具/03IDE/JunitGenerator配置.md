https://www.cnblogs.com/wangmingshun/p/6411885.html

二、修改JUnitGenerator V2.0的配置。

　　1、自动生成测试代码和java类在同一包下，不匹配maven项目标准测试目录。

　　　  修改Output Path为：${SOURCEPATH}/../../test/java/${PACKAGE}/${FILENAME}，

　　     Default Template选择JUnit 4。










了解Mysql；
基本操作：sql、数据导出/入、备份/恢复；
掌握常用的mysql存储引擎及索引方式；
会查看sql执行计划，掌握常用sql调优方法；
功能基本原理：复制、文件、日志、事务、锁；

# 基本概念 #



mysql的 insert into ... on duplicate key ... 只能插入不能update的原因是因为：

第一个字段必须是唯一索引或 unique 主键，

第一个字段必须是唯一索引或 unique 主键，

第一个字段必须是唯一索引或 unique 主键，否则执行就就只会insert 而不会执行update。

https://yq.aliyun.com/ziliao/22834


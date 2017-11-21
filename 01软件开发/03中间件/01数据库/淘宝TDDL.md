http://blog.csdn.net/diu_brother/article/details/51554555


淘宝根据自身业务需求研发了TDDL（Taobao Distributed Data Layer）框架，主要用于解决分库分表场景下的访问路由（持久层与数据访问层的配合）以及异构数据库之间的数据同步，它是一个基于集中式配置的JDBC DataSource实现，具有分库分表、Master/Salve、动态数据源配置等功能。就目前而言，许多大厂也在出一些更加优秀和社区支持更广泛的DAL层产品，比如Hibernate Shards、Ibatis-Sharding等。TDDL位于数据库和持久层之间，它直接与数据库建立交道，


百亿级别，目前是亿级


http://blog.csdn.net/moreevan/article/details/11977115/

https://www.cnblogs.com/dreamfree/p/4088431.html



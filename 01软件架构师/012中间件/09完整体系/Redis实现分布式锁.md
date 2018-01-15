http://blog.csdn.net/wgwgnihao/article/details/52098805


1.setnx

设置 key 对应的值为 string 类型的 value。 如果 key 已经存在，返回 0， nx 是 not exist 的意思。



2.setex

设置 key 对应的值为 string 类型的 value，并指定此键值对应的有效期。



//Tip： 2个时间，一个表示操作请求Timeout时间，另一个是这个Key的生存时间。。



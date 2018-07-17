https://segmentfault.com/a/1190000009922279


其实对数组的复制，有四种方法：

for
clone
System.arraycopy
arrays.copyof
本文章主要分析 System.arraycopy() ，带着几个问题去看这个方法：

深复制，还是浅复制
String 的一维数组和二维数组复制是否有区别
线程安全，还是不安全
高效还是低效


https://stackoverflow.com/questions/5924762/prevent-auto-increment-on-mysql-duplicate-insert
实际上是当
createList
updateList放在一起时，batchSave的反向设置SkuID，可能自增字段会设置错

213607
218378
218408
218412

https://blog.csdn.net/demonliuhui/article/details/54572908


几种浅拷贝

1、遍历循环复制

List<Person> destList=new ArrayList<Person>(srcList.size());  
for(Person p : srcList){  
    destList.add(p);  
}  
1
2
3
4
2、使用List实现类的构造方法

List<Person> destList=new ArrayList<Person>(srcList);  
1
3、使用list.addAll()方法

List<Person> destList=new ArrayList<Person>();  
destList.addAll(srcList);  
1
2
4、使用System.arraycopy()方法

Person[] srcPersons=srcList.toArray(new Person[0]);  
Person[] destPersons=new Person[srcPersons.length];  
System.arraycopy(srcPersons, 0, destPersons, 0, srcPersons.length);  






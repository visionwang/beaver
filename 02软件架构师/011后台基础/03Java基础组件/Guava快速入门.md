
http://bastengao.iteye.com/blog/1134887

Files.copy(from,to); //注意，只用了一行代码噢  

base              基本的工具类与接口
io                 io流相关的工具类与方法
net               网络地址相关的工具类与方法
primitives        原始类型的工具类
collect           通用集合接口与实现，与其集合相关工具类
util.concurrent 并发相关工具类

Preconditions.checkArgument(count > 0, "must be positive: %s", count); 


http://blog.csdn.net/ghsau/article/details/27962647

http://blog.csdn.net/L_Sail/article/details/70667015

https://www.cnblogs.com/dooor/p/5285484.html




guava缓存：http://ifeve.com/google-guava/


  SetView<Integer> intersection = Sets.intersection(sets, sets2);  
        for (Integer temp : intersection) {  
            System.out.println(temp);  
        }  
        // 差集  
        System.out.println("差集为：");  
        SetView<Integer> diff = Sets.difference(sets, sets2);  
        for (Integer temp : diff) {  
            System.out.println(temp);  
        }  
        // 并集  
        System.out.println("并集为：");  
        SetView<Integer> union = Sets.union(sets, sets2);  
        for (Integer temp : union) {  
            System.out.println(temp);  
        }  


        
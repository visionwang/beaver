avoid getfield opcode其实是一段注释，看代码时不理解，就查了查在这记录一下。

字面意思是：避免getfield操作。那么首先需要知道什么是getfield操作。查《深入理解java虚拟机》得： 
![](http://img.blog.csdn.net/20160801111626456)

现在有些理解了，String源码中避免的就是“获取指定类的实例域，并将其值压入栈顶”的操作。所以现在写两个代码，看看他们的字节码文件对比一下

    public class Test {

        char[] arr = new char[100];

        public static void main(String[] args) {
            Test t = new Test();
            t.test();
        }

        public void test() {
            System.out.println(arr[0]);
            System.out.println(arr[1]);
            System.out.println(arr[2]);
        }
    }
字节码
![](http://img.blog.csdn.net/20160801112130927)
图中三个框框分别代表三条输出语句，getfield操作也有三次。

下面看一下另一份代码：

public class Test2 {

    char[] arr = new char[100];

    public static void main(String[] args) {
        Test t = new Test();
        t.test();
    }

    public void test() {
        char[] a = arr;
        System.out.println(a[0]);
        System.out.println(a[1]);
        System.out.println(a[2]);
    }
}
![](http://img.blog.csdn.net/20160801112433616)
这次也是三部分，但是注意到**getfield操作只有一次，就是第一次将arr赋值给a的时候**
总结：<remark>在一个方法中需要大量引用实例域变量的时候，使用方法中的局部变量代替引用可以减少getfield操作的次数，提高性能。</remark>




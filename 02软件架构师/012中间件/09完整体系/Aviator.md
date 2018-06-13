Aviator是Zeus所使用的规则执行引擎，用于计算规则的执行结果
1.简介
Aviator是一个高性能、轻量级的java语言实现的表达式求值引擎，主要用于各种表达式的动态求值。现在已经有很多开源可用的java表达式求值引擎，为什么还需要Avaitor呢？
Aviator的设计目标是轻量级和高性能 ，相比于Groovy、JRuby的笨重，Aviator非常小，加上依赖包也才450K,不算依赖包的话只有70K；当然，Aviator的语法是受限的，它不是一门完整的语言，而只是语言的一小部分集合。
其次，Aviator的实现思路与其他轻量级的求值器很不相同，其他求值器一般都是通过解释的方式运行，而Aviator则是直接将表达式编译成Java字节码，交给JVM去执行。简单来说，Aviator的定位是介于Groovy这样的重量级脚本语言和IKExpression这样的轻量级表达式引擎之间。

2.特性
Aviator的特性
支持大部分运算操作符，包括算术操作符、关系运算符、逻辑操作符、位运算符、正则匹配操作符(=~)、三元表达式?: ，并且支持操作符的优先级和括号强制优先级，具体请看后面的操作符列表。
支持大整数和精度运算（2.3.0版本引入）
支持函数调用和自定义函数
内置支持正则表达式匹配，类似Ruby、Perl的匹配语法，并且支持类Ruby的$digit指向匹配分组。
自动类型转换，当执行操作的时候，会自动判断操作数类型并做相应转换，无法转换即抛异常。
支持传入变量，支持类似a.b.c的嵌套变量访问。
函数式风格的seq库，操作集合和数组
性能优秀
Aviator的限制：
没有if else、do while等语句，没有赋值语句，仅支持逻辑表达式、算术表达式、三元表达式和正则匹配。
不支持八进制数字字面量，仅支持十进制和十六进制数字字面量。




3.使用手册
3.1 执行表达式
最简单的例子，执行一个计算1+2+3的表达式：
"1+2+3"
细心的朋友肯定注意到结果是Long，而不是Integer。这是因为Aviator的数值类型仅支持Long和Double，任何整数都将转换成Long，任何浮点数都将转换为Double（这个规则在2.3.0引入big int和decimal类型后有所改变，具体见大数计算一节），包括用户传入的变量数值。这个例子的打印结果将是正确答案是Long类型的6。



3.2 String计算
Aviator的String是任何用单引号或者双引号括起来的字符序列，String可以比较大小（基于unicode顺序），可以参与正则匹配，可以与任何对象相加，任何对象与String相加结果为String。String中也可以有转义字符，如\n、\\、\'等。
" 'a\"b' ";   //字符串 a"b
" \"a\'b\" ";  //字符串 a'b
" 'hello '+3 ";  //字符串 hello 3
" 'hello '+ unknow ";  //字符串 hello null

3.3 调用函数
Aviator支持函数调用，函数调用的风格类似lua，下面的例子获取字符串的长度：
"string.length('hello')";
string.length('hello')是一个函数调用，string.length是一个函数,'hello'是调用的参数。
再用string.substring来截取字符串：
"string.contains(\"test\",string.substring('hello',1,2))"
通过string.substring('hello',1,2)获取字符串'e'，然后通过函数string.contains判断e是否在'test'中。可以看到，函数可以嵌套调用。
Aviator的内置函数列表请看后面


3.4 三元操作符
Aviator不提供if else语句，但是提供了三元操作符?:用于条件判断
"a>0? 'yes':'no'"
这个例子用来判断用户传入的数字是否是正整数，是的话打印yes。
Aviator的三元表达式对于两个分支的结果类型并不要求一致，可以是任何类型。
3.5 正则表达式匹配
Aviator支持类Ruby和Perl风格的表达式匹配运算，通过=~操作符，如下面这个例子匹配email并提取用户名返回：
"email=~/([\\w0-8]+)@\\w+[\\.\\w+]+/ ? $1:'unknow'"
email与正则表达式//([\\w0-8]+@\\w+[\\.\\w+]+)/通过=~操作符来匹配，结果为一个Boolean类型，因此可以用于三元表达式判断，匹配成功的时候返回$1，指代正则表达式的分组1，也就是用户名，否则返回unknown。这个例子将打印killme2008这个用户名。
Aviator在表达式级别支持正则表达式，通过//括起来的字符序列构成一个正则表达式，正则表达式可以用于匹配（作为=~的右操作数)、比较大小，匹配仅能与字符串进行匹配。匹配成功后，Aviator会自动将匹配成功的分组放入$num的变量中，其中$0指代整个匹配的字符串，而$1表示第一个分组，以此类推。
Aviator的正则表达式规则跟Java完全一样，因为内部其实就是使用java.util.regex.Pattern做编译的。


3.6 nil对象
nil是Aviator内置的常量，类似java中的null，表示空的值。nil跟null不同的在于，在java中null只能使用在==、!=的比较运算符，而nil还可以使用>、>=、<、<=等比较运算符。Aviator规定，任何对象都比nil大除了nil本身。用户传入的变量如果为null，将自动以nil替代。
"nil == nil";  //true
" 3> nil";    //true
" true!= nil";    //true
" ' '>nil ";  //true
" a==nil ";   //true,a is null
nil与String相加的时候，跟java一样显示为null
3.7 大数计算和精度
从2.3.0版本开始，aviator开始支持大数字计算和特定精度的计算，本质上就是支持java.math.BigInteger和java.math.BigDecimal两种类型，这两种类型在aviator中简称为big int和decimal类型。
类似“99999999999999999999999999999999”这样的数字在Java语言里是没办法编译通过的，因为它超过了Long类型的范围，只能用BigInteger来封装。但是aviator通过包装，可以直接支持这种大整数的计算，例如：
"99999999999999999999999999999999+99999999999999999999999999999999"
结果为类型big int的：
199999999999999999999999999999998

3.8 类型转换和提升
当big int或者decimal和其他类型的数字做运算的时候，按照 long < big int < decimal < double的规则做提升，也就是说运算的数字如果类型不一致，结果的类型为两者之间更“高”的类型。
例如:
1 + 3N， 结果为big int的4N
1 + 3.1M，结果为decimal的4.1M
1N + 3.1M，结果为decimal的4.1M
1.0 + 3N，结果为double的4.0
1.0 + 3.1M，结果为double的4.1
decimal的计算精度
Java的java.math.BigDecimal通过java.math.MathContext支持特定精度的计算，任何涉及到金额的计算都应该使用decimal类型。
默认Aviator的计算精度为MathContext.DECIMAL128，你可以自定义精度，通过:
 AviatorEvaluator.setMathContext(MathContext.DECIMAL64);
即可设置，更多关于decimal的精度问题请看java.math.BigDecimal的javadoc文档。



4.6 内置函数
函数名称	说明
sysdate()	返回当前日期对象java.util.Date
rand()	返回一个介于0-1的随机数，double类型
print([out],obj)	打印对象，如果指定out，向out打印，否则输出到控制台
println([out],obj)	与print类似，但是在输出后换行
now()	返回System.currentTimeMillis
long(v)	将值的类型转为long
double(v)	将值的类型转为double
str(v)	将值的类型转为string
date_to_string(date,format)	将Date对象转化化特定格式的字符串,2.1.1新增
string_to_date(source,format)	将特定格式的字符串转化为Date对象,2.1.1新增
string.contains(s1,s2)	判断s1是否包含s2，返回Boolean
string.length(s)	求字符串长度,返回Long
string.startsWith(s1,s2)	s1是否以s2开始，返回Boolean
string.endsWith(s1,s2)	s1是否以s2结尾,返回Boolean
string.substring(s,begin[,end])	截取字符串s，从begin到end，end如果忽略的话，将从begin到结尾，与java.util.String.substring一样。
string.indexOf(s1,s2)	java中的s1.indexOf(s2)，求s2在s1中的起始索引位置，如果不存在为-1
string.split(target,regex,[limit])	Java里的String.split方法一致,2.1.1新增函数
string.join(seq,seperator)	将集合seq里的元素以seperator为间隔连接起来形成字符串,2.1.1新增函数
string.replace_first(s,regex,replacement)	Java里的String.replaceFirst 方法，2.1.1新增
string.replace_all(s,regex,replacement)	Java里的String.replaceAll方法 ，2.1.1新增
math.abs(d)	求d的绝对值
math.sqrt(d)	求d的平方根
math.pow(d1,d2)	求d1的d2次方
math.log(d)	求d的自然对数
math.log10(d)	求d以10为底的对数
math.sin(d)	正弦函数
math.cos(d)	余弦函数
math.tan(d)	正切函数
map(seq,fun)	将函数fun作用到集合seq每个元素上，返回新元素组成的集合
filter(seq,predicate)	将谓词predicate作用在集合的每个元素上，返回谓词为true的元素组成的集合
count(seq)	返回集合大小
include(seq,element)	判断element是否在集合seq中，返回boolean值
sort(seq)	排序集合，仅对数组和List有效，返回排序后的新集合
reduce(seq,fun,init)	fun接收两个参数，第一个是集合元素，第二个是累积的函数，本函数用于将fun作用在集合每个元素和初始值上面，返回最终的init值
seq.eq(value)	返回一个谓词，用来判断传入的参数是否跟value相等,用于filter函数，如filter(seq,seq.eq(3)) 过滤返回等于3的元素组成的集合
seq.neq(value)	与seq.eq类似，返回判断不等于的谓词
seq.gt(value)	返回判断大于value的谓词
seq.ge(value)	返回判断大于等于value的谓词
seq.lt(value)	返回判断小于value的谓词
seq.le(value)	返回判断小于等于value的谓词
seq.nil()	返回判断是否为nil的谓词
seq.exists()	返回判断不为nil的谓词

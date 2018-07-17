一般我们使用BigDecimal进行比较精密的计算，我这里计算金额。注意使用double构造器的本质与String构造的本质，避免造成问题。

    new BigDecimal(interestAmt).setScale(2, RoundingMode.UP)

基本确定BigDecimal的double构造存在问题，String构造不存在问题。

**BigDecimal**
public BigDecimal(double val)
将 double 转换为 BigDecimal，后者是 double 的二进制浮点值准确的十进制表示形式。返回的 BigDecimal 的标度是使 (10scale × val) 为整数的最小值。
注：

此构造方法的结果有一定的不可预知性。有人可能认为在 Java 中写入 new BigDecimal(0.1) 所创建的 BigDecimal 正好等于 0.1（非标度值 1，其标度为 1），但是它实际上等于 0.1000000000000000055511151231257827021181583404541015625。**这是因为 0.1 无法准确地表示为 double（或者说对于该情况，不能表示为任何有限长度的二进制小数）。这样，传入 到构造方法的值不会正好等于 0.1（虽然表面上等于该值）。**
另一方面，String 构造方法是完全可预知的：写入 new BigDecimal("0.1") 将创建一个 BigDecimal，它正好 等于预期的 0.1。因此，比较而言，通常建议优先使用 String 构造方法。
当 double 必须用作 BigDecimal 的源时，请注意，此构造方法提供了一个准确转换；它不提供与以下操作相同的结果：先使用 Double.toString(double) 方法，然后使用 BigDecimal(String) 构造方法，将 double 转换为 String。要获取该结果，请使用 static valueOf(double) 方法。
参数：
val - 要转换为 BigDecimal 的 double 值。
抛出：
NumberFormatException - 如果 val 为无穷大或 NaN。

**valueOf**
public static BigDecimal valueOf(double val)
使用 Double.toString(double) 方法提供的 double 规范的字符串表示形式将 double 转换为 BigDecimal。
注：这通常是将 double（或 float）转化为 BigDecimal 的首选方法，因为返回的值等于从构造 BigDecimal（使用 Double.toString(double) 得到的结果）得到的值。

参数：
val - 要转换为 BigDecimal 的 double。
返回：
其值等于或约等于 val 值的 BigDecimal。
抛出：
NumberFormatException - 如果 val 为无穷大或 NaN。
从以下版本开始：
1.5

<remark>这就是2进制计算机的缺点</remark>

那如果参数为float类型的，比如BigDecimal.valueOf(0.99f)的值是多少呢？
首先此函数会自动进行精度扩展，将float类型的0.99转成double类型的，因为0.99本身就是无法用二进制表示的，也就说无论你的精度是多少位，都无法用二进制来精确表示0.99，或者你用二乘来判断（0.99*2=1.98 0.98*2=1.960.96*2=1.92 。。。永远无法得到一个整数）。这就是二进制计算机的缺点，就如同十进制也也无法表示1/3，1/6一样。
所以在0.99f转成double时，进行了精度扩展，变成了0.9900000095367432，而接着转成字符串，最后转成BigDecimal.

参考资料
![jdk手册](http://tool.oschina.net/apidocs/apidoc?api=jdk-zh)
![在线工具](http://tool.oschina.net/apidocs)





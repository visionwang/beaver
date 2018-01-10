[深入理解Java Stream流水线](https://www.cnblogs.com/CarpenterLee/archive/2017/03/28/6637118.html)

http://ifeve.com/stream/


上述代码求出以字母A开头的字符串的最大长度，一种直白的方式是为每一次函数调用都执一次迭代，这样做能够实现功能，但效率上肯定是无法接受的。类库的实现着使用流水线（Pipeline）的方式巧妙的避免了多次迭代，其基本思想是在一次迭代中尽可能多的执行用户指定的操作。为讲解方便我们汇总了Stream的所有操作。

Stream操作分类
中间操作(Intermediate operations)	无状态(Stateless)	unordered() filter() map() mapToInt() mapToLong() mapToDouble() flatMap() flatMapToInt() flatMapToLong() flatMapToDouble() peek()
有状态(Stateful)	distinct() sorted() sorted() limit() skip()
结束操作(Terminal operations)	非短路操作	forEach() forEachOrdered() toArray() reduce() collect() max() min() count()
短路操作(short-circuiting)	anyMatch() allMatch() noneMatch() findFirst() findAny()
Stream上的所有操作分为两类：中间操作和结束操作，中间操作只是一种标记，只有结束操作才会触发实际计算。中间操作又可以分为无状态的(Stateless)和有状态的(Stateful)，无状态中间操作是指元素的处理不受前面元素的影响，而有状态的中间操作必须等到所有元素处理之后才知道最终结果，比如排序是有状态操作，在读取所有元素之前并不能确定排序结果；结束操作又可以分为短路操作和非短路操作，短路操作是指不用处理全部元素就可以返回结果，比如找到第一个满足条件的元素。之所以要进行如此精细的划分，是因为底层对每一种情况的处理方式不同。


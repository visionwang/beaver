


[Java序列化框架性能比较](http://blog.csdn.net/smallnest/article/details/38847653)

[FastJson的使用方法总结](http://www.cnblogs.com/DreamDrive/p/5778959.html)

[json、javaBean、xml互转的几种工具介绍](http://blog.csdn.net/sdyy321/article/details/7024236/)


[FasterXML: POJO serialization to XML. JacksonXml Annotations not working](https://stackoverflow.com/questions/32384377/fasterxml-pojo-serialization-to-xml-jacksonxml-annotations-not-working)


[xmlmapper和xpath的使用](http://blog.csdn.net/lanwenbing/article/details/19112115)


XmlMapper 

Home » com.fasterxml.jackson.core » jackson-core


<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.9.0</version>
</dependency>

		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-core</artifactId>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
		</dependency>


目前来看还是dom4j比较给力




补充：XML基础知识

XML基本语法：标签语法，字符、命名；文档部分；元素，起始标签、结束标签、空元素标签、文档元素、元素嵌套、字符串；字符数据；属性，空白、行尾的处理；注释；CDATA部分，<![DATA[…]]>；格式正规的文档。
XML命名空间：声明，修饰名，作用范围。
DTD：过时的XML文档约束方案，现在对采用XML Schema和xslt。

XML Schema的文档结构
<?xml version=”1.0”?>
<Schema xmlns=”urn:schemas-microsoft-com:xml-data” xmlns:dt=”urn:schemas-microsoft-com:datatypes”>
</Schema>
元素的定义：<ElementType name=”date” dt:type=”date”>，出现次数maxOccurs=”*”

可扩展样式语言XSL,XSLT文档，文档结构
<xsl:stylesheet version=”1.0” xmlns:xsl=”http://www.w3.org/1999/XSL/Transform”>
</xsl:stylesheet>

XPath, XLink, XPointer使用的场景并不多，一笔带过。


	private List<OrderItemDetailDto> parseOrderItemDetailXml(String orderItemDetail, int orderType) {
		List<OrderItemDetailDto> result = Lists.newArrayList();

		try {
			SAXReader saxReader = new SAXReader();
			Document document = saxReader.read(new ByteArrayInputStream(orderItemDetail.getBytes("UTF-8")));
			// 3.1获取根元素
			Element rootElement = document.getRootElement();
			List<Element> itemElements = rootElement.elements();
			for (Element iElement : itemElements) {
				OrderItemDetailDto temp = new OrderItemDetailDto();
				temp.setAmount(BigDecimal.valueOf(Double.valueOf(iElement.attributeValue("Amount"))));
				temp.setCategoryID(Integer.valueOf(iElement.attributeValue("CategoryID")));
				temp.setNeedActive(parseNeedActive(iElement.attributeValue("NeedActive"), orderType));
				temp.setOrderItemID(Long.valueOf(iElement.attributeValue("OrderItemID")));
				temp.settCount(Integer.valueOf(iElement.attributeValue("TCount")));
				temp.setTcType(Integer.valueOf(iElement.attributeValue("TCType")));
				result.add(temp);
			}
			return result;
		} catch (Exception ex) {
			logger.warn(ex.getMessage(), ex);
		}
	}
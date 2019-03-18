---
title: Java中的XML之四种方式生成XML文档：DOM，SAX，JDOM，DOM4J
date: 2018-07-30 12:17:44
categories: "JavaSE"
tags:
    - JavaSE
    - XML
copyright:
---
现有以下XML文档books.xml,下面的示例生成此文档部分内容
```XML
<?xml version="1.0" encoding="UTF-8"?>
<bookstore>
	<book id="1">
		<name>冰与火之歌</name>
		<author>乔治马丁</author>
		<year>2014</year>
		<price>89</price>
	</book>
	<book id="2">
		<name>安徒生童话</name>
		<year>2004</year>
		<price>77</price>
		<language>English</language>
	</book>
</bookstore>
```
### DOM
1、创建DocumentBuilder对象
```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
DocumentBuilder db = dbf.newDocumentBuilder();
```
2、添加节点
```java
document.setXmlStandalone(true);
Element bookstore = document.createElement("bookStore");
//向bookstore根节点中添加子节点book
Element book = document.createElement("book");
Element name = document.createElement("name");
name.setTextContent("???");

book.appendChild(name);
book.setAttribute("id", "1");
//将book节点添加到bookstore根节点中
bookstore.appendChild(book);
//将bookstore节点（已经包含了book）添加到dom树中
document.appendChild(bookstore);
```
3、生成xml文件
```java
TransformerFactory tff = TransformerFactory.newInstance();
Transformer tf = tff.newTransformer();
//设置文件
tf.setOutputProperty(OutputKeys.INDENT, "yes");
tf.transform(new DOMSource(document),new StreamResult(new File("books1.xml")));
```
----
### SAX
1、创建一个TransformerFactory类的对象
```java
SAXTransformerFactory tff = (SAXTransformerFactory) SAXTransformerFactory.newsInstance();
```
2、通过SAXTransformerFactory对象创建一个TransformerHandler对象
```java
TransformerHandler handler = tff.newTransformerHandler();
```
3、通过handler对象创建一个Transformer对象
```java
Transformer tr = handler.getTransformer();
```
4、通过Transformer对象对生成的xml文件进行设置
```
// 设置xml的编码
tr.setOutputProperty(OutputKeys.ENCODING, "UTF-8");
// 设置xml的“是否换行”
tr.setOutputProperty(OutputKeys.INDENT, "yes");
5、创建一个Result对象
```java
File f = new File("newbooks.xml");
if (!f.exists()) {
    f.createNewFile();
}
```
6、创建Result对象，并且使其与handler关联
```java
Result result = new StreamResult(new FileOutputStream(f));
handler.setResult(result);
```
7、利用handler对象进行xml文件内容的编写O
```java
// 打开document
handler.startDocument();
```
8、添加节点属性和节点值
```java
AttributesImpl attr = new AttributesImpl();
handler.startElement("", "", "bookstore", attr);
for (Book book : bookList) {
    attr.clear();
    attr.addAttribute("", "", "id", "", book.getId());
    handler.startElement("", "", "book", attr);
    // 创建name节点
    if (book.getName() != null && !book.getName().trim().equals("")) {
        attr.clear();
        handler.startElement("", "", "name", attr);
        handler.characters(book.getName().toCharArray(), 0, book
                .getName().length());
        handler.endElement("", "", "name");
    }
    handler.endElement("", "", "book");
}
handler.endElement("", "", "bookstore");
// 关闭document
handler.endDocument();
```
----
### JDOM
1.生成一个根节点
```java
Element rss = new Element("rss");
```
2.为节点添加属性
```java
rss.setAttribute("version", "2.0");
```
3.生成一个document对象
```java
Document document = new Document(rss);
Element channel = new Element("channel");
rss.addContent(channel);
Element title = new Element("title");
title.setText("<![CDATA[上海移动互联网产业促进中心正式揭牌 ]]>");
channel.addContent(title);
//设置文件编码和换行
Format format = Format.getCompactFormat();
format.setIndent("");
format.setEncoding("GBK");
```
4.创建XMLOutputter的对象
```java
XMLOutputter outputer = new XMLOutputter(format);
```
5.利用outputer将document对象转换成xml文档
```java
outputer.output(document, new FileOutputStream(new File("rssnews.xml")));
```
----
### DOM4J

>使用DOM4J生成RSS文件

1.创建document对象，代表整个xml文档
```java
Document document = DocumentHelper.createDocument();
```
2.创建根节点rss
```java
Element rss = document.addElement("rss");
```
3.向rss节点中添加version属性
```java
rss.addAttribute("version", "2.0");
```
4.生成子节点及节点内容
```java
Element channel = rss.addElement("channel");
Element title = channel.addElement("title");
title.setText("<![CDATA[上海移动互联网产业促进中心正式揭牌 ]]>");
```
5.设置生成xml的格式
```java
OutputFormat format = OutputFormat.createPrettyPrint();
format.setEncoding("GBK");
```
6.生成xml文件
```java
File file = new File("rssnews.xml");
XMLWriter writer;
writer = new XMLWriter(new FileOutputStream(file), format);
//设置是否转义，默认值是true，代表转义
writer.setEscapeText(false);
writer.write(document);
writer.close();
```

> 参阅：
  [慕课网：Java眼中的XML---文件读取](https://www.imooc.com/learn/171)
  [java核心技术 卷II：高级特性](http://product.dangdang.com/25171892.html)
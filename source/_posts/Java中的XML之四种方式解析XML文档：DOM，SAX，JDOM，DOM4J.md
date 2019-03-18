---
title: Java中的XML之四种方式解析XML文档：DOM，SAX，JDOM，DOM4J
date: 2018-07-30 12:17:35
categories: "JavaSE"
tags:
    - JavaSE
    - XML
copyright:
---

现有以下XML文档books.xml,下面的解析示例解析此文档部分内容
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
### DOM解析
1、创建一个DocumentBuilder的对象
```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
DocumentBuilder db = dbf.newDocumentBuilder();
```
2、调用parser方法加载books.xml文件到当前项目下
```java
Document document = db.parse("books.xml");
```
3、进行解析
```java
//获取所有book节点的集合
NodeList bookList = document.getElementsByTagName("book");
//?????book??
for (int i = 0; i < bookList.getLength(); i++) {
    //通过 item(i)方法 获取一个book节点，nodelist的索引值从0开始
    Node book = bookList.item(i);

    //获取book节点的所有属性集合
    NamedNodeMap attrs = book.getAttributes();

    //前提：已经知道book节点有且只能有1个id属性
    //将book节点进行强制类型转换，转换成Element类型
    Element book = (Element) bookList.item(i);

    //通过getAttribute("id")方法获取属性值
    String attrValue = book.getAttribute("id");
    
    //解析book节点的子节点
    NodeList childNodes = book.getChildNodes();
}
```
> 更多解析方法查看API

----
### SAX解析
1、通过SAXParserFactory的静态newsInstance()方法获取SAXParserFactory实例
```java
SAXParserFactory factory = SAXParserFactory.newInstance();
```
2、通过SAXParserFactory实例的newSAXParser()方法返回SAXParser实例
```java
SAXParser parser = factory.newSAXParser();
```
3、创建一个类继承DefaultHandler，重写其中的一些方法进行处理，并创建这个类的实例handler
```java
SAXParserHandler handler = new SAXParserHandler();
parser.parse("books.xml", handler);
```
handler类
```java
//标识解析开始
@Override
public void startDocument() throws SAXException {
    // TODO Auto-generated method stub
    super.startDocument();
    System.out.println("SAX解析开始");
}

//标识解析结束
@Override
public void endDocument() throws SAXException {
    // TODO Auto-generated method stub
    super.endDocument();
    System.out.println("SAX解析结束");
}

//解析xml元素
@Override
public void startElement(String uri, String localName, String qName,
        Attributes attributes) throws SAXException {
    //调用DefaultHandler类的startElement方法
    super.startElement(uri, localName, qName, attributes);
    if (qName.equals("book")) {
        //已知book元素下属性的名称，根据属性名称获取属性值
        String value = attributes.getValue("id");
        System.out.println("book的属性值是：" + value);

    }else if (!qName.equals("name") && !qName.equals("bookstore")) {
        System.out.print("节点名是：" + qName + "---");
    }
}

@Override
public void endElement(String uri, String localName, String qName)
        throws SAXException {
    //调用DefaultHandler类的endElement方法
    super.endElement(uri, localName, qName);
    //判断是否针对一本书已经遍历结束
    if (qName.equals("book")) {
        System.out.println("结束遍历某一本书的内容");
    }
}

@Override
public void characters(char[] ch, int start, int length)
        throws SAXException {
    // TODO Auto-generated method stub
    super.characters(ch, start, length);
    value = new String(ch, start, length);
    if (!value.trim().equals("")) {
        System.out.println("节点值是：" + value);
    }
}
```
----
### JDOM解析
1、创建一个SAXBuilder的对象
```
SAXBuilder saxBuilder = new SAXBuilder();
```
2、创建一个输入流，将xml文件加载到输入流中
```
in = new FileInputStream("books.xml");
```
3、通过saxBuilder的build方法，将输入流加载到saxBuilder中
```
Document document = saxBuilder.build(in);
```
4、通过document对象获取xml文件的根节点
```
Element rootElement = document.getRootElement();
```
5、获取根节点下的子节点的List集合
```
List<Element> bookList = rootElement.getChildren();
for (Element book : bookList) {
    // 解析book的属性集合
    List<Attribute> attrList = book.getAttributes();
    //知道节点下属性名称时，获取节点值
    book.getAttributeValue("id");

    // 对book节点的子节点的节点名以及节点值的遍历
    List<Element> bookChilds = book.getChildren();
    for (Element child : bookChilds) {
        System.out.println("????" + child.getName() + "----????"
                + child.getValue());
    }
}

```
> 如果需要设置编码可以按照以下方式设置

```java
InputStreamReader isr = new InputStreamReader(in, "UTF-8");
Document document = saxBuilder.build(isr);
```
----
### DOM4J解析
1、通过reader对象的read方法加载books.xml文件,获取docuemnt对象
```java
SAXReader reader = new SAXReader();
Document document = reader.read(new File("src/res/books.xml"));
```
2、通过document对象获取根节点
```java
Element bookStore = document.getRootElement();
```
3、通过element对象的elementIterator方法获取迭代器
```java
Iterator it = bookStore.elementIterator();
```
4、遍历迭代器，获取根节点中的信息
```java
while (it.hasNext()) {
    Element book = (Element) it.next();
    // 获取book的属性名以及 属性值
    List<Attribute> bookAttrs = book.attributes();
    for (Attribute attr : bookAttrs) {
        System.out.println("????" + attr.getName() + "--????"
                + attr.getValue());
    }
    //遍历子节点
    Iterator itt = book.elementIterator();
    while (itt.hasNext()) {
        Element bookChild = (Element) itt.next();
        System.out.println("????" + bookChild.getName() + "--????" + bookChild.getStringValue());
    }
}

```
----
### 四种解析方式的区别
- DOM:将文件全部加载到内存中，形成树结构，适用于小文件，需要频繁修改时
    >优点：直观好理解，代码易编写；解析过程树结构保留在内存中，方便修改
    缺点：当xml文件较大时，对内存消耗比较大，容易影响解析性能并造成内存溢出
- SAX:采用事件驱动模式，适用于只需要处理xml中的数据时，不需要频繁修改时
    >优点：内存消耗小
    缺点：不易编码；很难同时访问同一个xml中的多处不同数据
- JDOM：API大量使用了Collections类
- DOM4J：JDOM的一种智能分支，合并了许多超出基本xml文档表示的功能，性能优异，灵活性好，功能强大，十分易用

> 参阅：
  [慕课网：Java眼中的XML---文件读取](https://www.imooc.com/learn/171)
  [java核心技术 卷II：高级特性](http://product.dangdang.com/25171892.html)
---
title: java中的XML
date: 2018-07-20 11:00:21
categories: "javase笔记" 
tags:
    - javase
---
#### HTML与XML的区别
- HTML对大小写不敏感，XML大小写敏感
- HTML结束标签可以省略，如`</p>`，XML不能
- XML只有单个标签而没有结束标签的元素必须以`/`结束
- XML属性值必须用引号括起来
- HTML属性可以没有值，XML所有属性必须有值

#### 解析XML文档
现有以下xml文档books.xml
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
- DOM解析
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

- SAX解析
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
    > handler类
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
- JDOM??
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
- DOM4J解析
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
- 四种解析方式的区别
    - DOM:将文件全部加载到内存中，形成树结构，适用于小文件，需要频繁修改时
        >优点：直观好理解，代码易编写；解析过程树结构保留在内存中，方便修改
        缺点：当xml文件较大时，对内存消耗比较大，容易影响解析性能并造成内存溢出
    - SAX:采用事件驱动模式，适用于只需要处理xml中的数据时，不需要频繁修改时
        >优点：内存消耗小
        缺点：不易编码；很难同时访问同一个xml中的多处不同数据
    - JDOM：API大量使用了Collections类
    - DOM4J：JDOM的一种智能分支，合并了许多超出基本xml文档表示的功能，性能优异，灵活性好，功能强大，十分易用
----    
#### 生成XML文档
- DOM
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
- SAX
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
- JDOM
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

- DOM4J
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

----
#### 验证XML文档
> 需要指定文档结构时，可以提供一个文档类型定义（DTD）或XML Schema
- 文档类型定义
    - 将这些规则纳入XML文档
        ```XML
        <?xml version="1.0"?>
        <!DOCTYPE configuration[

            <!ELEMENT configuration...>
            ...
        ]>
        <configuration>
        ...
        </configuration>
        ```
        >这些规则被纳入到DOCTYPE声明中，代码块[...]用来限定其接线，比如configuration
    - SYSTEM声明，将DTD存储在外面
        ```XML
        <!DOCTYPE configuration SYSTEM "config.dtd">
        <!DOCTYPE configuration SYSTEM "http://myserver.com/config.dtd">
        ```
    - 标记PUBLIC标识符
        ```java
        class MyEntityResolver implements EntityResolver{
            public InputSource resolveEntity(String publicId, String systemID){
                if(publicID.equals(a knowx ID)){
                    return new InputSource(DTD data):
                }else{
                    return null;
                }
            }
        }
        ```
    >PS：具体的类型规则看文档，注意在设计DTD时应该要么只包含文本，要么包含其他元素，避免解析混合式（标签和文本的混合）内容时空
- XML Schema
    - 声明Schema文件
    ```XML
    <?xml version="1.0"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    </project>
    ```
    - 使用xsd:表示XSL Schema定义的命名空间
    <xsd:element name="name" type="xsd:string"/>
    - ref属性引用Schema中位于别处的定义
    <xsd:element ref="name"/>
    <xsd:element name="style" type=StyleType"/>
        <xsd:restriction base="xsd:string"/>
            <xsd:enumeration value="PLAIN"/>
        </xsd:restriction>
    </xsd:element>
#### 使用XPath定位信息
- 查找下列的username的值，,通过XPath表达式/configuration/database/username
    ```
    <configfuration>
        <database>
            <username id="test">pinnuli</username>
            <password>123456</password>
        </database>
    </configfuration>
    ```
- 用XPathFactory创建一个XPath对象，调用evaluate方法计算XPath表达式
    ```
    XPathFactory xpFactory = XPathFactory.newInstance();
    path = xpfactory.newXPath();
    String username = path.evaluate(/configuration/database/username",doc);
    ```
>具体的语法看文档
----
#### XML的命名空间
- 使用xmlns给定命名空间
    ```XML
    <element xmlns="namespaceURI1">
        <child xmlns="namespaceURI2">
            grandchildren
        </child>
    </element>
    ```
    >这里第一个子元素和孙元素都是第二个命名空间的一部分
- 使用xmlns:prefix="namespaceURI"定义命名空间和前缀
    ``` 
    <xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">
        <xsd:element name="pinnuli" type="haha"/>
        ...
    </xsd:schema>
    ```
    >在这里xsd:schema实际上指的是命名空间`http://www.w3.org/2001/XMLSchema`中的`schema`
- 可以控制解析器对命名空间的处理，默认情况下DOM解析器并非“命名空间感知的”，可以调用DocumentBuilderFactory类的setNamespaceAware()打开命名空间处理特性
    ```
    factory.serNamespaceAware(true);
    ```
----

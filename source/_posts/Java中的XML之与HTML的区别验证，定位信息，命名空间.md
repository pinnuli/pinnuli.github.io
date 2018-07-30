---
title: Java中的XML之与HTML的区别验证，定位信息，命名空间
date: 2018-07-30 12:19:45
categories: "javase笔记"
tags:
    - JavaSE
    - XML
copyright:
---
### HTML与XML的区别
- HTML对大小写不敏感，XML大小写敏感
- HTML结束标签可以省略，如`</p>`，XML不能
- XML只有单个标签而没有结束标签的元素必须以`/`结束
- XML属性值必须用引号括起来
- HTML属性可以没有值，XML所有属性必须有值    

----
### 验证XML文档
需要指定文档结构时，可以提供一个文档类型定义（DTD）或XML Schema
#### 文档类型定义
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

>这些规则被纳入到DOCTYPE声明中，代码块[...]用来限定其界限，比如configuration

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

#### XML Schema
- 声明Schema文件
```XML
<?xml version="1.0"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
</project>
```
- 使用xsd:表示XSL Schema定义的命名空间
```XML
<xsd:element name="name" type="xsd:string"/>
- ref属性引用Schema中位于别处的定义
<xsd:element ref="name"/>
<xsd:element name="style" type=StyleType"/>
    <xsd:restriction base="xsd:string"/>
        <xsd:enumeration value="PLAIN"/>
    </xsd:restriction>
</xsd:element>
```
----
### 使用XPath定位信息
- 查找下列的username的值，,通过XPath表达式/configuration/database/username
```XML
<configfuration>
    <database>
        <username id="test">pinnuli</username>
        <password>123456</password>
    </database>
</configfuration>
```
- 用XPathFactory创建一个XPath对象，调用evaluate方法计算XPath表达式
```XML
XPathFactory xpFactory = XPathFactory.newInstance();
path = xpfactory.newXPath();
String username = path.evaluate(/configuration/database/username",doc);
```
>具体的语法看文档

----
### XML的命名空间
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

> 参阅：
  [慕课网：Java眼中的XML---文件读取](https://www.imooc.com/learn/171)
  [java核心技术 卷II：高级特性](http://product.dangdang.com/25171892.html)
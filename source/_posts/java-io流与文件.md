---
title: java io流与文件
date: 2018-07-19 06:27:55
categories: "JavaSE" 
tags:
	- JavaSE
---
### 一、流 
**读写字节**
`InputStream.read和OutpueStream.write`
**组合流过滤器**

- 某些流（如`FileInputStream`或者`FileOutputStream`）只能支持在字节级别上的读写，没有读入数据类型的方法，而其他的流（DataInputStream）这些类就之只能读入数值类型，无法从文件中获取数据，因而对二者进行组合。如：	

``` java
FileInputStream fin = new FileInputStream("test.txt");
DataInputStream din = new DataInputStream(fin);
double s = din.readDouble();
```

- 需要使用缓冲，可以使用一下构造器：

```java
DataInputStream din = new DataInputStream(
	new BUfferInputStream(
		new FileInputStream(test.txt")));
```

- 需要浏览下一个字节以确定是否是想要的值时，可以：

```java
PushbackInputStream pbin = new PushbackInputStream(
	new BUfferInputStream(
		new FileInputStream(test.txt")));
//预读写一个字节
int b = pbin.read();
//不是所期望时将其推回流中
if(b != '<') pbin.unead(b);
```
----
### 二、文本输入与输出
**输出：`PrintWrite`**
`PrintWrite out = new PrintWrite("test.txt");`等同于`PrintWrite out = new PrintWrite(new FileWrite("test.txt"));`
**输入：`Scanner`**
**文本格式存储对象**
用自己的格式依次存储各个字段，以特定字符分隔，如：`PINUULI|201625010417|1997|guangdong`

----

### 三、读写二进制数据
**读**：实现`DataInput`接口，如`DataInputStream，readInt，readBoolean`等方法
**写**：实现`DataOutpu`t接口，如`DataOutputStream，writeInt，writeBoolean`等方法
**随机访问文件**：`RandomAccessFile`
可以在文件中的任何位置查找或者写入数据：
```java
RandomAccessFile in = new RandomAccessFile("test.txt","r");
RandomAccessFile inOut = new RandomAccessFile("test.txt","rw");
```
----
### 四、ZIP文件
每个zip文档都有一个头，包含注入给个文件名字和使用的压缩方法等信息。
**读**：`ZipInputStream`
- 用`getNextEntry`方法返回文档中这些项（文件）的`ZipEntry`对象
- ` ZipInputStream`的`read`方法被修改为碰到当前项的结尾时返回-1，而不是整个zip文件的结尾，读完一个项之后，用`closeEntry`读入下一项
- 在读入单个zip项后，不要关闭zip输入流，否则就不能再读入后续的项
- 通读zip文件：

``` java
ZipInputStream zin = new ZipInputStream(new FileInputStream("test.zip"));
ZipEntry entry;
Scanner in = new Scanner(zin);
while((entry = zin.getNextEntry()) != null){
	while(in.hasNextLine()){
		System.out.println(in.nextLine());
	}
	zin.closeEntry();
}
```

**写**：`ZipOutputStream`
- 对于想要放入到zip文件中的每一个项，都应该创建一个`ZipEntry`对象，并将文件名传递给ZipEntry的构造器
- 调用`ZipOutputStream`的`putNextEntry`方法开始写出新文件，并将数据发送到zip流中
- 完成时调用`closeEntry`方法,如：

```java
FileOutputStream fout = new FilePutputStream("test.zip");
ZipOutputStream zout = new ZipOutputStream(fout);

// 写一个文件
ZipEntry ze = new ZipEntry("filename");
zout.putNextENtry(ze);
send data to zout;
zout.closeEntry();

zout.close();
```
----
### 五、对象流与序列化
- 序列化:`ObjectOutputStream.writeObject()`,
- 反序列化：`ObjectInputream.readObject()`，
- 都需要实现`Serializable`接口
- 只有读写对象时才能用writeObject/readObject方法，对于基本类型，使用writeInt/readInt等
- 序列化算法：
	- 对于遇到的每个对象都关联一个序列号
	- 对于每个对象，第一次遇到时，保存其对象数据到流中
	- 如果某个对象之前已经被保存过，那么只写出“与之前保存过的序列号为x的对象相同”，在反序列化时整个过程相反
	- 对于流中的对象，在第一次遇到其序列号时，构建它并使用流中的数据初始化，然后记录这个序列号与新对象之间的关联
	- 当遇到“与之前保存过的序列号为x的对象相同”标记时，获取与这个序列号相关联的对象引用，即相同对象的重复出现被存储为对这个对象的序列号的引用
- 修改默认的序列化机制

> 一些数据域是不可序列化，或者没必要序列化的，比如只对本地方法有意义的窗口句柄的整数值，重新加载或者传送到其他机器上都没有用，那么就可以将他们标记成是transient，这些域在序列化时就会被跳过。
可以把一些域存储为你想要的格式，想要为默认的读写行文添加验证时。

当你只是想跳过一些域，或者想将这些域保存为你想要的格式，而大部分域依然按照默认的格式保存时，可以仍然实现`Serializable`接口，将那些数据域标记成transient，读写时调用默认的读写方法之后，再做自己想要的处理，如：
		
```java
public class LabeledPoint implements Serializable{
	private String label;

	//对于类LabeledPoint，point不能序列化，那么标志成transient，序列化时就会被跳过,之后存储点的坐标
	private transient Point2D.Double point;
	···
	//重写读写方法
	private void writeObject(ObjectOutputStream out) throws Exception{
		out.defaultWriteObjecy();
		out.writeDouble(point.getX());
		out.writeDouble(point.getY());
	}

	private void readObject(ObjectInputStream in) throws IOException{
		in.defaultReadObject();
		double x = in.readDouble();
		double y = in.readDouble();
		point = new Point2D.DOuble(x,y);
	}
}
```
		
当你只需要保存一部分域时，使用transient关键字就有点麻烦，那么可以通过实现`Externalizable`接口，指定要保存的域

```java
public class Student implements Externalizable{
	private String name;
	private String stuId;
	private int age;
	···
	//重写读写方法
	private void writeExternal(ObjectOutput out) throws IOException{
		out.writeUTF(name);
		out.writeInt(age);
	}

	private void readExternal(ObjectInput in) throws IOException{
		name = in.readUTF():
		age = in.readInt();
	}
}
```
	> PS:readObjecty和writeObject方法时私有的，只有被序列化机制调用，在流中只记录该对象所属的类，而readExternal/writeExternal方法时公共的，而且对包括超类数据在内的整个对象的存储和回复负责。

----
### 六、操作文件
**Path**
- 静态的Paths.get方法接收一个或多个字符串，并将它们用默认文件系统的路径分隔符（类Unix文件系统是/，Windows是\）连接起来，返回一个Path对象，详情见API。如

```java
Path absolutye = Paths.get("/home"."cay");
Path relative = Paths.get("myprog","conf","user.properties");
```
**读写文件**：Files类可以使得普通文件操作变得快捷
- 读取文件所有内容：`byte[] bytes = Files.readAllBytes(path);`,之后可以将其当做字符串`String content = new String(bytes,charset);`
- 向指定文件追加内容：`Files.write(path,content.getBytes(charset),StandardOpenOption.APPEND);`


> 参阅：
  [慕课网：文件传输基础——Java IO流](https://www.imooc.com/learn/123)
  [java核心技术 卷II：高级特性](http://product.dangdang.com/25171892.html)

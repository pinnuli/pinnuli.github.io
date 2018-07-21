---
title: Java Socket应用¨
date: 2018-07-21 15:09:14
tags:
---
####一、 Socket使用时应当注意的一些问题
- 设置超时，从套接字读取信息时，在有数据可供访问之前，读操作会被阻塞，如果此时主机不可达，那么程序将会等待很长时间，并因为系统操作系统的限制最终导致超时
    - 调用setSoTimeout方法设置
    ```JAVA
    Socket s = new Socket(...);
    s.setSoTimeout(10000);
    ```
    - 对构造器Socket(String host,int port)，可以先构建一个无连接的套接字，再使用超时
    ```java
    Socket s = new Socket();
    s.connect(new InetSocketAddress(host,port),timeout);
    ````
- 可中断套接字，用SocketChannel类
- 需要解析因特网地址时，可以用InetAddress类
- 为多个客户端服务时，可以用多线程解决
- 半关闭：套接字连接的一段UN可以终止其输出，同时仍可以接受来自另一端的数据，反过来也一样，调用Socket.shutdownInput/Socket.shutdownOutput
####二、获取Web数
- URI和URL
    - URL是URI的一个特例，URI是个纯粹的语法结构，包含用来点位Web资源的字符串和各种组成功哪部分，URL包含了用于定位Web资源的足够信息，其他无法定位任何数据的URI，称之为URN
    - 一个URI具有一下语法：`[scema:]schemaSpecficPart[#fragment]`
    > I.包含schema:部分的URI成为绝对URI，否则为相对URI
    II.绝对URI的schemaSpecficPart不是以`/`揩油，则称为不透明的，如:`mialto:pinnuli!hostname.com`
    III.所有绝对的透明URI和所有相对URI都是分层的，如：`http://hostname.com/index.html`，`../../java/net/Socket.html#Socket()`
    IV.一个分层URI的URI的schemaSpecficPart具有一下结构：[//authority][path][?query],基于服务器的URI，authority具有一下形式:[user-info@]host[:port]
    - java中URI类的作用
        - 解析表示福并将它分解成各种不同组成成分
        - 标识符的相对化和解析相对标识符
- 使用URLCollection


----
#### 二、基于TCP的SOcket通信
1.创建ServerSocket和Socket
2.打开连接到Socket的输入/输出流
3.按照协议对Socket进行读/写操作
4.关闭输入/输出流，关闭Socket
##### 服务端（多线程响应多个客户端）

    ```java
    //创建一个服务器端Socket，即ServerSocket，指定绑定的端口，并负责监听此端口
    ServerSocket serverSocket=new ServerSocket(8888);
    Socket socket=null;
    System.out.println("***服务器即将启动，等待客户端的连接***");
    while(true){
        //调用accept()方法开始监听，等待客户端的连接
        socket=serverSocket.accept();
        //创建一个新的线程
        ServerThread serverThread=new ServerThread(socket);
        //启动线程
        serverThread.start();
    }
    ```

> ServerThread类

    ```java
    public class ServerThread extends Thread {
	Socket socket = null;
	public ServerThread(Socket socket) {
		this.socket = socket;
	}
	
	//线程执行的操作，响应客户端的请求
	public void run(){
		InputStream is=null;
		InputStreamReader isr=null;
		BufferedReader br=null;
		OutputStream os=null;
		PrintWriter pw=null;
		try {
			//获取输入流，并读取客户端信息
			is = socket.getInputStream();
			isr = new InputStreamReader(is);
			br = new BufferedReader(isr);
			String info=null;
			while((info=br.readLine())!=null){//循环读取客户端的信息
				System.out.println("我是服务器，客户端说："+info);
			}
			socket.shutdownInput();//关闭输入流，半关闭
			//获取输出流，响应客户端的请求
			os = socket.getOutputStream();
			pw = new PrintWriter(os);
			pw.write("欢迎您！");
			pw.flush();//调用flush()方法将缓冲输出
		} catch (IOException e) {
			e.printStackTrace();
		} finally{
			//关闭资源
			try {
				if(pw!=null)
					pw.close();
				if(os!=null)
					os.close();
				if(br!=null)
					br.close();
				if(isr!=null)
					isr.close();
				if(is!=null)
					is.close();
				if(socket!=null)
					socket.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
}

    ```

#####客户端

    ```java
    //1.创建客户端Socket，指定服务器地址和端口
    Socket socket=new Socket("localhost", 8888);
    //2.获取输出流，向服务器端发送信息
    OutputStream os=socket.getOutputStream();//字节输出流
    PrintWriter pw=new PrintWriter(os);//将输出流包装为打印流
    pw.write("用户名：alice;密码：789");
    pw.flush();
    socket.shutdownOutput();//关闭输出流
    //3.获取输入流，并读取服务器端的响应信息
    InputStream is=socket.getInputStream();
    BufferedReader br=new BufferedReader(new InputStreamReader(is));
    String info=null;
    while((info=br.readLine())!=null){
        System.out.println("我是客户端，服务器说："+info);
    }
    //4.关闭资源
    br.close();
    is.close();
    pw.close();
    os.close();
    socket.close();
    ```

----
#### 二、基于UDP的SOcket通信
1.定义发送信息
2.创建DatagramPacket，包含将要发送的信息
3.创建DatagramSocket
4.发送数据
##### 服务端：
- 接收客户端发送的数据

    ```java
    //1.创建服务器端DatagramSocket，指定端口
    DatagramSocket socket=new DatagramSocket(8800);
    //2.创建数据报，用于接收客户端发送的数据
    byte[] data =new byte[1024];//创建字节数组，指定接收的数据包的大小
    DatagramPacket packet=new DatagramPacket(data, data.length);
    //3.接收客户端发送的数据
    socket.receive(packet);//此方法在接收到数据报之前会一直阻塞
    //4.读取数据
    String info=new String(data, 0, packet.getLength());
    System.out.println("我是服务器，客户端说："+info);
    ```
- 向客户端响应数据

    ```java
    //1.定义客户端的地址、端口号、数据
    InetAddress address=packet.getAddress();
    int port=packet.getPort();
    byte[] data2="欢迎您!".getBytes();
    //2.创建数据报，包含响应的数据信息
    DatagramPacket packet2=new DatagramPacket(data2, data2.length, address, port);
    //3.响应客户端
    socket.send(packet2);
    //4.关闭资源
    socket.close();
    ```
##### 客户端：
- 向服务器端发送数据
    ```java
    //1.定义服务器的地址、端口号、数据
    InetAddress address=InetAddress.getByName("localhost");
    int port=8800;
    byte[] data="用户名：admin;密码：123".getBytes();
    //2.创建数据报，包含发送的数据信息
    DatagramPacket packet=new DatagramPacket(data, data.length, address, port);
    //3.创建DatagramSocket对象
    DatagramSocket socket=new DatagramSocket();
    //4.向服务器端发送数据报
    socket.send(packet);
    ```
- 接收服务器端响应的数据
    ```java
    //1.创建数据报，用于接收服务器端响应的数据
    byte[] data2=new byte[1024];
    DatagramPacket packet2=new DatagramPacket(data2, data2.length);
    //2.接收服务器响应的数据
    socket.receive(packet2);
    //3.读取数据
    String reply=new String(data2, 0, packet2.getLength());
    System.out.println("我是客户端，服务器说："+reply);
    //4.关闭资源
    socket.close();
    ```
    > 当Socket关闭时，输入输出流也就关闭了

> 参阅[Java Socket应用---通信是这样练成的](https://www.cnblogs.com/tocy/p/markdown-footnote_link.html)
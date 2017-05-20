---
layout: post
title:  "异步非阻塞通信"
date:   2017-05-18 18:00:05
categories: IoT
excerpt: IoT Netty 开发实践笔记。
---

* content
{:toc}

在JAVA I/O编程过程中，我们会遇到各种IO相关的概念,通过学习UNIX五种I/O模型可以帮助我们理解JAVA I/O模型。
同步与异步的主要区别就在于：会不会导致请求进程（或线程）阻塞。
 
## UNIX五种I/O模型  
1.阻塞I/O模型：

![io_model_1](http://itliang.github.io/blog/public/img/IoT/block_io.png)

2.非阻塞I/O模型：

![io_model_1](http://itliang.github.io/blog/public/img/IoT/nblock_io.png)

3.I/O复用模型：

![io_model_1](http://itliang.github.io/blog/public/img/IoT/select_io.png)

4.信号驱动I/O模型：

![io_model_1](http://itliang.github.io/blog/public/img/IoT/signal_io.png)

5.异步I/O模型： 

![io_model_2](http://itliang.github.io/blog/public/img/IoT/a_io.png)

---

JDK1.4版本提供了新的NIO类库，可以支持非阻塞I/O。在JDK1.7发布的版本中，对NIO类库进行了升级，提供了AIO功能。我从最简单的JAVA BIO编程模型开始讲起：

## JAVA BIO编程

下面是伪异步I/O编程的示例。

### 伪异步I/O编程

在基于传统同步阻塞模型开发中，ServerSocket 负责绑定IP 地址，启动监听端口；Socket 负责发起连接操作。连接成功之后，双方通过输入和输出流进行同步阻塞式通信。当客户端并发访问量增加后，服务端的线程个数和客户端并发访问数呈1:1的关系，随着并发访问量的增大，系统会发生线程堆栈溢出，创建新线程失败，最终导致进程宕机或者僵死。

下面介绍伪异步阻塞I/O，后端通过一个线程池来处理多个客户端的请求，线程池可以灵活地调配线程资源，设置线程的最大值，防止海量并发接入导致线程耗尽。

Client示例代码：
		
    	try{
		Socket socket = new Socket("127.0.0.1", port);
		BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
		PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
		out.println("QUERY SERVER");
		String resp = in.readLine();
		System.out.println("Server response is : "+resp);			
	}catch(Exception e){
	......
	}

Server示例代码:  
线程池的定义：
	
	public class ServerHandlerExecutePool {
		public ExecutorService executor;
		public ServerHandlerExecutePool(int maxPoolSize, int queueSize){
			executor = new ThreadPoolExecutor(Runtime.getRuntime().availableProcessors(), maxPoolSize, 	120L, TimeUnit.SECONDS, new ArrayBlockingQueue<java.lang.Runnable>(queueSize));
		}
	
		public void execute(java.lang.Runnable task){
			executor.execute(task);
		}
	}
	
将请求Socket封装成一个Task：  

	public class MyServerHandler implements Runnable{
		private Socket socket;
		public MyServerHandler(Socket socket){
			this.socket = socket;
		}
	
		@Override
		public void run() {
			// TODO Auto-generated method stub
			BufferedReader in = null;
			PrintWriter out = null;
			try{
				in = new BufferedReader(new InputStreamReader(this.socket.getInputStream()));
				out = new PrintWriter(this.socket.getOutputStream(), true);
				String currentTime = null;
				String body = null;
				while(true){
					body = in.readLine();
					if(body == null){
						break;
					}
					System.out.println("The time server receive order : "+body);
					currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body) ? new java.util.Date(
							System.currentTimeMillis()).toString() : "BAD ORDER" ;
					out.println(currentTime);
				}
			}catch(Exception e){
				......
			}
		}
	}

创建一个服务器处理类的线程池,然后调用线程池的execute方法执行：  

	ServerSocket server = null;
	try{
		server = new ServerSocket(port);
		Socket socket = null;
		ServerHandlerExecutePool singleExecutor = new ServerHandlerExecutePool(50, 10000);
		while(true){
			socket = server.accept();
			singleExecutor.execute(new MyServerHandler(socket));
		}			
		
	} finally {
		if(server != null){
			System.out.println("Time server will close");
			server.close();
			server = null;
		}
	}

---

## JAVA NIO编程
JAVA在NIO类库中加入Buffer对象，所有数据都是用Buffer处理的。  

Channel是一个通道，可以通过它读取和写入数据。和流的不同之处在于Channel是双向的，而流是单向的。与Socket类和ServerSocket类相对应，NIO也提供了SocketChannel和ServerSocketChannel两种不同的套接字通道实现。这两种新增的通道都支持阻塞和非阻塞两种模式。数据可以从Channel读取到Buffer中，也可以从Buffer写到Channel中。

多路复用器Selector，会不断地轮询注册在其上的Channel,如果某个Channel上面有新的TCP连接接入，读和写事件，Channel就会处于就绪状态，被Selector轮询出来，然后通过SelectionKey可以获取就绪Channel的集合。进行后续的I/O操作。一个Selector可以同时轮询多个Channel，只需要一个线程负责Selector的轮询，就可以接入成千上万的客户端。
  
Client代码示例：  
1.客户端主函数：
  
	new Thread(new NioClientHandler("127.0.0.1", 8080), "Nio-Client-1").start();

2.创建线程来处理异步连接和读写操作：

	public class NioClientHandler implements Runnable {

		private Selector selector;
		private SocketChannel socketChannel;
		
		private String host;
		private int port;
		
		private boolean stop;
		
		public NioClientHandler(String host, int port){
			this.host = host;
			this.port = port;
			this.stop = false;
			try{
				selector = Selector.open();
				socketChannel = SocketChannel.open();
				socketChannel.configureBlocking(false);
			}catch (IOException e){
				e.printStackTrace();
				System.exit(1);
			}
		}
	
	
		@Override
		public void run() {
			// TODO Auto-generated method stub
			try{
				boolean result = socketChannel.connect(new InetSocketAddress(host, port));
				if(result){
					System.out.println("Connection Synchronous");
					socketChannel.register(selector, SelectionKey.OP_READ);
					sendToServer(socketChannel, "Hello Server!");
				}else{
					System.out.println("Connection Asynchronous");
					socketChannel.register(selector, SelectionKey.OP_CONNECT);
				}
			}catch(IOException e){
				e.printStackTrace();
			}
			while(!stop){
				try{
					selector.select(1000);
					Set<SelectionKey> selectedKeys = selector.selectedKeys();
					Iterator<SelectionKey> it = selectedKeys.iterator();
					SelectionKey key = null;
					while(it.hasNext()){
						key = it.next();
						it.remove();
						try{
							receiveFromServer(key);						
						}catch(IOException e){
							......
						}
					}
				}catch(Exception e){
					e.printStackTrace();
					System.exit(1);
				}
			}
			if(selector != null){
				try{
					selector.close();
				}catch(IOException e){
					e.printStackTrace();
				}
			}
		}
		
		private void sendToServer(SocketChannel sc, String content) throws IOException{
			byte []Bytes = content.getBytes();
			ByteBuffer writeBuffer = ByteBuffer.allocate(Bytes.length);
			writeBuffer.put(Bytes);
			writeBuffer.flip();
			sc.write(writeBuffer);
			if(!writeBuffer.hasRemaining()){
				System.out.println("Send to server again");
			}
		}
	
		private void receiveFromServer(SelectionKey key) throws IOException{
			SocketChannel sc = (SocketChannel)key.channel();
			if(key.isConnectable()){
				if(sc.finishConnect()){
					sc.register(selector, SelectionKey.OP_READ);
					sendToServer(sc, "What will server send?");
				}
			}else if(key.isReadable()){
				ByteBuffer readBuffer = ByteBuffer.allocate(1024);
				int readBytes = sc.read(readBuffer);
				if(readBytes > 0){
					readBuffer.flip();
					byte[] bytes = new byte[readBuffer.remaining()];
					readBuffer.get(bytes);
					String body = new String(bytes, "UTF-8");
					System.out.println("Client received :"+body);
					this.stop = true;
				}else if(readBytes < 0){
					System.out.println("Connection lost");
					key.cancel();
					sc.close();
				}
			}
		}
	}
	
	
Server代码示例：  
1.启动服务器：

	MultiplexerServer server = new MultiplexerServer(8080);
	new Thread(server, "NIO-MultiplexerServer-1" ).start();

2.多路复用类，轮询多路复用器Selector,处理多个客户端的并发接入：


	public class MultiplexerServer implements Runnable {
		private ServerSocketChannel serverChannel;
		private Selector selector;
		
		private volatile boolean stop;
		
		public MultiplexerServer(int port){
			try{
				selector = Selector.open();
				serverChannel = ServerSocketChannel.open();
				serverChannel.configureBlocking(false);
				serverChannel.bind(new InetSocketAddress(port), 1024);
				serverChannel.register(selector, SelectionKey.OP_ACCEPT);
			}catch(IOException e){
				......
			}
		}
				
		@Override
		public void run() {
			// TODO Auto-generated method stub
			while(!stop){
				try{				
					selector.select(1000); //Wake up in a second
					Set<SelectionKey> selecteKeys = selector.selectedKeys();
					Iterator<SelectionKey> it = selecteKeys.iterator();
					SelectionKey key = null;
					while(it.hasNext()){
						key = it.next();
						it.remove();
						try{
							System.out.println("Client Request");
							handleClientRequest(key);						
						}catch(Exception e){
							......
						}
					}
				}catch(Throwable t){
					......
				}			
			}
			if(selector != null){
				try{
					selector.close();
				}catch(IOException e){
					e.printStackTrace();
				}
			}
		}
		
		private void handleClientRequest(SelectionKey key) throws IOException{
			if(key.isValid()){
				if(key.isAcceptable()){
					//Accept new connection
					ServerSocketChannel ssc = (ServerSocketChannel)key.channel();
					SocketChannel sc = ssc.accept();
					sc.configureBlocking(false);
					sc.register(selector, SelectionKey.OP_READ);
					System.out.println("Accept connection request");
				}else if(key.isReadable()){
					//Read data
					System.out.println("Read data");
					SocketChannel sc = (SocketChannel)key.channel();
					ByteBuffer readBuffer = ByteBuffer.allocate(1024);
					int readBytes = sc.read(readBuffer); //None block, Check with return value
					if(readBytes > 0){
						readBuffer.flip(); //limit == position, position = 0;
						byte []bytes = new byte[readBuffer.remaining()];
						readBuffer.get(bytes);
						String body = new String(bytes, "UTF-8");
						System.out.println("Server received:"+body);						
						String currentTime = new java.util.Date(System.currentTimeMillis()).toString();
						sendToClient(sc, currentTime);					
					}else if (readBytes < 0){
						//Linkage lost, release resource
						key.cancel();
						sc.close();
					}else if (readBytes == 0){
						//Normal case, ignore
					}
				}
			}
		}
		
		private void sendToClient(SocketChannel channel, String content) throws IOException{
			if(content !=null && content.trim().length() > 0){
				byte []bytes = content.getBytes();
				ByteBuffer replyBuffer = ByteBuffer.allocate(bytes.length);
				replyBuffer.put(bytes);
				replyBuffer.flip();
				channel.write(replyBuffer);
			}
		}
	}

---

## JAVA AIO编程

NIO2.0引入了新的异步通道概念，并提供了异步文件通道和异步套接字通道的实现。它是真正的异步非阻塞I/O,对应UNIX网络编程中的事件驱动I/O（AIO）。

Client代码示例：  
1.客户端主函数：  
  
	AsyncClientHandler client = new AsyncClientHandler("127.0.0.1", 8080);
	new Thread(client, "AIO-AyncClientHandler-1").start();

2.独立的I/O线程创建异步服务器客户端handler:  

	public class AsyncClientHandler implements CompletionHandler<Void, AsyncClientHandler>, Runnable {
		private AsynchronousSocketChannel client;
		private CountDownLatch latch;
		private int port;
		private String host;
		public AsyncClientHandler(String host, int port) throws IOException{
			this.port = port;
			this.host = host;
			client = AsynchronousSocketChannel.open();
		}
		
		@Override
		public void run() {
			// TODO Auto-generated method stub
			latch = new CountDownLatch(1);
			client.connect(new InetSocketAddress(host, port), this, this);
			try{
				latch.await();
			}catch(InterruptedException e){
				e.printStackTrace();
			}
		}
	
		@Override
		public void completed(Void result, AsyncClientHandler attachment) {
			// TODO Auto-generated method stub
			byte []req = "Hello, Server".getBytes();
			ByteBuffer writeBuffer = ByteBuffer.allocate(req.length);
			writeBuffer.put(req);
			writeBuffer.flip();
			client.write(writeBuffer, writeBuffer, new CompletionHandler<Integer, ByteBuffer>(){
				@Override
				public void completed(Integer result, ByteBuffer attachment) {
					// TODO Auto-generated method stub
					if(attachment.hasRemaining()){
						client.write(attachment, attachment, this);
					} else {
						ByteBuffer readBuffer = ByteBuffer.allocate(1024);
						client.read(readBuffer, readBuffer, new CompletionHandler<Integer, ByteBuffer>(){
							@Override
							public void completed(Integer result, ByteBuffer attachment) {
								// TODO Auto-generated method stub
								attachment.flip();
								byte[] bytes = new byte[attachment.remaining()];
								attachment.get(bytes);
								String body;
								try {
									body = new String(bytes, "UTF-8");
									System.out.println("Receive from server:"+body);
									latch.countDown();
								} catch (UnsupportedEncodingException e) {
									......
								}							
							}
							@Override
							public void failed(Throwable exc, ByteBuffer attachment) {
								......							
							}						
						});
					}
				}
	
				@Override
				public void failed(Throwable exc, ByteBuffer attachment) {
					......
				}			
			});
		}
	
		@Override
		public void failed(Throwable exc, AsyncClientHandler attachment) {
			// TODO Auto-generated method stub
			......		
		}
	}


Server示例代码：  
1.服务器主函数：  
	
	AsyncServerHandler server = new AsyncServerHandler(8080);
	new Thread(server, "AIO-AyncServerHandler-1").start();

2.创建服务器异步处理类：  

	public class AsyncServerHandler implements Runnable {	
		private int port;
		public CountDownLatch latch;
		public AsynchronousServerSocketChannel asynchronousServerSocketChannel;
		public AsyncServerHandler(int port){
			try{
				asynchronousServerSocketChannel = AsynchronousServerSocketChannel.open();
				asynchronousServerSocketChannel.bind(new InetSocketAddress(port));			
			}catch(IOException e){
				e.printStackTrace();
			}
		}		
		@Override
		public void run() {
			// TODO Auto-generated method stub
			latch = new CountDownLatch(1);
			asynchronousServerSocketChannel.accept(this, new AcceptCompletionHandler());
			try{
				latch.await();
			}catch(InterruptedException e){
				e.printStackTrace();
			}
		}
	}

3.创建handler接收通知：

	public class AcceptCompletionHandler implements CompletionHandler<AsynchronousSocketChannel, AsyncServerHandler> {
		@Override
		public void completed(AsynchronousSocketChannel result, AsyncServerHandler attachment) {
			// TODO Auto-generated method stub
			//***To accept other client connection request
			attachment.asynchronousServerSocketChannel.accept(attachment, this);
			
			ByteBuffer buffer = ByteBuffer.allocate(1024);
			result.read(buffer, buffer, new ReadCompletionHandler(result));
		}
	
		@Override
		public void failed(Throwable exc, AsyncServerHandler attachment) {
			// TODO Auto-generated method stub
			exc.printStackTrace();
			attachment.latch.countDown();
		}
	}
	
4.创建handler读取消息和发送应答：

	public class ReadCompletionHandler implements CompletionHandler<Integer, ByteBuffer> {
		private AsynchronousSocketChannel channel;
		public ReadCompletionHandler(AsynchronousSocketChannel channel){
			if(this.channel == null) this.channel = channel;
		}
		
		@Override
		public void completed(Integer result, ByteBuffer attachment) {
			// TODO Auto-generated method stub
			attachment.flip();
			byte []body = new byte[attachment.remaining()];
			attachment.get(body);
			try{
				String req = new String(body, "UTF-8");
				System.out.println("Server received:"+req);
				sendClientContent("How are you!");
			}catch(UnsupportedEncodingException e){
				e.printStackTrace();
			}
		}
	
		@Override
		public void failed(Throwable exc, ByteBuffer attachment) {
			this.channel.close();		
		}
		
		private void sendClientContent(String content){			
			byte[] bytes = content.getBytes();
			ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);
			writeBuffer.put(bytes);
			writeBuffer.flip();
			channel.write(writeBuffer, writeBuffer, new CompletionHandler<Integer, ByteBuffer>(){
				@Override
				public void completed(Integer result, ByteBuffer attachment) {
					// TODO Auto-generated method stub
					if(attachment.hasRemaining())
						channel.write(attachment, attachment, this);
				}
	
				@Override
				public void failed(Throwable exc, ByteBuffer attachment) {
					channel.close();					
				}				
			});
		}
	}


---

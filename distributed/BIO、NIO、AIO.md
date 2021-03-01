#### BIO

同步阻塞IO，面向流，从流中读取数据，向流中写入数据。

线程调用accept()、read()、write()时，该线程被阻塞，直到有客户端连接、数据读取完成、数据写入完成为止，线程在此期间不能干其他事情。

```java
// 服务端是ServerSocket
ServerSocket server = new ServerSocket(8080);
while (true) {
  	//等待客户端连接，accept是阻塞的
  	Socket client = server.accept();
    // 获取流
  	InputStream is = client.getInputStream();
  	// 通过流读取数据
  	byte [] buff = new byte[1024];
  	int len = is.read(buff);
  	if(len > 0){
  		String msg = new String(buff, 0 ,len);
  	}
}

// 客户端是Socket，localhost:8080是服务端地址
Socket client = new Socket("localhost", 8080);
// 获取流
OutputStream os = client.getOutputStream();
// 通过流写入数据
os.write("hello".getBytes());
```

#### NIO

同时支持同步阻塞IO与同步非阻塞IO，通过configureBlocking(true/false)控制，面向缓冲区（buffer），数据从通道（channel）读取到buffer中，写入也是到buffer中。

线程请求从channel读取数据到buffer，如果目前没有数据，线程可以去做别的事情，当有数据读取到buffer中后，线程再继续处理数据。写数据也是一样的，线程请求写入数据到channel，但不需要等待它完全写入，线程可以去做别的事情，比如可以在其他channel上执行IO操作，所以一个线程可以管理多个channel。

```java
ServerSocketChannel serverChannel = ServerSocketChannel.open();
serverChannel.bind(new InetSocketAddress(8080));
// NIO模型默认是阻塞式，在此开启非阻塞模式
serverChannel.configureBlocking(false);
Selector selector = Selector.open();
// OP_ACCEPT表示有客户端发起TCP连接
serverChannel.register(selector, SelectionKey.OP_ACCEPT);
// 不断地循环，就叫轮询
while (true) {
    // select方法阻塞的等待有IO的信道，直到有信道了返回或者等待超时返回
    selector.select();
    Set<SelectionKey> keys = selector.selectedKeys();
    Iterator<SelectionKey> iter = keys.iterator();
    //同步体现在这里，因为每次只能拿一个key，每次只能处理一种状态
    while (iter.hasNext()){
        SelectionKey key = iter.next();
        if (key.isAcceptable()) {
            // 非阻塞模式下，accept方法不是阻塞的，如果没有信道，就返回null
            SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept();
            clientChannel.configureBlocking(false);
            //当数据准备就绪的时候，将状态改为可读
            clientChannel.register(selector, SelectionKey.OP_READ);
        } else if (key.isReadable()) {
            //key.channel 从多路复用器中拿到客户端的引用
            SocketChannel clientChannel = (SocketChannel)key.channel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            int len = clientChannel.read(buffer);
            // 非阻塞模式下，read方法可能没读取到数据时就返回了，所以得判断下
            if(len > 0){
                buffer.flip();
                String content = new String(buffer.array(), 0, len);
                key = channel.register(selector, SelectionKey.OP_WRITE);
                //在key上携带一个附件，一会再写出去
                key.attach(content);
            }
        } else if (key.isWritable()) {
            SocketChannel channel = (SocketChannel)key.channel();
            String content = (String)key.attachment();
            channel.write(ByteBuffer.wrap((content).getBytes()));
            channel.close();
        }
        // Selector不会自己从已选择键集中移除SelectionKey实例,必须在处理完通道时自己移除
        iter.remove();
    }
}
```


#### AIO

异步非阻塞IO

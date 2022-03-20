# 分布式网络通信



# 分布式架构网络通信

## 1.基本原理

要实现网络机器间的通讯，首先得来看看计算机系统网络通信的基本原理，在底层层面去看，网络通信需要做的就 是将流从一台计算机传输到另外一台计算机，基于传输协议和网络IO来实现，其中传输协议比较出名的有tcp、 udp等等，tcp、udp都是在基于Socket概念上为某类应用场景而扩展出的传输协议，网络IO，主要有bio、nio、 aio三种方式，所有的分布式应用通讯都基于这个原理而实现，只是为了应用的易用，各种语言通常都会提供一些 更为贴近应用易用的应用层协议。

## 2.什么是RPC

**RPC全称为remote procedure call，即远程过程调用。**

借助RPC可以做到像本地调用一样调用远程服务，是一种进程间的通信方式

比如两台服务器A和B，A服务器上部署一个应用，B服务器上部署一个应用，A服务器上的应用想调用B服务器上的 应用提供的方法，由于两个应用不在一个内存空间，不能直接调用，所以需要通过网络来表达调用的语义和传达调 用的数据。

需要注意的是**RPC并不是一个具体的技术，而是指整个网络远程调用过程**。 

###  RPC架构

![image-20200722161929048](/assets/img/post/system/rpc.png)

一个完整的RPC架构里面包含了四个核心的组件，分别是Client，Client Stub，Server以及Server Stub，这个Stub 可以理解为存根。

- 客户端(Client)，服务的调用方。
- 客户端存根(Client Stub)，存放服务端的地址消息，再将客户端的请求参数打包成网络消息，然后通过网络远程发送给服务方。
- 服务端(Server)，真正的服务提供者。
- 服务端存根(Server Stub)，接收客户端发送过来的消息，将消息解包，并调用本地的方法。

### RPC调用过程

![image-20200722162133104](/assets/img/post/system/rpc_process.png)

(1) 客户端(client)以本地调用方式(即以接口的方式)调用服务;

(2) 客户端存根(client stub)接收到调用后，负责将方法、参数等组装成能够进行网络传输的消息体(将消息体对象序列化为二进制);

(3) 客户端通过sockets将消息发送到服务端;

(4) 服务端存根( server stub)收到消息后进行解码(将消息对象反序列化);

(5) 服务端存根( server stub)根据解码结果调用本地的服务;

(6) 本地服务执行并将结果返回给服务端存根( server stub);

(7) 服务端存根( server stub)将返回结果打包成消息(将结果消息对象序列化);

(8) 服务端(server)通过sockets将消息发送到客户端;

(9) 客户端存根(client stub)接收到结果消息，并进行解码(将结果消息发序列化);

(10) 客户端(client)得到最终结果。

RPC的目标是要把2、3、4、7、8、9这些步骤都封装起来。

注意:无论是何种类型的数据，最终都需要转换成二进制流在网络上进行传输，数据的发送方需要将对象转换为二 进制流，而数据的接收方则需要把二进制流再恢复为对象。

在java中RPC框架比较多，常见的有Hessian、gRPC、Thrift、HSF (High Speed Service Framework)、Dubbo 等，其实对 于RPC框架而言，核心模块 就是通讯和序列化

## 3.RMI

### 简介

Java RMI 指的是远程方法调用 (Remote Method Invocation),是java原生支持的远程调用 ,采用JRMP(Java Remote Messageing protocol)作为通信协议，可以认为是纯java版本的分布式远程调用解决方案， RMI主要用 于不同虚拟机之间的通信，这些虚拟机可以在不同的主机上、也可以在同一个主机上，这里的通信可以理解为一个 虚拟机上的对象调用另一个虚拟机上对象的方法。

1.客户端:

- 存根/桩(Stub):远程对象在客户端上的代理;
- 远程引用层(Remote Reference Layer):解析并执行远程引用协议;
- 传输层(Transport):发送调用、传递远程方法参数、接收远程方法执行结果。

2.服务端: 

- 骨架(Skeleton):读取客户端传递的方法参数，调用服务器方的实际对象方法， 并接收方法执行后的返回值;
- 远程引用层(Remote Reference Layer):处理远程引用后向骨架发送远程方法调用; 
- 传输层(Transport):监听客户端的入站连接，接收并转发调用到远程引用层。

3.注册表(Registry):以URL形式注册远程对象，并向客户端回复对远程对象的引用。

![rmi](/assets/img/post/system/rmi.png)

#### 远程调用过程:

> 1)客户端从远程服务器的注册表中查询并获取远程对象引用。
>
> 2)桩对象与远程对象具有相同的接口和方法列表，当客户端调用远程对象时，实际上是由相应的桩对象代理完成的。
>
> 3 )远程引用层在将桩的本地引用转换为服务器上对象的远程引用后，再将调用传递给传输层(Transport)，由传输层通 过TCP协议发送调用; 
>
> 4)在服务器端，传输层监听入站连接，它一旦接收到客户端远程调用后，就将这个引用转发给其上层的远程引用层; 
>
> 5)服务器端的远程引用层将客户端发送的远程应用转换为本地虚拟机的引用后，再将请求传递给骨架(Skeleton); 
>
> 6)骨架读取参数，又将请求传递给服务器，最后由服务器进行实际的方法调用。

#### 结果返回过程:

> 1)如果远程方法调用后有返回值，则服务器将这些结果又沿着“骨架->远程引用层->传输层”向下传递;
>
>  2)客户端的传输层接收到返回值后，又沿着“传输层->远程引用层->桩”向上传递，然后由桩来反序列化这些返回值，并 将最终的结果传递给客户端程序。

### 开发流程

#### 服务端:

1)定义Remote子接口，在其内部定义要发布的远程方法，并且这些方法都要Throws RemoteException; 

2)定义实现远程接口，并且继承:UnicastRemoteObject 

3)启动服务器:依次完成注册表的启动和远程对象绑定。

#### 客户端:

1)通过符合JRMP规范的URL字符串在注册表中获取并强转成Remote子接口对象; 

2)调用这个Remote子接口对象中的某个方法就是为一次远程方法调用行为。

### 代码实现 

1.创建远程接口

```java
import java.rmi.Remote;
import java.rmi.RemoteException;
/**
* 远程服务对象接口必须继承Remote接口;同时方法必须抛出RemoteExceptino异常 */
public interface Hello extends Remote {
    public String sayHello(User user) throws RemoteException;
}
```

其中有一个引用对象作为参数

```java
import java.io.Serializable;
/**
* 引用对象应该是可序列化对象，这样才能在远程调用的时候:1. 序列化对象 2. 拷贝 3. 在网络中传输
* 4. 服务端反序列化 5. 获取参数进行方法调用; 这种方式其实是将远程对象引用传递的方式转化为值传递的方式 */
public class User implements Serializable {
    private String name;
    private int age;
    public String getName() {
        return name;
    }
		public void setName(String name) { 
  		this.name = name;
    }
    public int getAge() {
			return age; 
    }
		public void setAge(int age) { this.age = age;}
}
```

2. 实现远程服务对象

```java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;
/**
* 远程服务对象实现类写在服务端;必须继承UnicastRemoteObject或其子类
**/
public class HelloImpl extends UnicastRemoteObject implements Hello {
/**
* 因为UnicastRemoteObject的构造方法抛出了RemoteException异常，因此这里默认的构造方法必须写，必须
声明抛出RemoteException异常 *
     * @throws RemoteException
*/
		private static final long serialVersionUID = 3638546195897885959L;
    protected HelloImpl() throws RemoteException {
        super();
				// TODO Auto-generated constructor stub
		}
    @Override
    public String sayHello(User user) throws RemoteException { 	
      	System.out.println("this is server, hello:" + user.getName()); 
        return "success";
		} 
}
```

3. 服务端程序

```java
import java.net.MalformedURLException; 
import java.rmi.Naming;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry; 
/**
* 服务端程序
**/
public class Server {
    public static void main(String[] args) {
          try {
          		Hello hello = new HelloImpl(); // 创建一个远程对象，同时也会创建stub对象、skeleton对象 
            //本地主机上的远程对象注册表Registry的实例，并指定端口为8888，这一步必不可少(Java默认端口是1099)，必不可缺的一步，缺少注册表创建，则无法绑定对象到远程注册表上
            //启动注册服务
            	LocateRegistry.createRegistry(8080); 
          try { 
            //绑定的URL标准格式为:rmi://host:port/name(其中协议名可以省略，下面两种写法都是正确的)
          		Naming.bind("//127.0.0.1:8080/zm", hello); //将stub引用绑定到服务地址上 
          } catch (MalformedURLException e) {
          // TODO Auto-generated catch block
          		e.printStackTrace(); }
          		System.out.println("service bind already!!");
          } catch (RemoteException e) {
          // TODO Auto-generated catch block e.printStackTrace();
          }
   } 
}  
```

4. 客户端程序

```java
import java.net.MalformedURLException; 
import java.rmi.Naming;
import java.rmi.NotBoundException; 
import java.rmi.RemoteException;
/**
* 客户端程序 * @author zm *
*/
public class Client {
    public static void main(String[] args) {
        try { //在RMI服务注册表中查找名称为RHello的对象，并调用其上的方法
            Hello hello = (Hello) Naming.lookup("//127.0.0.1:8080/zm");
          	//获取远程对象 
          	User user = new User();
            user.setName("james");
            System.out.println(hello.sayHello(user));
        } catch (MalformedURLException e) {
        // TODO Auto-generated catch block e.printStackTrace();
        } catch (RemoteException e) {
        // TODO Auto-generated catch block e.printStackTrace();
        } catch (NotBoundException e) {
        // TODO Auto-generated catch block e.printStackTrace();
        } 
    }
}
```

5. 启动服务端程序 
6. 客户端调用

## 4.BIO、NIO、AIO 同步和异步

### 同步和异步

**同步(synchronize)、异步(asychronize)是指应用程序和内核的交互而言的.** 

**同步**:

指用户进程触发IO操作等待或者轮训的方式查看IO操作是否就绪。 

同步举例:

银行取钱,我自己去取钱,取钱的过程中等待.

**异步**:

当一个异步进程调用发出之后，调用者不会立刻得到结果。而是在调用发出之后，被调用者通过状态、通知来通知 调用者，或者通过回调函数来处理这个调用。

使用异步IO时，Java将IO读写委托给OS处理，需要将数据缓冲区地址和大小传给OS，OS需要支持异步IO操作 举个例子:

异步举例:

我请朋友帮我取钱,他取到钱后返回给我. (委托给操作系统OS, OS需要支持IO异步API)

### 阻塞和非阻塞 

**阻塞和非阻塞是针对于进程访问数据的时候,根据IO操作的就绪状态来采取不同的方式.**

简单点说就是一种读写操作方法的实现方式. 阻塞方式下读取和写入将一直等待, 而非阻塞方式下,读取和写入方法 会理解返回一个状态值.

举个例子: 

**阻塞**:

ATM机排队取款，你只能等待排队取款(使用阻塞IO的时候，Java调用会一直阻塞到读写完成才返回。) 

**非阻塞**:

柜台取款，取个号，然后坐在椅子上做其他事，等广播通知，没到你的号你就不能去，但你可以不断的问大堂经理 排到了没有。(使用非阻塞IO时，如果不能读写Java调用会马上返回，当IO事件分发器会通知可读写时再继续进行 读写，不断循环直到读写完成)

例子

老张煮开水。 老张，水壶两把(普通水壶，简称水壶;会响的水壶，简称响水壶)。

 1 老张把水壶放到火上，站立着等水开。(同步阻塞)

 2 老张把水壶放到火上，去客厅看电视，时不时去厨房看看水开没有。(同步非阻塞)

 3 老张把响水壶放到火上，立等水开。(异步阻塞)

 4 老张把响水壶放到火上，去客厅看电视，水壶响之前不再去看它了，响了再去拿壶。(异步非阻塞)

#### BIO 

**同步阻塞IO，B代表blocking**

服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连 接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。

适用场景:Java1.4之前唯一的选择，简单易用但资源开销太高

![image-20200722165654984](/assets/img/post/system/bio.png)

服务端

```java
import java.io.IOException;
import java.net.InetSocketAddress; 
import java.net.ServerSocket; 
import java.net.Socket;
public class IOServer {
    public static void main(String[] args) throws Exception {
        //首先创建了一个serverSocket
        ServerSocket serverSocket = new ServerSocket(); 
      	serverSocket.bind(new InetSocketAddress("127.0.0.1",8081)); 
      	while (true){
          Socket socket = serverSocket.accept(); //同步阻塞 
          new Thread(()->{
            try {
                byte[] bytes = new byte[1024];
                int len = socket.getInputStream().read(bytes); 
                //同步阻塞 
                System.out.println(new String(bytes,0,len)); 	
                socket.getOutputStream().write(bytes,0,len); 
                socket.getOutputStream().flush();
            } catch (IOException e) { 
              e.printStackTrace();
            } 
          }).start(); 
        }
    } 
}
```

客户端

```java
import java.io.IOException;
import java.net.Socket; 
public class IOClient {
		public static void main(String[] args) throws IOException { 
      	Socket socket = new Socket("127.0.0.1",8081); 	
      	socket.getOutputStream().write("hello".getBytes()); 
      	socket.getOutputStream().flush(); 
      	System.out.println("server send back data ====="); 
      	byte[] bytes = new byte[1024];
				int len = socket.getInputStream().read(bytes); 
      	System.out.println(new String(bytes,0,len)); 
      	socket.close();
	}
}
```

### NIO 

#### NIO介绍

同步非阻塞IO (non-blocking IO / new io)是指JDK 1.4 及以上版本。 服务器实现模式为一个请求一个通道，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有IO请求时才启动一个线程进行处理。 

**通道(Channels)**

NIO 新引入的最重要的抽象是通道的概念。Channel 数据连接的通道。 数据可以从Channel读到Buffer中，也可以 从Buffer 写到Channel中 .

**缓冲区(Buffers)** 

通道channel可以向缓冲区Buffer中写数据，也可以像buffer中存数据。 

**选择器(Selector)**

使用选择器，借助单一线程，就可对数量庞大的活动 I/O 通道实时监控和维护。

![image-20200722170518459](/assets/img/post/system/nio.png)

#### 特点

当一个连接创建后，不会需要对应一个线程，这个连接会被注册到多路复用器，所以一个连接只需要一个线程即可，所有的连接需要一个线程就可以操作，该线程的多路复用器会轮询，发现连接有请求时，才开启一个线程处 理。

**BIO和NIO的区别**

- BIO是每个新连接都创建一个新线程，NIO新连接先进行注册，然后使用一个线程处理所有新连接；
- BIO读取是以字节形式，NIO是以字节快读取，效率更高。

### AIO 

异步非阻塞IO。A代表asynchronize

当有流可以读时,操作系统会将可以读的流传入read方法的缓冲区,并通知应用程序,对于写操作,OS将write方法的流 写入完毕是操作系统会主动通知应用程序。因此read和write都是异步 的，完成后会调用回调函数。

使用场景:连接数目多且连接比较长(重操作)的架构，比如相册服务器。重点调用了OS参与并发操作，编程比 较复杂。Java7开始支持

### 5.Netty

Netty 是由 JBOSS 提供一个异步的、 基于事件驱动的网络编程框架。

Netty 可以帮助你快速、 简单的开发出一 个网络应用， 相当于简化和流程化了 NIO 的开发过程。 作为当前最流行 的 NIO 框架， Netty 在互联网领域、 大数据分布式计算领域、 游戏行业、 通信行业等获得了广泛的应用， 知名 的 Elasticsearch 、 Dubbo 框架内部都采用了 Netty。

![image-20200722171154517](/assets/img/post/system/netty.png)

#### 为什么使用Netty 

##### NIO缺点

- NIO 的类库和 API 繁杂，使用麻烦。你需要熟练掌握 Selector、ServerSocketChannel、SocketChannel、 ByteBuffer 等.
- 可靠性不强，开发工作量和难度都非常大
- NIO 的 Bug。例如 Epoll Bug，它会导致 Selector 空轮询，最终导致 CPU 100%。

##### Netty优点

- 对各种传输协议提供统一的 API 
- 高度可定制的线程模型——单线程、一个或多个线程池 
- 更好的吞吐量，更低的等待延迟

- 更少的资源消耗
- 最小化不必要的内存拷贝

#### 线程模型

![image-20200722171514037](/assets/img/post/system/netty_thread.png)

Netty 抽象出两组线程池， BossGroup 专门负责接收客 户端连接， WorkerGroup 专门负责网络读写操作。

 NioEventLoop 表示一个不断循环执行处理 任务的线程， 每个 NioEventLoop 都有一个 selector， 用于监听绑定 在其上的 socket 网络通道。 

NioEventLoop 内部采用串行化设计， 从消息的读取->解码->处理->编码->发送， 始 终由 IO 线 程 NioEventLoop 负责。

#### Netty核心组件

**ChannelHandler 及其实现类**

ChannelHandler 接口定义了许多事件处理的方法， 我们可以通过重写这些方法去实现具 体的业务逻辑

我们经常需要自定义一个 Handler 类去继承 ChannelInboundHandlerAdapter， 然后通过 重写相应方法实现业 务逻辑， 我们接下来看看一般都需要重写哪些方法:

> public void channelActive(ChannelHandlerContext ctx)， 通道就绪事件
> public void channelRead(ChannelHandlerContext ctx, Object msg)， 通道读取数据事件
> public void channelReadComplete(ChannelHandlerContext ctx) ， 数据读取完毕事件
> public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause)， 通道发生异常事件

**ChannelPipeline**

ChannelPipeline 是一个 Handler 的集合， 它负责处理和拦截 inbound 或者 outbound 的事 件和操作， 相当于

一个贯穿 Netty 的链。

> ChannelPipeline addFirst(ChannelHandler... handlers)， 把一个业务处理类(handler) 添加到链中的第一 个位置
> ChannelPipeline addLast(ChannelHandler... handlers)， 把一个业务处理类(handler) 添加到链中的最后 一个位置

**ChannelHandlerContext**

这 是 事 件 处 理 器 上 下 文 对 象 ， Pipeline 链 中 的 实 际 处 理 节 点 。 每 个 处 理 节 点 ChannelHandlerContext 中 包 含 一 个 具 体 的 事 件 处 理 器 ChannelHandler ， 同 时 ChannelHandlerContext 中也绑定了对应的 pipeline 和 Channel 的信息，方便对 ChannelHandler 进行调用。 常用方法如下所示

> ChannelFuture close()， 关闭通道
> ChannelOutboundInvoker flush()， 刷新
> ChannelFuture writeAndFlush(Object msg) ， 将 数 据 写 到 ChannelPipeline 中 当 前  ChannelHandler 的下一个 ChannelHandler 开始处理(出站)

**ChannelFuture**

表示 Channel 中异步 I/O 操作的结果， 在 Netty 中所有的 I/O 操作都是异步的， I/O 的调 用会直接返回， 调用者

并不能立刻获得结果， 但是可以通过 ChannelFuture 来获取 I/O 操作 的处理状态。 常用方法如下所示:

> Channel channel()， 返回当前正在进行 IO 操作的通道
>
> ChannelFuture sync()， 等待异步操作执行完毕

**EventLoopGroup 和其实现类 NioEventLoopGroup**

EventLoopGroup 是一组 EventLoop 的抽象， Netty 为了更好的利用多核 CPU 资源， 一般 会有多个 EventLoop 同时工作， 每个 EventLoop 维护着一个 Selector 实例。 EventLoopGroup 提供 next 接口， 可以从组里面按照一 定规则获取其中一个 EventLoop 来处理任务。 在 Netty 服务器端编程中， 我们一般都需要提供两个 EventLoopGroup， 例如: BossEventLoopGroup 和 WorkerEventLoopGroup。

> public NioEventLoopGroup()， 构造方法
> public Future<?> shutdownGracefully()， 断开连接， 关闭线程

**ServerBootstrap 和 Bootstrap**

ServerBootstrap 是 Netty 中的服务器端启动助手，通过它可以完成服务器端的各种配置; Bootstrap 是 Netty 中

的客户端启动助手， 通过它可以完成客户端的各种配置。 常用方法如下 所示:

> public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup)，该方法用于 服务器端， 用来设置两个 EventLoop
> public B group(EventLoopGroup group) ， 该方法用于客户端， 用来设置一个 EventLoop
> public B channel(Class<? extends C> channelClass)， 该方法用来设置一个服务器端的通道实现
>
> public <T> B option(ChannelOption<T> option, T value)， 用来给 ServerChannel 添加配置
> public <T> ServerBootstrap childOption(ChannelOption<T> childOption, T value)， 用来给接收到的 通道添加配置
> public ServerBootstrap childHandler(ChannelHandler childHandler)， 该方法用来设置业务处理类(自定 义的 handler)
> public ChannelFuture bind(int inetPort) ， 该方法用于服务器端， 用来设置占用的端口号
> public ChannelFuture connect(String inetHost, int inetPort) 该方法用于客户端， 用来连接服务器端



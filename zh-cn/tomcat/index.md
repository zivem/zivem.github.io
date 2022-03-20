# Tomcat详解


# Tomcat详解

## 目录结构

### bin：tomocat启动文件

### conf

- 1.logging.properties 日志配置；
- 2.server.xml服务器端口等配置；
- 3.tomcat-users.xml tomcat用户角色、权限配置；
- 4.web.xml应用全局默认配置，若服务中进行了配置，则会覆盖

### lib：基础jar包

### temp ：临时文件

### webapps ：发布项目的目录，管理界面

### work：JSP 解析成Java类，中间会产生过程文件，编译运行中间文件

## 架构及原理剖析

### 架构

- http请求过程
- tomcat是一个http服务器，能够接收http请求
- Tomcat Servlet容器处理流程

	- 1.HTTP服务器会把请求信息使用ServletRequest对象封装起来 ;
2.进一步去调用Servlet容器中某个具体Servlet;
3.在 (2)中，Servlet容器拿到请求后，根据URL和Servlet的映射关系，找到相应的Servlet ;
4.如果Servlet还没有被加载，就用反射机制创建这个Servlet，并调用Servlet的init方法来完成初始化 ;
5.接着调用这个具体Servlet的service方法来处理请求，请求处理结果使用ServletResponse对象封装 ;
6.把ServletResponse对象返回给HTTP服务器，HTTP服务器会把响应发送给客户端

- 总体架构，核心组件

	- 连接器Connector

		- 职责：负责对外交流: 处理Socket连接，负责网络字节流与Request和Response对象的转化;
		- 连接器组件 Coyote

			- 简介

				- 封装网络通信，(Socket 请求及响应处理)
				- Catalina 容器(容器组件)与具体的请求协议及IO操作方式完全解耦
				- 将Socket 输入转换封装为 Request 对象，进一步封装后交由Catalina 容器进行处理，处 理请求完成后, Catalina 通过Coyote 提供的Response 对象将结果写入输出流
				- Coyote 负责的是具体协议(应用层)和IO(传输层)相关内容

			- I/O模型及协议

				- 应用层协议

					- Http/1.1 默认，Web访问协议
					- AJP：和WX集成（如Apache），已实现对静态资源的优化以及集群部署
					- HTTP/2

				- I/O模型（传输层）

					- NIO：非阻塞I/O
					- NIO2:异步I/O，采用JDK 7 NIO2类库
					- APR ：采用Apache可移植运行库实现

			- 内部组件

				- EndPoint：通信端点，通信监听接口，Socket接收和发送处理，传输层的抽象，用来实现TCP/IP协议。
				- Processor：应用层协议抽象，协议处理接口，实现Http协议，接收EndPoint的Socket，读取字节流解析成Tomcat Request和Response对象，通过Adapter提交到容器处理。
				- ProcessorHandler：Coyote协议接口，通过EndPoint和Processor，实现具体协议的处理能力
				- Adapter：由于协议不同，客户端发送的请求信息不同，Tomcat定义自己的Request对象，ProcessorHandler解析请求生存Tomcat  Request对象，适配器模式，连接器抵用CoyoteAdapter的Service方法，将Tomcat Request 转成ServletRequest，再调用容器

			- 流程

	- 容器Contain

		- 职责：负责内部处理:加载和管理Servlet，以及具体处理Request请求;
		- 容器 Catalina

			- 结构

				- 一个Catalina实例
				- 一个Server实例
				- 包含多个Service服务
				- 多个Container，这里不再是Catalina，Catalina概念提升

					- Engine：整个Catalina Servlet引擎，一个Service智能有一个Engine
					- Host：代表一个虚拟主机或站点
					- Context：表示一个Web应用程序
					- Wrapper：表示一个Servlet，容器的最底层，不能包含子容器

- Tomcat分层结构图

	- Servlet容器 Catalina，核心
	- 连接器Coyote，网络通信
	- JSP引擎Jasper
	- Naming（命名服务）
	- Juli服务器日志

## 核心配置

### server.xml

- Server：根元素，创建一个Server实例
- Listener：监听器

	- VersionLoggerListener：输出服务器，操作系统，JVM版本信息
	- AprLifecycleListener：加载销毁APR
	- JreMemoryLeakPreventListener：避免JRE内存溢出
	- GlobalResourcesLifecycleListener： 加载(服务器启动) 和 销毁(服务器停止) 全局命名服务
	- ThreadLocalLeakPreventionListener：在Context停止时重建 Executor 池中的线程， 以避免ThreadLocal 相关的内存泄漏

- GlobalNamingResources：定义服务器全局JNDI资源
- Service：服务实例，name= Catalina

	- Connector标签

		- port:  端口号，Connector 用于创建服务端Socket 并进行监听， 以等待客户端请求链接。如果该属性设置 为0， Tomcat将会随机选择一个可用的端口号给当前Connector 使用
		- protocol:  当前Connector 支持的访问协议。 默认为 HTTP/1.1 ， 并采用自动切换机制选择一个基于 JAVA NIO 的链接器或者基于本地APR的链接器(根据本地是否含有Tomcat的本地库判定) 
		- connectionTimeOut:  Connector 接收链接后的等待超时时间， 单位为 毫秒。 -1 表示不超时。 
		- redirectPort:  当前Connector 不支持SSL请求， 接收到了一个请求， 并且也符合security-constraint 约束， 需要SSL传输，Catalina自动将请求重定向到指定的端口。默认：8443
		- executor:指定共享线程池的名称， 也可以通过maxThreads、minSpareThreads 等属性配置内部线程池。
		- URIEncoding:用于指定编码URI的字符编码， Tomcat8.x版本默认的编码为 UTF-8 , Tomcat7.x版本默认为ISO-8859-1

	- Listener标签
	- Executor标签：共享线程池，可以共享给多个Connector使用

		- name: 线程池名称，用于 Connector中指定
		- namePrefix: 所创建的每个线程的名称前缀，一个单独的线程名称为 namePrefix+threadNumber
		- maxThreads: 池中最大线程数 
		- minSpareThreads: 活跃线程数，也就是核心池线程数，这些线程不会被销毁，会一直存在 
		- maxIdleTime:线程空闲时间，超过该时间后，空闲线程会被销毁，默认值为6000(1分钟)，单位毫秒
		- maxQueueSize:在被执行前最大线程排队数目，默认为Int的最大值，也就是广义的无限。除非特殊情况，这个值 不需要更改，否则会有请求不会被处理的情况发生 
		- prestartminSpareThreads:启动线程池时是否启动 minSpareThreads部分线程。默认值为false，即不启动 
		- threadPriority:线程池中线程优先级，默认值为5，值从1到10 
		- className:线程池实现类，未指定情况下，默认实现类为org.apache.catalina.core.StandardThreadExecutor。如果想使用自定义线程池首先需要实现 org.apache.catalina.Executor接口

	- Engine标签

		- name: 用于指定Engine 的名称， 默认为Catalina
		- defaultHost:默认使用的虚拟主机名称， 当客户端请求指向的主机无效时， 将交由默认的虚拟主机处 理， 默认为localhost


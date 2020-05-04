# Http协议请求格式

```http
GET / HTTP/1.1 ##请求行
Host: localhost:9001    ##以下请求头
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.113 Safari/537.36 chrome-extension chrome-extension
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: Idea-fd09797b=e91c003e-3002-4648-9e7f-57edbabd1d60
空行
##请求体
```

![](images\图片1.png)

# Http协议响应格式

![](D:\0_LeargingSummary\Tomcat&Http\images\图片2.png)

# Tomcat

![](images\tomcat整体架构图.webp)

![](images\Tomcat核心类图.png)

## Catalina

```java
//Catalina的启动/关闭Shell程序。
    public class Catalina {}
```

## Server

```java
/*
一个Server元素代表了整个Catalina Servlet容器。它的属性代表了整个Servlet容器的特征。一个server可以包含一个或多个Services，还有顶级命名资源集。
正常情况下，这个接口的实现也将实现Lifecycle，这样当start和stop方法被调用后，所有已定义的Services也被启动和停止。
在这之间，实现者必须在port属性指定端口数字上打开server socket。当接收到一个连接，读取第一行并将其与指定的关闭命令比较。如果匹配，服务器发起关闭。
注：该类的具体实现应该在构造函数中向ServerFactory中注册（单例）
 */
public interface Server extends Lifecycle {
    
    public Catalina getCatalina();
}
```

## Service

service是包含Connector和Container的集合，Service用适当的Connector接受用户的请求，在发给相应的Container来处理

```java
/*
一个Service是一组一个或多个连接器，共享一个单独的容器去执行接收到的请求。比如，一个非SSL和SSL连接共享相同的数量的web应用程序。
一个给定的JVM能包含任何数量的Service实例；但是，他们是完全互相独立的并且只在系统类路径上共享基础的JVM设备和Class.
*/
public interface Service extends Lifecycle {
    public Server getServer();
    
    public Container getContainer();
    
    public void addConnector(Connector connector);
}
```



## Connector

实现某一协议的连接器，如默认的实现HTTP、HTTPS、AJP协议

```java
public class Connector extends LifecycleMBeanBase  {
    
    protected Service service = null;
    public Service getService() {
        return this.service;
    }
    
    **
     * The request scheme that will be set on all requests received
     * through this connector.
     */
    protected String scheme = "http";
}
```

## Container

```java
/*
一个Container是一个Object能够执行从Client接收到的请求，并且能基于这些请求返回响应。一个容器可以选择支持Valves管道，在运行时按配置顺序执行请求，通过实现Pipeline接口。

Container在Cataline中存在几种概念级别：
Engine:表示整个Catalina Servlet驱动，很有可能包含一个或多个子容器，可以是主机或是上下文实现，或是其他自定义组。
Host：表示虚拟主机包含了多个上下文
Context:表示一个单独的ServletContext,这个通常包含一个或多个Wrappers支持servlet
Wrapper:表示一个单独Servlet定义（如果servlet本身实现了SingleThreadModel,支持多Servlet实例）。

一个给定部署的Catalina不需要包含上述多有级别的容器。比如，一个嵌入在网络设备（例如路由器）中的管理应用程序可能只包含一个单独的上下文和几个Wrappers,或者就一个单独的Wrapper如果一个应用绝对小。因此，需要设计容器实现，那样他们在给定部署中缺少父容器的情况下正确运行。

一个容器可以与许多支持的组件相关联，这些组件提供了共享（通过将它添加到父容器）或单独定制的功能。下面支持的组件是目前公认的：
Loader:类加载器，用于将新的java classes整合到运行Catalina的JVM容器中。
Logger:ServletContext接口的log()方法的实现
Manager:该容器的会话池管理器
Realm:安全域的只读接口，为了验证用户的身份和他们相应的角色
Resources:JNDI目录上下文能够访问静态资源，当Catalina在较大的服务器中是嵌入式的，可以对服务器组件自定义连接。
*/
public interface Container extends Lifecycle {
    public Pipeline getPipeline();
    public Manager getManager();
    public Log getLogger();
    public Loader getLoader();
    public Realm getRealm();
    public void setParent(Container container);
    public Container getParent();
    public void addChild(Container child);
    public Container findChild(String name);
}
```

> 可以理解为处理某类型请求的容器，处理的方式一般把处理请求的处理器包装为Valve对象，并按照一定的顺序放入类型为Pipeline的管道里。Container有很多种子类型：Engine、Host、Context、Wrapper

## Engine

```java
/**
一个Engine是一个容器，代表了整个Catalina Servlet引擎。在以下的类型方案中是很有用的：
期望使用拦截器，查看通过整个engine处理的每个单独请求；
希望用单独的HTPP连接器运行Catalina,但是仍然想支持多虚拟主机。

一般来说，当部署连接到web服务的Catalina时，不会使用Engine,因为连接器将利用web服务设备来确定哪个Context(甚至是哪个包装器)被用来处理这个request;

孩子容器被添加到Engine中的通常是Host（代表主机）或Context（单独的Servlet context）的实现,取决于Engine的实现。

如果被使用，Engin在Catalina的级别中总是最高级容器。因此，实现者的setParent的方法应该抛出异常IllegalArgumentException
*/
public interface Engine extends Container {

    public String getDefaultHost();

    public void setDefaultHost(String defaultHost);

    public String getJvmRoute();

    public void setJvmRoute(String jvmRouteId);

    public Service getService();

    public void setService(Service service);
}
```

> Engine包含Host和Context，接到请求后仍给相应的Host在相应的Context里处理

## Host

```java
/*
Host是一个容器，在Catalina Servlet引擎中代表了一个虚拟主机。在下面这几个情景类型中使用：
希望使用拦截器，通过特定的虚拟主机查看处理每个单独的请求；
希望用独立的HTTP连接器运行Catalina,但是仍然想支持多个虚拟主机。

*/
public interface Host extends Container {
    
}
```

> 就是虚拟主机

## Context

```java
/*
Context是一个容器在Catalina servlet引擎中代表一个servlet上下文，因此是一个单独的web应用程序。即使是连接到web服务的连接器使用web server的设备来表示合适的Wrapper来处理这个请求，它在每一个部署的catalina中都是很有用的。它也提供了方便的机制使用拦截器，通过特定的web应用来查看每一个请求处理。

通常关联到上下文的父容器是HOST，但是可能是一些其他的实现，或者如果不是必须的可能被忽略。

附加到Context中的子容器通常是Wrapper的实现（代表了但单独的servlet定义）
*/
public interface Context extends Container {
    
}
```

> 就是我们所部署的具体的web应用上下文，每个请求都是在相应的上下文利处理的

## Wrapper

```java
/*
一个Wrapper是一个容器，代表了web应用程序的部署描述符中的单独的servlet定义。提供方便的机制使用拦截器，查看表示该定义的servlet的每个单独的请求。

Wrapper的实现是负责管理servlet生命周期，包括在合适的时机调用init()和destroy()，同时遵守servlet class本身声明的SingleThreadModel。

附加到Wrapper中的父容器通常是一个Context实现，表示了servlet上下文（因此是一个web应用）

包装器实现不允许使用子容器，所以addChild方法会抛出异常IllegalArgumentException
*/
public interface Wrapper extends Container {}
```

> Wrapper针对每个Servlet的Container，每个Servlet都有相应的Wrapper来管理

> ### 从上面一系列的介绍，看出Server,Service,Connector,Container,Engine,Host,Context,Wrapper这些核心组件的作用范围是逐层递减，并逐层包含。

## Container基础组件

### Loader

```java
/*
Loader代表了java ClassLoader实现，，在容器中用来加载class文件，旨在根据要求重新加载，以及有一个检测机制，查看基础存储库是否发生了改变。

为了Loader的实现和Context的实现能顾成功一起运行，必须遵守以下约束：
必须实现Lifecycle，这样上下文能够指示需要新的类加载器；
start()方法必须无条件创建一个新的ClassLoader实现；
stop()必须放弃先前使用ClassLoader引用，以便可以垃圾回收，回收所有通过它加载的类以及这些类的对象；
必须在同一个Loader实例上先调用stop再调用start方法；
基于实现选择策略，通过这个class loader检测到一个或多个class文件被加载，在自己的上下文中用context.redload方法
*/
public interface Loader {
    public Container getContainer();
}
```

> 被Container用来载入各种所需的Class

### Manager

```java
/*
Manager管理关联指定容器的会话池。不同的Manager实现可以支持Value-added特性，比如持久化存储session数据，以及可分发Web应用程序的转发会话。

为了Manager实现和Context的实现一起成功操作，它必须遵守以下几点：
必须实现Lifecycle,那样Context可以指示重新启动，在需要的时候；
必须在同一个Manager实例中，允许调用stop()其次调用start()
*/
public interface Manager {
    public Container getContainer();
    public int getSessionIdLength();
    public Session createSession(String sessionId);
    public Session findSession(String id) throws IOException;
}
```

> 就是用来管理session池的

### Realm

```java
/*
Realm用于底层安全领域验证单个用户，并且坚定这些用户的安全角色。Realms能够附加到任何级别的Container,但是典型的被关联到Context，或是更高级别Container。
*/
public interface Realm {
    public Container getContainer();
    public Principal authenticate(String username);
    public Principal authenticate(String username, String credentials);
}
```

> 用来处理安全授权与认证

## Pipeline

```java
/*
该接口描述了Valves集合，当invoke()方法被执行，该集合应该被有序执行。这要求管道中某处的Valve（通常是最后一个）必须处理request并且创建相应的response。

通常有一个单独的Pipeline实例关联到每一个容器中。容器的常规请求处理特性通常被封装到指定容器的Valve中，这个在管道最后应该总会被执行。setBasic()方法提供天添加Valve实例。在基础的Valve执行前，其他的Valves将被按顺序执行
*/
public interface Pipeline {
    public void setBasic(Valve valve);
    public Valve getBasic();
    public Container getContainer();
    public void setContainer(Container container);
    public Valve[] getValves();
}
```

## Valve

```java
/*
Valve关联到特定的Container中是request处理组件。通常一系列的Valves在Pipeline中互相关联。Valve详细内容在invoke()方法中。
注：之所以为该概念分配“阀门”名称，是因为您在现实世界的管道中使用阀门来控制和/或修改通过它的流量。
*/
public interface Valve {
    public Valve getNext();
    public void setNext(Valve valve);
    public void invoke(Request request, Response response)
        throws IOException, ServletException;
    public boolean isAsyncSupported();
}
```

## Tomcat启动流程分析

![](images\Tomcat启动流程图.png)

从上图可知，tomcat启动分为两步，先是init和start两个过程，核心的类都是实现了Lifecycle接口，都需实现start方法，都是从Server开始逐层调用子组件的start过程。

```
1.运行引导程序BootStrap的main函数
	创建Bootstrap的对象bootstrap，调用bootstrap的init方法，（1）这里初始化设置Catalina的home、基础信息，创建catalinaClassLoader并设置给当前线程；（2）通过catalinaClassLoader加载Catalina，并创建对应的Catalina对象（startupInstance），通过反射的获取startupInstance对象的setParentClassLoader方法并执行
	
2.调用bootstrap的load方法
	通过反射获得启动程序catalina的load方法，并执行load方法；
	创建digester，解析xml文件的输入流，比如server.xml
	给StandardServer，的initInternal()方法；服务器对象设置Catalina对象，调用服务器对象的init方法（因为继承了LifecycleMBeanBase）,会进入到LifecycleBase的init方法，然后再调用StandardServer的initInternal()方法：
	（1）获得JmxMBeanServer对象；
	（2）创建beanfactory并设置容器；
	（3）通过catalina获得classload，如果存在需要加载的路径（这里还涉及到了Classload的双亲委派机制）；
	（4）调用services的init方法，同样service继承LifecycleMBeanBase，调用initInternal：
		（1）StandardEngine初始化
		（2）初始化connector:
				(1)给Http11Protocol设置CoyoteAdapter（请求处理器的实现，请求处理器全部委托给该CoyoteAdapter）
		（3）设置setParseBodyMethods，比如POST
		（4）Http11Protocol的初始化init
				(1)创建SSLImplementation对象
				（2）设置endpoint，然后初始化endpoint(JIoEndpoint)

3.调用bootstrap的start方法
	通过反射获取catalina的start方法，并执行
	获得StandardServer，调用start方法，进入到startInternal方法：
	（1）发射生命周期事件configure_start
	（2）设置状态为starting
	（3）调用startardService的start方法：
		（1）设置service的状态为starting
		（2）standardEngine的start方法
			提交任务standardHost开启到线程池中
			调用standardPipeline的start方法，设置状态starting
		（3）调用connector的start方法
			设置状态starting
			调用protocolHandler（http11protocol）的start方法
				调用endpoint的start方法
				*******这里涉及到tomcat怎么处理接受的请求，具体代码看下面***********
```

```java
//endpoint.start();到下方代码
public final void start() throws Exception {
    if (bindState == BindState.UNBOUND) {
        bind();
        bindState = BindState.BOUND_ON_START;
    }
    startInternal(); //进入这行代码
}

//来到了JIoEndPoint类中
public void startInternal() throws Exception {
    if (!running) {
        running = true;
        paused = false;
        // Create worker collection
        if (getExecutor() == null) {
            createExecutor();//创建了一个线程池
        }
        initializeConnectionLatch();
        
        //重点：进入这行代码
        startAcceptorThreads();
        
        //这里开启一个后台线程，用来检查异步线程超时
        Thread timeoutThread = new Thread(new AsyncTimeout(),
                                          getName() + "-AsyncTimeout");
        timeoutThread.setPriority(threadPriority);
        timeoutThread.setDaemon(true);
        timeoutThread.start();
    }
}

//来到AbstractEndPoint类
protected final void startAcceptorThreads() {
    int count = getAcceptorThreadCount();
    acceptors = new Acceptor[count];

    for (int i = 0; i < count; i++) {
        //进入这行代码，这里是创建了一个线程任务
        acceptors[i] = createAcceptor();
        String threadName = getName() + "-Acceptor-" + i;
        acceptors[i].setThreadName(threadName);
        Thread t = new Thread(acceptors[i], threadName);
        t.setPriority(getAcceptorThreadPriority());
        t.setDaemon(getDaemon());
        t.start();// 开启加收请求线程
    }
}

//来到JIoEndpoint类
protected class Acceptor extends AbstractEndpoint.Acceptor {
    @Override
    public void run() {
        int errorDelay = 0;
        // Loop until we receive a shutdown command
        while (running) {//这里是无限循环
            // Loop if endpoint is paused
            while (paused && running) {
                state = AcceptorState.PAUSED;
                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                    // Ignore
                }
            }
            if (!running) {
                break;
            }
            state = AcceptorState.RUNNING;
            try {
                //if we have reached max connections, wait
                countUpOrAwaitConnection();
                Socket socket = null;
                try {
                    //这里就是用来接收请求*************
                    socket = serverSocketFactory.acceptSocket(serverSocket);
                } catch (IOException ioe) {
                    countDownConnection();
                    // Introduce delay if necessary
                    errorDelay = handleExceptionWithDelay(errorDelay);
                    // re-throw
                    throw ioe;
                }
                // Successful accept, reset the error delay
                errorDelay = 0;
                // Configure the socket
                if (running && !paused && setSocketOptions(socket)) {
                    // Hand this socket off to an appropriate processor
                    //处理接收到的请求socket,进入这行代码*****************
                    if (!processSocket(socket)) {
                        countDownConnection();
                        // Close socket right away
                        closeSocket(socket);
                    }
                } else {
                    countDownConnection();
                    // Close socket right away
                    closeSocket(socket);
                }
				//下面代码省略不展示
    }
}
//来到处理请求代码
protected boolean processSocket(Socket socket) {
    // Process the request from this socket
    try {
        //将这个socket包装成SocketWrapper
        SocketWrapper<Socket> wrapper = new SocketWrapper<Socket>(socket);
        wrapper.setKeepAliveLeft(getMaxKeepAliveRequests());
        wrapper.setSecure(isSSLEnabled());
        // During shutdown, executor may be null - avoid NPE
        if (!running) {
            return false;
        }
        //**************包装成一个线程任务提交到线程池中
        getExecutor().execute(new SocketProcessor(wrapper));
    } catch (RejectedExecutionException x) {
        log.warn("Socket processing request was rejected for:"+socket,x);
        return false;
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        // This means we got an OOM or similar creating a thread, or that
        // the pool and its queue are full
        log.error(sm.getString("endpoint.process.fail"), t);
        return false;
    }
    return true;
}
```

> 从这里开始，这上面的启动流程tomcat7的bio，现在tomcat8.5已经换成了nio

```java
//tomcat8.5
//当调用Connector的startInternal方法后
protected void startInternal() throws LifecycleException {
    // Validate settings before starting
    if (getPort() < 0) {
        throw new LifecycleException(sm.getString(
            "coyoteConnector.invalidPort", Integer.valueOf(getPort())));
    }
    setState(LifecycleState.STARTING);//设置Connector的状态为Starting
    try {
        protocolHandler.start();//进入这行代码(Http11NioProtocl)
    } catch (Exception e) {
        throw new LifecycleException(
            sm.getString("coyoteConnector.protocolHandlerStartFailed"), e);
    }
}

//来到了AbstractProtocol
public void start() throws Exception {
    if (getLog().isInfoEnabled()) {
        getLog().info(sm.getString("abstractProtocolHandler.start", getName()));
    }

    endpoint.start();//进入该行代码

    // Start timeout thread
    asyncTimeout = new AsyncTimeout();
    Thread timeoutThread = new Thread(asyncTimeout, getNameInternal() + "-AsyncTimeout");
    int priority = endpoint.getThreadPriority();
    if (priority < Thread.MIN_PRIORITY || priority > Thread.MAX_PRIORITY) {
        priority = Thread.NORM_PRIORITY;
    }
    timeoutThread.setPriority(priority);
    timeoutThread.setDaemon(true);
    timeoutThread.start();
}

//来到AbstractEndPoint类
public final void start() throws Exception {
    if (bindState == BindState.UNBOUND) {
        bind();
        bindState = BindState.BOUND_ON_START;
    }
    startInternal();//进入该行代码
}

//来到了NioEndPoint类
public void startInternal() throws Exception {
        if (!running) {
            running = true;
            paused = false;
            processorCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                    socketProperties.getProcessorCache());
            eventCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                            socketProperties.getEventCache());
            nioChannels = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,
                    socketProperties.getBufferPool());
            // Create worker collection
            if ( getExecutor() == null ) {
                createExecutor();
            }
            //创建一个LimitLatch，用来控制连接并发数
            initializeConnectionLatch();
            // Start poller threads
            pollers = new Poller[getPollerThreadCount()];
            for (int i=0; i<pollers.length; i++) {
                pollers[i] = new Poller();
                Thread pollerThread = new Thread(pollers[i], getName() + "-ClientPoller-"+i);
                pollerThread.setPriority(threadPriority);
                pollerThread.setDaemon(true);
                pollerThread.start();
            }

            startAcceptorThreads();
        }
    }
```



## 生命周期Lifecycle

### Lifecycle

```java
/*
组件生命周期的通用接口方法。Catalina组件为了提供开始和停止组件可以实现这个接口（以及适用于他们支持的功能性接口）。

支持组件的有效状态的转换:
任何状态能够过渡到Failed状态。
当组件处于Starting_prep,starting或started状态时调用start()方法是无效的；
当组件状态是NEW将会调用start()方法，在start()方法被调用后init()方法会立即被调用；
当组件状态处于stopping_prep,stopping或stopped时，调用stop()方法时无效的；
当组件处于状态NEW时调用stop()方法，状态会转化到Stopped。当组件启动失败并且没有启动它的所有子组件时，通常会遇到这种情况。当组件是停止的，它将尝试停止所有的子组件，即使没有启动的组件；

尝试任何其他转换将抛出异常LifecycleException

如果尝试转换是无效的，不会触发任何生命周期事件。
*/
public interface Lifecycle {}
```

### LifecycleBase

```java
/*
Lifecycle接口的基础实现，实现状态转化的规则start()和stop()方法。
*/
public abstract class LifecycleBase implements Lifecycle {}
```

### LifecycleMBeanBase

```java
public abstract class LifecycleMBeanBase extends LifecycleBase
        implements JmxEnabled {}
```

## Tomcat一次完整请求过程

### 线程模型结构

![](D:\0_LeargingSummary\Tomcat&Http\images\线程模型.jpg)

1. NioEndpoint中startInternal方法中开启上图中的Acceptor线程，Poller线程以及Executor线程池

   ```java
   // Create worker collection
   if ( getExecutor() == null ) {
       createExecutor();
   }
   // Start poller threads
   pollers = new Poller[getPollerThreadCount()];
   for (int i=0; i<pollers.length; i++) {
       pollers[i] = new Poller();
       Thread pollerThread = new Thread(pollers[i], getName() + "-ClientPoller-"+i);
       pollerThread.setPriority(threadPriority);
       pollerThread.setDaemon(true);
       pollerThread.start();
   }
   
   startAcceptorThreads();
   ```

2. tomcat接受请求

   - Acceptor线程任务就收到连接请求：socket = serverSock.accept();

   - 设置socket的操作：setSocketOptions(socket)

     - NioChannel channel = nioChannels.pop();获得一个NioChannel对象

     - channel.setIOChannel(socket);用NioChannel包装socket

     - getPoller0().register(channel);获得一个Poller线程，注册channel

       - ```java
         public void register(final NioChannel socket) {
                     socket.setPoller(this);
             		//将socket包装成NioSocketWrapper
                     NioSocketWrapper ka = new NioSocketWrapper(socket, NioEndpoint.this);
                     socket.setSocketWrapper(ka);
                     ka.setPoller(this);
             		//这里就是设置一些配置
                     ka.setReadTimeout(getSocketProperties().getSoTimeout());
                     ka.setWriteTimeout(getSocketProperties().getSoTimeout());
                     ka.setKeepAliveLeft(NioEndpoint.this.getMaxKeepAliveRequests());
                     ka.setSecure(isSSLEnabled());
                     ka.setReadTimeout(getConnectionTimeout());
                     ka.setWriteTimeout(getConnectionTimeout());
             		//这里获得一个PollerEvent任务，这是一个空任务
                     PollerEvent r = eventCache.pop();
                     ka.interestOps(SelectionKey.OP_READ);
                     if ( r==null) r = new PollerEvent(socket,ka,OP_REGISTER);
             		//用PollerEvent对socket，NioSocketWrapper及感兴趣的操作 包装
                     else r.reset(socket,ka,OP_REGISTER);
             //将这个PollerEvent添加到队列中SynchronizedQueue<PollerEvent> events
                     addEvent(r);
                 }
         ```

   - Poller处理events中的任务

     ```java
     while (iterator != null && iterator.hasNext()) {
         SelectionKey sk = iterator.next();
         NioSocketWrapper attachment = (NioSocketWrapper)sk.attachment();
         // Attachment may be null if another thread has called
         // cancelledKey()
         if (attachment == null) {
             iterator.remove();
         } else {
             iterator.remove();
             processKey(sk, attachment);//处理,进入下方代码
         }
     }
     ```

     ```java
     //写数据之前先读数据
     if (sk.isReadable()) {
         //进入这个判断中的代码，进入下方代码
         if (!processSocket(attachment, SocketEvent.OPEN_READ, true)) {
             closeSocket = true;
         }
     }
     if (!closeSocket && sk.isWritable()) {
         if (!processSocket(attachment, SocketEvent.OPEN_WRITE, true)) {
             closeSocket = true;
         }
     }
     if (closeSocket) {
         cancelledKey(sk);
     }
     ```

     ```java
     SocketProcessorBase<S> sc = processorCache.pop();
     if (sc == null) {
         sc = createSocketProcessor(socketWrapper, event);
     } else {
         sc.reset(socketWrapper, event);//用SocketProcessorBase对socketWrapper,event包装
     }
     //提交到了Worker线程池中
     Executor executor = getExecutor();
     if (dispatch && executor != null) {
         executor.execute(sc);
     } else {
         sc.run();
     }
     ```

### 处理流程

见流程图tomcat请求流程

对request的处理

1. Http11Processor阶段
   - parseRequestLine解析请求行
   - parseHeaders 解析请求头
   - prepareRequest()设置请求过滤器
2. CoyoteAdapter阶段
   - postParseRequest 解析并设置Catalina和指定配置

## JMX

JMX（Java Management Extensions，即Java管理扩展）是一个为应用程序、设备、系统等[植入](https://baike.baidu.com/item/植入/7958584)管理功能的框架

### MBeanServer

```java
/*
这是在代理端进行MBean操作的接口。 它包含创建，注册和删除MBean所必需的方法，以及已注册MBean的访问方法。 这是JMX基础结构的核心组件。

用户代码通常不实现此接口。 而是使用{@link javax.management.MBeanServerFactory}类中的方法之一获得实现此接口的对象。

添加到MBean服务器的每个MBean都变得易于管理：可以通过连接到该MBean服务器的连接器/适配器远程访问其属性和操作。 除非Java对象是符合JMX的MBean，否则无法在MBean服务器中注册。

*/
```

想要注册到MBeanServer中就必须要符合JMX的标准

```java
//例子
//作为标准MBEAN而被管理，就需要实现一个接口。这个接口的名称必须是类名加上MBean。
public interface TomcatUtilMBean {
    void setServerName(String serverName);

    String getServerName();

    void setPort(int port);

    int getPort();

    String getTomcatInfo();

    int add(int x ,int y);
}

public class TomcatUtil implements TomcatUtilMBean {

    private String serverName = "Catalina";
    private int port = 8080;

    @Override
    public String getServerName() {
        return serverName;
    }

    @Override
    public void setServerName(String serverName) {
        this.serverName = serverName;
    }

    @Override
    public int getPort() {
        return port;
    }

    @Override
    public void setPort(int port) {
        this.port = port;
    }

    @Override
    public String getTomcatInfo() {
        return "The Tomcat's name is " + serverName + ",port is " + port;
    }

    @Override
    public int add(int x, int y) {
        return x+y;
    }
}

public class TomcatMonitor {
    public static void main(String[] args) throws MalformedObjectNameException, NotCompliantMBeanException, InstanceAlreadyExistsException, MBeanRegistrationException, InterruptedException {
        MBeanServer mBeanServer = ManagementFactory.getPlatformMBeanServer();
        TomcatUtil tomcatUtil = new TomcatUtil();
        mBeanServer.registerMBean(tomcatUtil,new ObjectName("myBean:name=tomcatUtil"));
        while (true) {
            Thread.sleep(1000);
        }
    }
}
```

> 启动jconsole,对这个java程序进行监控，点击MBean，可以查看管理注册的实例

## 基于JMX

Tomcat会为每个组件进行注册过程，通过Registry管理起来，而Registry是基于JMX来实现的，因此在看组件的init和start过程实际上就是初始化MBean和触发MBean的start方法，会大量看到形如：

Registry.getRegistry(null, null).invoke(mbeans, "init", false);

Registry.getRegistry(null, null).invoke(mbeans, "start", false);

这样的代码，这实际上就是通过JMX管理各种组件的行为和生命期。

## 事件侦听

各个组件在其生命期中会有各种各样行为，而这些行为都有触发相应的事件，Tomcat就是通过侦听这些时间达到对这些行为进行扩展的目的。在看组件的init和start过程中会看到大量如：

fireLifecycleEvent(AFTER_START_EVENT, null);这样的代码，这就是对某一类型事件的触发，如果你想在其中加入自己的行为，就只用注册相应类型的事件即可。

## Tomcat问题

1、context是如何选择对应的Servlet？

每一个Wrapper代表了一个Servlet，在StandardContextValve中通过（org.apache.catalina.connector）Request.getWrapper获取对应的Wrapper,每一个Request中包含了MappingData对象，MappingData中包含了Wrapper。

2、tomcat类加载机制

![](images\Tomcat类加载机制.png)

前三个深色的类加载器，是JVM默认的类加载，后面浅色的则是tomcat的自定义的 类加载器，

CommonClassLoader加载的路径/common/*

CatalinaClassLoader加载的路径/server/*

ShardClassLoader加载的路径lib目录和WEB-INF/*

WebAppClassLoader和JasperLoader通常会存在多个实例，每一个Web应用对应一个WebAppClassLoader,

一个jsp文件对应一个JasperLoader。

- Common ClassLoader:Tomcat中最基本的类加载器，加载class可以被Tomcat容器本身以及各个Webapp访问
- catalina Loader：Tomcat容器私有的类加载器，加载路径中的class对于Webapp不可见；
- shared Loader：各个Webapp共享的类加载器，加载路径中的class对于所有Webapp可见，但是对于Tomcat容器不可见；
- WebappClassLoader：各个Webapp私有的类加载器，加载路径中的class只对当前Webapp可见
  

> CommonClassLoader能加载的类都可以被Catalina ClassLoader和SharedClassLoader使用，从而实现了公有类库的共用，而CatalinaClassLoader和Shared ClassLoader自己能加载的类则与对方相互隔离。
>
> WebAppClassLoader可以使用SharedClassLoader加载到的类，但各个WebAppClassLoader实例之间相互隔离。
>
> 而JasperLoader的加载范围仅仅是这个JSP文件所编译出来的那一个.Class文件，它出现的目的就是为了被丢弃：当Web容器检测到JSP文件被修改时，会替换掉目前的JasperLoader的实例，并通过再建立一个新的Jsp类加载器来实现JSP文件的HotSwap功能。

**那么问题来了tomcat是否破坏了双亲委派机制？**

双亲委派机制，要求从顶层classloader开始加载，当父类加载器加载不到class，就会让其子类加载器加载，到最后也加载不到，就会抛异常ClassNotFoundException。

很显然，tomcat加载并没有让父加载器开始加载，每个WebappClassLoader都是直接加载自己指定路径下的，不会传给父类加载器，所以说tomcat是破坏了双亲委派机制。

**在一个问题，如果tomcat的CommonLoader想要加载WebAppClassLoaer中的类，怎么办**

使用线程上下文类加载器实现，实现线程上下文加载器，可以让父类加载器请求子类加载器去完成类加载 的动作。

**假如我们自己写了一个java.lang.String的类，我们是否可以替换调JDK本身的类**

jvm具体规定了针对java.*开头的类，必须是有BootStrap类加载器来加载。所以就算是自定义一个ClassLoader，打破双亲委派机制，也是无法加载自定义java.lang.String。

还有就是因为默认加载的机制，从顶向下加载，Boostrap会在指定的路径下加载java.lang.String。

> ### 线程上下文类加载器
>
> 这个类加载器可以通过Thread的setContextClassLoader进行设置。如果创建线程还未设置，那么将会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过，那么这个类加载器默认就是应用程序类加载器。

3、Tomcat是个web容器，那么它还要解决什么问题？

1. 一个web容器可以部署多个应用，但是不同应用可能依赖同一个第三方类库的不同版本，所以不能要求同一个类库在一个服务器中只有一份，因此要保证每个应用程序的类库都是独立的，保证相互隔离。
2. 部署在同一服务器的同一类库的相同版本可以共享，否则服务器中有10个应用程序，就会加载10份相同的类库到虚拟机。
3. web容器也有自己依赖的类库，不能与应用程序的类库混淆。基于安全考虑，应该让容器的类库和程序的类库隔离开来。
4. web容器要支持jsp的修改，修改后的jsp是不会重新加载，每个jsp文件对应一个唯一的类加载器，当一个jsp文件修改了，就直接卸载这个jsp类加载器。重新创建类加载器，重新加载jsp文件


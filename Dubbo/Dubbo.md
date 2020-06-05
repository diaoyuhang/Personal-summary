# Dubbo

## RMI

JDK自带的能力。可与原生RMI互操作，基于TCP协议

```java
//提供接口，该接口需要继承Remote
public interface IMyService extends Remote {
    //这里需要将异常抛出
    public String toSay(String name) throws Exception;
}

//实现该接口
public class MyServiceImpl implements IMyService {
    @Override
    public String toSay(String name) throws Exception {
        System.out.println("远程服务接收到name:" + name);
        return "收到请求参数：" + name;
    }
}

//服务提供者
public class RmiProvider {
    public static void main(String[] args){
        try {
            //开启服务
            IMyService myService = (IMyService) UnicastRemoteObject.exportObject(new MyServiceImpl(), 9999);
            TestService testService = (TestService) UnicastRemoteObject.exportObject(new TestServiceImpl(), 9999);
            //开启注册中心
            Registry registry = LocateRegistry.createRegistry(6666);
            //将服务绑定到注册中心
            registry.bind("IMyService",myService);
            registry.bind("TestService",testService);
        } catch (RemoteException | AlreadyBoundException e) {
            e.printStackTrace();
        }
    }
}

//服务调用者
public class RmiConsumer {
    public static void main(String[] args) {
        try {
            //获得注册中心
            Registry registry = LocateRegistry.getRegistry(6666);
            //到注册中心中查找服务
            IMyService myServiceImpl = (IMyService) registry.lookup("IMyService");
            //调用远程服务
            String result1 = myServiceImpl.toSay("刁宇航");
            TestService testService= (TestService) registry.lookup("TestService");
            String result2 = testService.toSay("刁宇航");
            System.out.println(result1);
            System.out.println(result2);
        } catch (RemoteException | NotBoundException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Dubbo架构图

![](D:\0_LeargingSummary\Dubbo\images\图片1.png)

1. start:启动Spring后，会自动启动Dubbo的Provider
2. register:Provider启动后，会向注册中心Registry注册内容,注册的内容包括
   1. Provider的Ip，端口
   2. 对外提供的列表，哪些方法，哪些接口类
   3. Dubbo版本
   4. 访问Provider的版本
3. subscribe:订阅，当Consumer启动后，会向Registry中获取所有已注册服务的信息
4. notify:当Provider发生变化，自动由Registry向Consumer推送通知
5. invoke:Consumer调用Provider中的方法
   1. 同步请求：消耗性能，必须同步，需要接收到方法后的结果
6. count:Consumer和Provider默认每隔一段时间，向Monitor发送访问次数
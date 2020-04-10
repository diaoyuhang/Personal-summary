# ActiveMQ

## 默认Queue代码

### Producer

```java
import javax.jms.Connection;
import javax.jms.JMSException;
import javax.jms.MessageProducer;
import javax.jms.Queue;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnectionFactory;

public class Sender {
	public static void main(String[] args) throws Exception {
		// 1、建立工厂对象
		ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://localhost:61616");

		// 2、从工厂里拿一个连接
		Connection connection = connectionFactory.createConnection(ActiveMQConnectionFactory.DEFAULT_USER,
				ActiveMQConnectionFactory.DEFAULT_PASSWORD);
		connection.start();

		// 3、从连接中获取Session会话
		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		
		//从会话中获取目的地Destination(没有该队列就创建),消费者会从这个目的地获取消息
		Queue queue = session.createQueue("mq");
		//从会话中创建消息发送者
		MessageProducer producer = session.createProducer(queue);
        
        //设置不持久化，默认是DeliveryMode.PERSISTENT
		producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
        
		for(int i=0;i<20;i++) {
			TextMessage message = session.createTextMessage("消息"+i);
			producer.send(message);
			Thread.sleep(1000);
		}
		
		connection.close();
		System.out.println("exit");
	}
}

```

### Consumer

```java
import javax.jms.Connection;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageConsumer;
import javax.jms.Queue;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.activemq.ActiveMQConnectionFactory;

public class Receiver {

	public static void main(String[] args) throws Exception {
		ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
				ActiveMQConnectionFactory.DEFAULT_USER, ActiveMQConnectionFactory.DEFAULT_PASSWORD,
				"tcp://localhost:61616");

		Connection connection = connectionFactory.createConnection();

		connection.start();

		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		Queue queue = session.createQueue("mq");

		MessageConsumer messageConsumer = session.createConsumer(queue);

		while (true) {
            //这步是阻塞的，接收不到消息就一直停在这里
			TextMessage msg = (TextMessage) messageConsumer.receive();

			System.out.println(msg.getText());
		}
	}
}
```

## Active MQ的安全机制  

### web控制台安全  

修改conf目录下的jetty-realm.properties文件

```xml
# Defines users that can access the web (console, demo, etc.)
# username: password [,rolename ...]
admin: admin, admin
user: user, user
```

### 消息安全机制  

修改conf目录下的activemq.xml,在broker结点中添加

```java
<plugins>
			<simpleAuthenticationPlugin>
				<users>
    
    		<!-- 不同的账户有着不同的权限，管理员，发布者，消费者，游客 -->
    
					<authenticationUser username="admin" password="admin"
					groups="admins,publishers,consumers"/>
					<authenticationUser username="publisher" password="publisher"
					groups="publishers,consumers"/>
					<authenticationUser username="consumer" password="consumer"
					groups="consumers"/>
					<authenticationUser username="guest" password="guest"
					groups="guests"/>
				</users>
			</simpleAuthenticationPlugin>
		</plugins>
```

## JDBC持久化

在activemq.xml中broker中添加bean

```xml
<bean id="mysql-ds" class="org.apache.commons.dbcp.BasicDataSource" destroymethod="close">
		<property name="driverClassName" value="com.mysql.jdbc.Driver"/>
		<property name="url" value="jdbc:mysql://localhost/activemq?
		relaxAutoCommit=true"/>
		<property name="username" value="root"/>
		<property name="password" value="123456"/>
		<property name="maxActive" value="200"/>
		<property name="poolPreparedStatements" value="true"/>
	</bean>
```

修改persistenceAdapter 标签

```java
 <persistenceAdapter>
         <!-- <kahaDB directory="${activemq.data}/kahadb"/> -->
		<jdbcPersistenceAdapter dataSource="#mysql-ds" reateTablesOnStartup="true" />
 </persistenceAdapter>
```

在lib目录下添加依赖的jar包

> ### commons-dbcp commons-pool mysql-connector-java  

## 事务

开启事务

```java
// 3、从连接中获取Session会话
		Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
```

当开启事务后，发送数据就需要手动提交

```java
TextMessage message = session.createTextMessage("消息"+i);
producer.send(message);
session.commit();
```

当发生错误时，可以回滚数据

```java
try {
    TextMessage message = session.createTextMessage("消息"+i);
    producer.send(message);
    session.commit();
    Thread.sleep(1000);
} catch (Exception e) {
    e.printStackTrace();
    session.rollback();
}
```

## 确认JMS消息机制

只有在被确认之后，才认为已经被成功地消费了。
消息的成功消费通常包含三个阶段：客户接收消息、客户处理消息和消息被确认。
在事务性会话中，当一个事务被提交的时候，确认自动发生。
在非事务性会话中，消息何时被确认取决于创建会话时的应答模式（acknowledgement mode）。该
参数有以下三个可选值：
Session.AUTO_ACKNOWLEDGE。当客户成功的从receive方法返回的时候，或者从
MessageListener.onMessage方法成功返回的时候，会话自动确认客户收到的消息。
Session.CLIENT_ACKNOWLEDGE。客户通过消息的acknowledge方法确认消息。需要注意的是，
在这种模式中，确认是在会话层上进行：确认一个被消费的消息将自动确认所有已被会话消费的消
息。例如，如果一个消息消费者消费了10个消息，然后确认第5个消息，那么所有10个消息都被确
认。
Session.DUPS_ACKNOWLEDGE。该选择只是会话迟钝的确认消息的提交。如果JMS Provider失
败，那么可能会导致一些重复的消息。如果是重复的消息，那么JMS Provider必须把消息头的
JMSRedelivered字段设置为true。  

```java
//发送端，接收端要统一设置
//设置成手动发送确认消息，第一个参数不能设置成开启事务，不然会被修改成SESSION_TRANSACTED
Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
```

```java
//接收端，接收到消息后返回确认
//发送一次确认，可以确认接收到多条消息
textMessage.acknowledge();
```

>  ### 注：如果开启事务，发送确认消息后，依然需要发送session.commit(),不然，消息依然是未消费的；

## 消息优先级

设置activemq.xml文件

```xml
<!-- 指定队列，开启优先级-->
<policyEntry queue="queue1" prioritizedMessages="true" />
```

```java
//1-9数字越大，优先级越高
producer.setPriority(1);
//获得调用producer.send(message, deliveryMode, priority, timeToLive);
```

## 监听器

使用监听器来处理接收数据，实现接口javax.jms.MessageListener

```java
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

public class MyListener implements MessageListener {

	@Override
	public void onMessage(Message message) {
		TextMessage textMessage = (TextMessage) message;
		try {
			System.out.println(textMessage.getText());
		} catch (JMSException e) {
			e.printStackTrace();
		}
	}
}
```

## 消息类型

### Object

> ### 自定义一个类型实现接口java.io.Serializable

在接收端中连接工厂创建连接之前添加需要信任的类型的包路径

```java
//添加信任类型
connectionFactory.setTrustedPackages(Arrays.asList(Girl.class.getPackage().getName()));

Girl girl = new Girl("zhou",23);		
ObjectMessage girlMessage = session.createObjectMessage(girl);
producer.send(girlMessage);
```

### ByteMessage

```JAVA
BytesMessage bytesMessage = session.createBytesMessage();
bytesMessage.writeUTF("字节消息类型");

producer.send(bytesMessage);
```

## MapMessage

```java
MapMessage mapMessage = session.createMapMessage();
mapMessage.setString("MapMessage","map消息类型");
producer.send(mapMessage);
```

## 超时

设置消息超时时间

```java
producer.setTimeToLive
```

### 死信队列

消息超时后默认会进入ActiveMQ.DLQ，

获取死信队列

```java
Queue DLQ = session.createQueue("ActiveMQ.DLQ");
MessageConsumer messageConsumer2 = session.createConsumer(DLQ);
messageConsumer2.setMessageListener(new MyListener());
```

### 修改死信队列名称

```java
<policyEntry queue="mq" prioritizedMessages="true" >
    <deadLetterStrategy> 
<!-- 设置死信队列前缀名称-->
    <individualDeadLetterStrategy   queuePrefix="died." useQueueForQueueMessages="true" /> 

    </deadLetterStrategy> 
    </policyEntry>
```

> ### 注：不持久化数据过期后是不会进入死信队列

```xml
<!-- 让非持久化数据过期也进入死信队列-->
<individualDeadLetterStrategy   queuePrefix="died." useQueueForQueueMessages="true"  processNonPersistent="true" />
```

```java
<!--设置过期后不进入死信队列 -->
<individualDeadLetterStrategy   processExpired="false"  /> 
```

## 独占消费者

```java
Queue queue = session.createQueue("mq?consumer.exclusive=true");
```

## 消息过滤

```java
//消息发送
		MapMessage msg1 = session.createMapMessage();
		msg1.setString("name", "qiqi");
		msg1.setString("age", "18");
		
		msg1.setStringProperty("name", "qiqi");
		msg1.setIntProperty("age", 18);
		MapMessage msg2 = session.createMapMessage();
		msg2.setString("name", "lucy");
		msg2.setString("age", "18");
		msg2.setStringProperty("name", "lucy");
		msg2.setIntProperty("age", 18);
		MapMessage msg3 = session.createMapMessage();
		msg3.setString("name", "qianqian");
		msg3.setString("age", "17");
		msg3.setStringProperty("name", "qianqian");
		msg3.setIntProperty("age", 17);
```

```java
//消息接收
String selector1 = "age > 17";
String selector2 = "name = 'lucy'";
MessageConsumer consumer = session.createConsumer(queue,selector2);
```

## 延迟消息投递

在配置文件中开启延迟和调度 schedulerSupport="true"

```xml
    <broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost" dataDirectory="${activemq.data}" schedulerSupport="true">
```

设置消息延迟投递

```java
//设置该消息延迟投递
mapMessage.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, 1000*10);
```

带间隔的重复发送

```java
//带间隔重复发送
long delay=10*1000;
long period=2*1000;//间隔2秒
int repeat = 9;//重复投递9次，不包括第一次
mapMessage.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, delay);
mapMessage.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_PERIOD, period);
mapMessage.setIntProperty(ScheduledMessage.AMQ_SCHEDULED_REPEAT, repeat);
producer.send(mapMessage);
```


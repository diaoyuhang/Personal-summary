# SpringCloud

> ## 技术的选型
>
> dubbo：zookeeper+dubbo+springmvc/springboot
> 通信方式：rpc
> 注册中心：zookeeper
> 配置中心：diamond（淘宝开发）
>
> spring cloud：spring+Netflix
> 通信方式：http restful
> 注册中心：eureka				
> 配置中心：config
> 断路器：hystrix
> 网关：zuul
> 分布式追踪系统：sleuth+zipkin
>
> |            | **dubbo**                               | **spring cloud** |           |
> | ---------- | --------------------------------------- | ---------------- | --------- |
> | 背景       | 国内影响大                              | 国外影响大       | 平手      |
> | 社区活跃度 | 低                                      | 高               | cloud胜出 |
> | 架构完整度 | 不完善（dubbo有些不提供，需要用第三方） | 比较完善         | cloud胜出 |
> | 学习成本   | dubbo需要配套学习                       | 无缝spring       | cloud胜出 |

## 一、Eureka

> ### 作用：服务管理，维护注册表，心跳检测机制

### 注册中心项目搭建

> 服务端：搭建Eureka单点或是高可用集群
>
> 客户端：初始化服务，注册到Eureka

#### 1) 服务端配置搭建

- pom.xml文件引入Eureka服务

  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
  </dependency>
  
  <!-- 安全认证，可选的-->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
  </dependency>
  ```

- application.yml对Eureka配置

```yml
##单点配置
#应用名称及验证账号
spring: 
  application: 
    name: eureka
#这里启动安全认证登录，需要引入上面安全认证依赖
  security: 
    user: 
      name: root
      password: root
      
spring:
  profiles: 7900
server: 
  port: 7900
eureka:
  instance:
    instance-id: eureka
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:                      
      defaultZone: http://root:root@localhost:7900/eureka/
```

```yaml
##集群的配置
#应用名称及验证账号
spring: 
  application: 
    name: eureka
    
  security: 
    user: 
      name: root
      password: root
---
spring:
  profiles: 7901
server: 
  port: 7901
eureka:
  instance:
    hostname: eureka-7901  
  client:
         #设置服务注册中心的URL
    service-url:                      
      defaultZone: http://root:root@localhost:7902/eureka/,http://root:root@localhost:7903/eureka/,http://root:root@localhost:7901/eureka/
---    
spring:
  profiles: 7902
server: 
  port: 7902
eureka:
  instance:
    hostname: eureka-7902  
  client:
       #设置服务注册中心的URL
    service-url:                      
      defaultZone: http://root:root@localhost:7902/eureka/,http://root:root@localhost:7903/eureka/,http://root:root@localhost:7901/eureka/
---
spring:
  profiles: 7903
server: 
  port: 7903
eureka:
  instance:
    hostname: eureka-7903
  client:
       #设置服务注册中心的URL
    service-url:                      
      defaultZone: http://root:root@localhost:7902/eureka/,http://root:root@localhost:7903/eureka/,http://root:root@localhost:7901/eureka/

```

- 启动类添加注解

```java
@EnableEurekaServer
```

- 开启安全认证

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		// 关闭csrf
		http.csrf().disable(); 
		// 开启认证
		http.authorizeRequests().anyRequest().authenticated().and().httpBasic(); 
	}
}
```



#### 2) 客户端配置搭建

- pom.xml引入Eureka客户端

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

- application.yml配置

```yaml
#应用名称及验证账号
spring:
  application:
    name: api-passenger
    admin:
      enabled: true
eureka:
  client:
    serviceUrl:
      defaultZone: http://root:root@localhost:7900/eureka/
  instance:
    instance-id: api-passenger

#服务端口
server: 
  port: 9001

logging:
  level:
    org.springframework: DEBUG
```

- 启动类上添加注解

```java
@EnableEurekaClient
```

#### 3) 流程图

- Eureka客户端生命周期工作流程图

![](D:\0_LeargingSummary\SpringCloud\images\Eureka Client工作流程图.jpg)

#### 4）Eureka总结

Eureka也存在缺陷。
由于集群间的同步复制是通过HTTP的方式进行，基于网络的不可靠性，集群中的Eureka Server间的注册表信息难免存在不同步的时间节点，不满足CAP中的C(数据一致性)。

## 二、服务调用

> ### spring cloud提供ribbon和feign进行服务间的调用
>
> - #### 客户端负载均衡  和 服务端负载均衡的区别在于服务端地址列表的存储位置
>
> - #### ribbon和feign都是客户端负载均衡，默认是以RandonRobin方法轮询选择服务器
>
> ### Restful
>
> - #### Resource Representational State Transfer 资源表现层状态的转移
>
> - #### 通过Http的动作，来完成的Restful
>
> - #### 客户端访问服务的过程中必然涉及数据和状态的转化。如果客户端想要操作服务端资源，必须通过某种手段，让服务器端资源发生“状态转移”。而这种转化是建立在表现层之上的，所以被称为“表现层状态转移”。客户端通过使用HTTP协议中的四个动词来实现上述操作，它们分别是：获取资源的GET、新建或更新资源的POST、更新资源的PUT和删除资源的DELETE。
>
> - #### Http和Restful的区别，就是Restful使用上述的http的四个动作实现

### Ribbon

- 将RestTemplate组件加入到容器中

```java
@Bean
@LoadBalanced//实现客户端的负载均衡
public RestTemplate restTemplate() {
	return new RestTemplate();
}
```

> RestTemplate是Spring提供的同步HTTP网络客户端接口，它可以简化客户端与HTTP服务器之间的交互，并且它强制使用RESTful风格。它会处理HTTP连接和关闭，只需要使用者提供服务器的地址(URL)和模板参数。

- RestTemplate的使用

```java
@Autowired
private RestTemplate restTemplate;

String serviceName = "service-sms";
String url = "http://"+serviceName+"/send/alisms-template";
//ribbon调用
		ResponseEntity<ResponseResult> resultEntity = restTemplate.postForEntity(url, smsSendRequest, ResponseResult.class);
		ResponseResult result = resultEntity.getBody();
```

### Feign

![](D:\0_LeargingSummary\SpringCloud\images\图片1.png)

- 在pom.xml中引入依赖

```xml
<!-- 引入feign依赖 ，用来实现接口伪装 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

- 启动类上加：
  @EnableFeignClients,就像是一个开关，只有使用了该注解，OpenFeign相关的组件和配置机制才会生效。

- 定义接口

  ```java
  @FeignClient(name = "service-sms")
  public interface SmsClient {
  	/**
  	 * 按照短信模板发送验证码
  	 * @param smsSendRequest
  	 * @return
  	 */
  	@RequestMapping(value="/send/alisms-template", method = RequestMethod.POST)
  	public ResponseResult sendSms(@RequestBody SmsSendRequest smsSendRequest);
  }
  ```

- 引入feign接口

```java
	@Autowired
	private SmsClient smsClient;
```

- 通过定义的接口，直接发送请求

```java
//feign调用
ResponseResult result = smsClient.sendSms(smsSendRequest);
```

- @EnableFeignClients作用

  > 1、引入FeignClientsRegistrar
  >
  > 2、指定扫描FeignClient的包信息，就是指定FeignClient接口类所在的包名
  >
  > 3、指定FeignClient接口类的自定义配置类

## 三、熔断降级hystrix

> #### 解决的问题：
>
> #### 在分布式系统中，一个服务依赖多个服务，可能存在某个服务调用失败，比如超时，异常，如何保证在某些服务故障时，对整个服务没有影响。
>
> #### 熔断后序：
>
> #### 出现错误后用fallback返回错误的处理信息，或者兜底数据。

### 熔断处理的两种方式

> ### 没有引入Feign依赖

- 在pom.xml中引入依赖

```xml
<!-- 引入hystrix(熔断)依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

- 在启动类上加入注释

```java
@EnableCircuitBreaker // 开启熔断
```

- api方法,在方法上加上@HystrixCommand注解，其中的fallbackMethod的值要与方法名一致

```java
@HystrixCommand(fallbackMethod = "sendFail")
@PostMapping("/send")
public ResponseResult send(@RequestBody ShortMsgRequest shortMsgRequest) {
    String phoneNumber = shortMsgRequest.getPhoneNumber();
    System.out.println(phoneNumber);
    // 临时的验证码
    String code = "123456";
    ResponseResult result = shortMsgService.send(phoneNumber, code);
    System.out.println(result.getCode());
    return ResponseResult.success(null);
}

private ResponseResult sendFail(ShortMsgRequest shortMsgRequest) {
    return ResponseResult.fail(-1, "熔断");
}
```



> ### OpenFeign是自带Hystrix,但是默认没有打开

- 开启feignhystrix

```yaml
feign:
  hystrix:
    enabled: true
```

- 新增实现类，实现调用其他服务的接口

```java
@FeignClient(name = "service-sms",fallback = SmsClientFallback.class)
public interface SmsClient {
	/**
	 * 按照短信模板发送验证码
	 * @param smsSendRequest
	 * @return
	 */
	@RequestMapping(value="/send/alisms-template", method = RequestMethod.POST)
	public ResponseResult sendSms(@RequestBody SmsSendRequest smsSendRequest);
}
```

```java
@Component
public class SmsClientFallback implements SmsClient {


	@Override
	public ResponseResult sendSms(SmsSendRequest smsSendRequest) {
		System.out.println("不好意思，我熔断了");		
		return null;
	}
}
```

### 熔断hystrix--告警通知

> #### 实现的方式有很多种，这里用的redis

```java
@Component
public class SmsClientFallback implements SmsClient {

	@Autowired
	private StringRedisTemplate redisTemplate;

	@Override
	public ResponseResult sendSms(SmsSendRequest smsSendRequest) {
		System.out.println("不好意思，我熔断了");

		String key = "service-sms";
		String message = redisTemplate.opsForValue().get(key);
		if (!StringUtils.isNotBlank(message)) {
			System.out.println("通知熔断的逻辑。。。");

			redisTemplate.opsForValue().set(key, "发生熔断",20,TimeUnit.SECONDS);
		}else {
			System.out.println("存在该通知，稍后再通知");
		}
		return null;
	}
}
```

## 四、网关

> #### 解决的问题
>
> #### （1）客户端会多次请求不同微服务，增加了客户端复杂性
>
> #### （2）认证复杂，每个服务都需要独立认证
>
> #### （3）难以重构，多个服务可能将会合并成一个或拆分成多个
>
> #### （4） 安全性，微服务不需要向外暴露，通过物理网络隔离
>
> #### 功能
>
> #### 统一接入：智能路由，AB测试、灰度测试，日志埋点
>
> #### 流量监控：限流处理
>
> #### 安全防护：鉴权，网络物理隔离
>
> 
>
> #### 注：需要引入zuul组件

#### 网关基本使用

- 初始化项目，引入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

- 在启动类上添加注释

```java
@EnableZuulProxy//标识该项目为网关项目
```

- 配置

```yaml
server:
  port: 9000
  
spring:
  application:
    name: online-taxi-gateway
    admin:
      enabled: true
      
eureka:
  client:
    service-url:
      defaultZone: http://root:root@localhost:7900/eureka/
  instance:
    instance-id: online-taxi-gateway
```

- 基本使用

默认路由：
原服务地址：
127.0.0.1:8002/service-sms-gateway-test/hello
通过网关的地址：
127.0.0.1:9000/service-sms/service-sms-gateway-test/hello
不同：原来地址变成网关地址+eureka中服务名。

- 自定义路由

```yaml
zuul:
  routes:
  #服务名称: 匹配的路径
    service-sms: /gate-way/sms/**
    api-passenger: /gate-way/api/**
```

127.0.0.1:9000/gate-way/sms/service-sms-gateway-test/hello
等价
127.0.0.1:8002/service-sms-gateway-test/hello

- 忽略服务，再配合内网隔离，就做到了，只暴露网关。真实拿不到内网IP

```yaml
zuul:
   ignored-services:
   - api-passenger
   - service-sms
#或者
zuul:
  routes:
  #忽略正则
     ignored-patterns: /*-passenger/**
```

#### 过滤器

> #### 上下文：RequestContext，贯穿整个过滤器。
>
> #### order越小越先执行
>
> 
>
> #### 1、继承ZuulFilter
>
> #### 2、加上@Component，将自己定义的 过滤器加入到容器中
>
> #### 3、实现ZuulFilter中具体方法

![](D:\0_LeargingSummary\SpringCloud\images\网关过滤器流程.png)

- 登录鉴权

```java
@Component
public class AuthFilter extends ZuulFilter {

    //只有该方法返回true时，才会执行下方的run方法
	@Override
	public boolean shouldFilter() {
		RequestContext context = RequestContext.getCurrentContext();
		HttpServletRequest request = context.getRequest();

		String uri = request.getRequestURI();
		System.out.println("uri:" + uri);

		// 只有此接口/api-passenger/api-passenger-gateway-test/hello才被拦截
		String checkUri = "/api-passenger/api-passenger-gateway-test/hello";
		if (checkUri.equalsIgnoreCase(uri)) {
			return true;
		}
		return false;
	}

	@Override
	public Object run() throws ZuulException {
		System.out.println("拦截");

		RequestContext context = RequestContext.getCurrentContext();
		HttpServletRequest request = context.getRequest();

		String token = request.getHeader("Authorization");

		if (StringUtils.isNotBlank(token) && "1234".equals(token)) {
			System.out.println("校验通过");
		} else {
			context.setSendZuulResponse(false);
			context.setResponseStatusCode(HttpStatus.SC_UNAUTHORIZED);
		}

		return null;
	}

	@Override
	public String filterType() {
		return FilterConstants.PRE_TYPE;
	}

	@Override
	public int filterOrder() {
		return 4;
	}
}
```

## 五、链路追踪

> #### 链路追踪通过流水号跟踪链路
>
> #### 解决错综复杂的服务调用中链路的查看。排查慢服务

在服务中的pom.xml中添加依赖sleuth

```xml
<!-- 链路追踪 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

添加依赖后的重启项目，发起请求，日志中会打印如下的日志

> ####  [online-taxi-gateway,b2c9be544be51e48,b2c9be544be51e48,false]
> #### [服务名称，traceId（一条请求调用链中 唯一ID），spanID（基本的工作单元，获取数据等），是否让zipkin收集和展示此信息]



zipkin和sleuth中结合可以提供可视化web界面分析调用链路耗时情况

安装zipkin

```sh
curl -sSL https://zipkin.io/quickstart.sh | bash -s
java -jar zipkin.jar
```

>sleuth收集跟踪信息通过http请求发送给zipkin server，zipkin将跟踪信息存储，以及提供RESTful API接口，zipkin ui通过调用api进行数据展示。
>
>默认内存存储，可以用mysql，ES等存储



在每个服务中添加zipkin依赖，zipkin中已经包含了sleuth

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

配置sleuth收集信息发送给zipkin

```yaml
spring:
#sleuth收集信息后发送给zipkin
  zipkin:
    base-url: http://localhost:9411/
```

配置采集信息的比例

```yaml
spring:
#采样比例1，百分之八采样
  sleuth:
    sampler:
      rate: 1 
```

## 六、配置中心

> #### 在github中新建仓库，并在仓库中存放配置文件

添加配置中心依赖

```yaml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

配置文件

```yaml
spring:
  cloud:
    config:
      server:
        git:
        #https://github.com/diaoyuhang/online-taxi-config-server.git
          uri: https://github.com/diaoyuhang/online-taxi-config-server
          username: 
          password:
```

在启动类上添加注解

```java
@EnableConfigServer
```

启动项目后可以直接访问到github中存储的文件

```http
更改后缀名可以显示不同的格式
http://localhost:6001/service-verification-code.yml
```

> #### 配置中心-client的调用
>
> #### 注：从配置中心加载的配置会覆盖当前配置文件中配置



- 在pom.xml文件中添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

- 要将配置文件的名称改为bootstrap.xml,该文件会优先加载

```yaml
spring: 
  application: 
    name: service-verification-code
  cloud:
    config:
      discovery:
        enabled: true
        service-id: config-server
      profile: qa
```

> #### 消息总线

pom.xml添加依赖

```xml
<!-- 服务监控开启refresh 端口 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!-- 总线 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-bus-amqp</artifactId>
		</dependency>
```

添加配置

```yaml
management:
  endpoints:
    web:
      exposure:
      #yml加双引号，properties不用加
        include: "*"
```

在类上添加@RefreshScope

动态刷新

```http
http://localhost:8011/actuator/bus-refresh
```


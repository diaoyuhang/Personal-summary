# Spring注解驱动开发

## 给容器中注册bean

### @Bean注解

原始xml示例

```xml
    <bean id="person" class="annotation.com.diao.bean.Person">
        <property name="name" value="diao"/>
        <property name="age" value="24"/>
    </bean>
```

```java
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        Object person = context.getBean("person");
        System.out.println(person);
```



使用注解

```java
//使用一个配置类代替xml配置文件
@Configuration//告诉spring这是一个配置类
public class Config {
	
    //注册的id名默认是方法名，或者可以在@Bean中添加:@Bean(name = "person")
    @Bean
    public Person person(){
        return new Person("diao",24);
    }
}
```

```java
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
        Object person = context.getBean("person");
        System.out.println(person);
```

###  @ComponentScan

xml示例

```xml
    <context:component-scan base-package="annotation.com.diao" use-default-filters="false">
        <context:include-filter type="" expression=""/>
        <context:exclude-filter type="" expression=""/>
    </context:component-scan>
```

使用注解

```java
@Configuration//告诉spring这是一个配置类
@ComponentScan(value = {"annotation.com.diao"},//扫描指定的路径
//        excludeFilters = {//排除过滤掉
//                @Filter(type = FilterType.ANNOTATION, value = Repository.class)//FilterType.ANNOTATION基于注解的过滤
//        },
        useDefaultFilters = false,//不使用默认的扫描包含策略，使用includeFilters需要将该项设置成false
        includeFilters = {//包含指定类型
//                @Filter(type = FilterType.ASSIGNABLE_TYPE, value = PersonService.class),// FilterType.ASSIGNABLE_TYPE指定类型
                @Filter(type = FilterType.CUSTOM, value = MyFilter.class)//FilterType.CUSTOM自定义规则
        }
)
public class Config {

    @Bean(name = "person")
    public Person person() {
        return new Person("diao", 24);
    }
}

//自定义FilterType
public class MyFilter implements TypeFilter {
    /**
     * metadataReader 能够读取到当前正在扫描的类的信息
     * metadataReaderFactory 获得其他的metadataReader
     */
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        //获得当前扫描的类的注解信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        Set<String> annotationTypes = annotationMetadata.getAnnotationTypes();

        //当前扫描的类的类信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        //当前扫描的类的路径
        Resource resource = metadataReader.getResource();

        String className = classMetadata.getClassName();
        System.out.println("--->"+className);
        if(className.contains("Person")){
            return true;
        }
        return false;
    }
}
```

### @Scope

设置bean的作用域

```java
	 /* @see ConfigurableBeanFactory#SCOPE_PROTOTYPE
	 * @see ConfigurableBeanFactory#SCOPE_SINGLETON
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_SESSION
	 */
@Scope
@Bean(name = "person")
public Person person() {
    return new Person("diao", 24);
}
```

### @Lazy

用于设置单例bean懒加载

```java
@Lazy
@Bean(name = "person")
public Person person() {
    return new Person("diao", 24);
}
```

### @Conditioinal

满足给定的条件决定才注入bean

```java
//当写在方法上，就只作用在该方法，写在类上作用整个类
@Conditional(value = {MyConditional.class})
@Bean(name = "person")
public Person person() {
    return new Person("diao", 24);
}
```

```java
public class MyConditional implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment environment = context.getEnvironment();
        String name = environment.getProperty("os.name");
        System.out.println(name);
        //如果当前操作系统是Windows，条件成立
        if(name.contains("Windows")){
            return true;
        }
        return false;
    }
}
```

### @Import

```java
@Configuration
@Import(value={Red.class, MyImportSelector.class,MyImportBeanDefinitionRegistor.class})//导入添加指定bean
public class Config {
}

public class MyImportSelector implements ImportSelector {
    //返回的就是需要注入到容器中的组件的全类名
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"annotation.com.diao.bean.Black"};
    }
}

//向BeanDefinitionRegistry直接注册bean
public class MyImportBeanDefinitionRegistor implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Blue.class);
        registry.registerBeanDefinition("blue", rootBeanDefinition);
    }
}
```

### @FactoryBean

用于生产指定bean，根据id获得的就是指定bean实例，在id前加上&就可以获得FactoryBean

```java
@Bean
public ColorFactoryBean colorFactoryBean(){
    return new ColorFactoryBean();
}
```

```java
public class ColorFactoryBean implements FactoryBean {
    @Override
    public Object getObject() throws Exception {
        return new Color();
    }

    @Override
    public Class<?> getObjectType() {
        return Color.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

### Bean生命周期

bean创建---初始化----销毁的过程

容器管理bean的生命周期；

我们可以自定义初始化和销毁方法；容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法

1. 通过@Bean指定init-method和destory-method方法

   ```java
   @Bean(name = "person",initMethod = "init",destroyMethod = "destory")
   public Person person() {
       return new Person("diao", 24);
   }
   
   public class Person {
       private String name;
       private int age;
   
       public Person(String name, int age) {
           this.name = name;
           this.age = age;
       }
       
       public void init(){
           System.out.println("person...init..");
       }
   
       public void destory(){
           System.out.println("person...destory");
       }
   }
   ```

2. 通过让bean实现InitializingBean，DisposableBean接口

   ```java
   public class Blue implements InitializingBean,DisposableBean {
       public Blue() {
           System.out.println("blue..constructor");
       }
   
       @Override
       public void destroy() throws Exception {
           System.out.println("Blue....destory");
       }
   
       @Override
       public void afterPropertiesSet() throws Exception {
           System.out.println("Blue...init");
       }
   }
   ```

3. 使用JSR250的注解：@PostConstruct，@PreDestroy

   ```java
   public class Black {
   
       @PostConstruct
       public void init(){
           System.out.println("Black...init...");
       }
   
       @PreDestroy
       public void destory(){
           System.out.println("Black...destory");
       }
   }
   ```

4. BeanPostProcessor：bean的后置处理器

   ```java
   public class MyBeanPostProcessor implements BeanPostProcessor {
       //在初始化之前工作
       @Override
       public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
           System.out.println("postProcessBeforeInitialization"+bean);
           return bean;
       }
   
       //在初始化之后工作
       @Override
       public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
           System.out.println("postProcessAfterInitialization:"+bean);
           return bean;
       }
   }
   ```

   BeanPostProcessor源码执行流程：

   发生在AbstractApplicationContext的refresh()方法中的finishBeanFactoryInitialization(beanFactory)阶段的；该阶段是实例化所有剩下的非拦截在的单例bean

   ```java
   //AbstractApplicationContext
   beanFactory.preInstantiateSingletons();
   
   //DefaultListableBeanFactory
   preInstantiateSingletons();
   
   //AbstractBeanFactory
    getBean(String name)
    doGetBean(name, null, null, false)
    
    //DefaultSingletonBeanRegistry
    getSingleton(String beanName, ObjectFactory<?> singletonFactory)
        
   //AbstractBeanFactory
        getObject();
   
   //AbstractAutowireCapableBeanFactory
    doCreateBean(beanName, mbdToUse, args);
   initializeBean(beanName, exposedObject, mbd)
       
   //在initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd)方法中
       if (mbd == null || !mbd.isSynthetic()) {
           //beanpostprocessor的前置处理
   			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
   		}
   
   		try {
               //这里执行bean的初始化方法
   			invokeInitMethods(beanName, wrappedBean, mbd);
   		}
   		catch (Throwable ex) {
   			throw new BeanCreationException(
   					(mbd != null ? mbd.getResourceDescription() : null),
   					beanName, "Invocation of init method failed", ex);
   		}
   
   		if (mbd == null || !mbd.isSynthetic()) {
               //beanpostprocessor的后置处理处理
   			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
   		}
   ```

   **ApplicationContextAwareProcessor**后置处理器是bean生命周期的第一个阶段，经过一系列的xxxAware把bean需要的信息调用setxxx给bean

   ```java
   /*全套的初始化方法及其标准顺序：
    *The full set of initialization methods and their standard order is:
    * <ol>
    
    1、任何实现了EnvironmentAware、EmbeddedValueResolverAware、ResourceLoaderAware、ApplicationEventPublisherAware、MessageSourceAware、ApplicationContextAware接口的bean，都会经过一系列的xxxAware把bean需要的信息调用setxxx给bean
    * <li>BeanNameAware's {@code setBeanName}
    * <li>BeanClassLoaderAware's {@code setBeanClassLoader}
    * <li>BeanFactoryAware's {@code setBeanFactory}
    * <li>EnvironmentAware's {@code setEnvironment}
    * <li>EmbeddedValueResolverAware's {@code setEmbeddedValueResolver}
    * <li>ResourceLoaderAware's {@code setResourceLoader}
   */
   @Override
   public Object postProcessBeforeInitialization(final Object bean, String beanName) throws BeansException {
      AccessControlContext acc = null;
   
      if (System.getSecurityManager() != null &&
            (bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
                  bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
                  bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)) {
         acc = this.applicationContext.getBeanFactory().getAccessControlContext();
      }
   
      if (acc != null) {
         AccessController.doPrivileged(new PrivilegedAction<Object>() {
            @Override
            public Object run() {
               invokeAwareInterfaces(bean);
               return null;
            }
         }, acc);
      }
      else {
         invokeAwareInterfaces(bean);
      }
   
      return bean;
   }
   
   private void invokeAwareInterfaces(Object bean) {
      if (bean instanceof Aware) {
         if (bean instanceof EnvironmentAware) {
            ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
         }
         if (bean instanceof EmbeddedValueResolverAware) {
            ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
         }
         if (bean instanceof ResourceLoaderAware) {
            ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
         }
         if (bean instanceof ApplicationEventPublisherAware) {
            ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
         }
         if (bean instanceof MessageSourceAware) {
            ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
         }
         if (bean instanceof ApplicationContextAware) {
            ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
         }
      }
   }
   ```

## 自动装配

### @Value

可从环境中取出配置信息

```java
@Value("${proper.name}")
private String name;

@Value("${proper.age}")
private String age;
```

### @PropertySource

加载配置文件,基于xml

```xml
<context:property-placeholder location="classpath:proper.properties"></context:property-placeholder>
```

基于注解,加载配置类上

```java
@PropertySource(value={"classpath:proper.properties"})
```

### @Autowired

```java
//获取指定id的实例
@Qualifier(value = "personService01")
//指定为非必须的，默认是按照类型注入，如果有多个同类型的实例，就按照名称注入
@Autowired(required = false)
private PersonService personService;
```

能够标注的位置：构造器，方法，参数，属性，这些都从容器中获取

1. 标注在方法上,不加@Autowired，也能实现自动装配

   ```java
   @Autowired
   @Bean
   public ColorFactoryBean colorFactoryBean(PersonService personService){
       System.out.println(personService);
       return new ColorFactoryBean();
   }
   ```

### @Primary

在配合@Autowired时，优先注入被该注释标识的bean实例，如果有@Qualifier，依然按照@Qualifier的要求注入

```java
@Primary
@Bean
public PersonService personService01(){
    return new PersonService() ;
}
```

### @Resource

JSR250规范，能够和@Autowired一样实现自动装配的功能，默认按照组件名，但是不支持@Primary和required功能

```jav
@Resource(name = "personService01")
private PersonService personService;
```

### @Inject

JSR330规范，需要额外导入javax.inject的包，不支持没有required

### @Profile

用于加载指定的环境

Spring为我们提供的可以根据当前环境，动态的激活和切换一系列组件的功能；

 * 1）、加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中。默认是default环境
 * 2）、写在配置类上，只有是指定的环境的时候，整个配置类里面的所有配置才能开始生效
 * 3）、没有标注环境标识的bean在，任何环境下都是加载的；

```java
@Configuration
@PropertySource(value = {"classpath:proper.properties"})
public class ConfigOfProfile implements EmbeddedValueResolverAware{
    
    @Value("${db.username}")
    private String name;

    private String driver;

    private StringValueResolver resolver;

    @Profile(value = {"test"})//指定为测试环境
    @Bean
    public DataSource dataSource01(@Value("${db.password}") String password){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUsername(name);
        dataSource.setPassword(password);
        dataSource.setDriverClassName(driver);
        dataSource.setUrl("jdbc:mysql://localhost:3306/crud?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true&useSSL=false");
        return dataSource;
    }

    @Profile(value = {"dev"})//指定为开发环境
    @Bean
    public DataSource dataSource03(@Value("${db.password}") String password){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUsername(name);
        dataSource.setPassword(password);
        dataSource.setDriverClassName(driver);
        dataSource.setUrl("jdbc:mysql://localhost:3306/crud?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true&useSSL=false");
        return dataSource;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        driver=resolver.resolveStringValue("${db.driver}");
    }
}
```

## AOP

### @EnableAspectJAutoProxy

开启aop注解

```java
@Configuration
@EnableAspectJAutoProxy //开启apo注解
public class ConfigOfAop {

    @Bean
    public MathCalculator mathCalculator(){
        return new MathCalculator();
    }

    @Bean
    public LogAspects logAspects(){
        return new LogAspects();
    }
}
```

原始xml形式

```xml
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

### @Aspect

声明该类为切面类

```java
@Aspect//向spring表示该类是切面类
public class LogAspects {}
```

### @Pointcut

声明切入点

```java
//抽取出公共切入点
@Pointcut("execution(public int annotation.com.diao.aop.MathCalculator.*(..))")
public void pointCut(){}
```

### @Before

前置通知

```java
//JoinPoint能够获取到切入点的相关信息
@Before(value = "pointCut()")
public void before(JoinPoint joinPoint){
    System.out.println(joinPoint.getSignature().getName()+" starting");
}
```

### @After

后置通知

```java
@After(value = "pointCut()")
public void after(JoinPoint joinPoint){
    System.out.println(joinPoint.getSignature().getName()+" ending");
}
```

### @AfterReturning

返回通知

```java
//这里需要标识哪个参数是用来接收返回值的
@AfterReturning(value = "pointCut()",returning = "result")
public void afterReturning(JoinPoint joinPoint,Object result){
    System.out.println(joinPoint.getSignature().getName()+" ending,result="+result);
}
```

### @AfterThrowing

异常通知

```java
@AfterThrowing(value = "pointCut()",throwing = "e")
public void afterThrowing(JoinPoint joinPoint,Exception e){
    System.out.println(joinPoint.getSignature().getName()+" ending,exception="+e);
}
```

### @Around

环绕通知

```java
@Around(value = "pointCut()")
public Object around(ProceedingJoinPoint joinPoint){
    Object[] args = joinPoint.getArgs();
    Object proceed=null;
    try {
        System.out.println("环绕通知前");
        proceed   = joinPoint.proceed(args);
        System.out.println("环绕通知后");


    } catch (Throwable throwable) {
        throwable.printStackTrace();
    }
    return proceed;
}
```

### 源码

```
//从开关注解进入
@EnableAspectJAutoProxy
->@Import(AspectJAutoProxyRegistrar.class)//导入了一个AspectJAutoProxyRegistrar类

AspectJAutoProxyRegistrar实现了ImportBeanDefinitionRegistrar接口
 该类会向容器中注册internalAutoProxyCreator-> AnnotationAwareAspectJAutoProxyCreator
 
AnnotationAwareAspectJAutoProxyCreator继承
->AspectJAwareAdvisorAutoProxyCreator继承
  ->AbstractAdvisorAutoProxyCreator继承
    ->AbstractAutoProxyCreator继承ProxyProcessorSupport
      实现SmartInstantiationAwareBeanPostProcessor，BeanFactoryAware
      
 SmartInstantiationAwareBeanPostProcessor继承
  ->InstantiationAwareBeanPostProcessor继承BeanPostProcessor
  
InstantiationAwareBeanPostProcessor添加了实例化之前的回调，和实例化之后的回调函数
   ->postProcessBeforeInstantiation
   ->postProcessAfterInstantiation
   
//创建AnnotationAwareAspectJAutoProxyCreator（BeanPostProcessor）流程
refresh() //容器刷新
 ->registerBeanPostProcessors(beanFactory) //注册beanPostProcessor
   ->PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this)
     ->beanFactory.getBean(ppName, BeanPostProcessor.class) //获得beanPostProcessor
       ->doGetBean->getSingleton->createBean->doCreateBean //这步bean已实例化
       ->initializeBean->invokeAwareMethods(beanName, bean)//设置beanfactory
       处理完xxxAware之后，
       应用applyBeanPostProcessorsBeforeInitialization
          invokeInitMethods
          applyBeanPostProcessorsAfterInitialization
  =============至此AnnotationAwareAspectJAutoProxyCreator创建结束===============
  
AnnotationAwareAspectJAutoProxyCreator会在所有bean创建之前拦截，调用postProcessBeforeInstantiation

同样在创建bean时，执行到createBean方法，内部会在doCreateBean(beanName, mbdToUse, args)调用前，先调用resolveBeforeInstantiation(beanName, mbdToUse)方法，给beanpostprocessors返回一个代理对象的机会。
进入方法内，执行到
applyBeanPostProcessorsBeforeInstantiation(targetType, beanName)
 1、获得所有的BeanPostProcessor，遍历所有找到属于InstantiationAwareBeanPostProcessor的BeanPostProcessor
 2、存在该BeanPostProcessor，就执行postProcessBeforeInstantiation
 	 1、判断当前bean是否在advisedBeans中（需要增强的bean）
 	 2、isInfrastructureClass判断是否是基础类型Advice、Pointcut、Advisor、AopInfrastructureBean，或者判断是否包含@Aspect注解
 	 3、判断是否需要跳过
 	 	找到所有的候选方法（通知方法），使用的Advisor对方法进行包装，判断没有advisor是否属于AspectJPointcutAdvisor，不是就直接返回false
 	创建bean实例后，执行postProcessAfterInitialization
 	  1、wrapIfNecessary 包装bean
 	     获取对应bean的所有的候选通知，
 	     保存当前bean在advisedBean中，为true
 	     创建bean的代理对象
 			将所有的通知方法添加到proxyfactory
                proxyFactory.addAdvisors(advisors);
                proxyFactory.setTargetSource(targetSource);
                customizeProxyFactory(proxyFactory);
 			重点：proxyFactory.getProxy，获得代理对象
 				JdkDynamicAopProxy、ObjenesisCglibAopProxy有Spring判断进行
 		 最后返回的是Cglib增强的代理对象
================================至此代理对象创建完成====================== 

执行代理对象对应的方法
  1、CglibAopProxy.intercept();
     获得ProxyFactory中对应的增强方法
List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
	   遍历所有的通知方法advisor,
	   registry.getInterceptors(advisor)，将其转换成MethodInterceptor，
	   最终添加到interceptorList中
	   创建CglibMethodInvocation，并执行
	   new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
 ======================>>> 内部使用就是责任链模式，
	   依次取出拦截器方法，
	   调用((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
	   		->mi.proceed()
	   如果拦截器都已取出则执行对应的实际方法
	   invokeJoinpoint()，
	   执行完毕后，依次返回再经过每个拦截器方法
```

## 事务

### @EnableTransactionManagement

开启事务管理器

```java
@Configuration
@EnableTransactionManagement//开启事务管理器
public class ConfigOfTransaction {
    @Bean
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUsername("root");
        dataSource.setPassword("123456");
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/crud?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true&useSSL=false");
        return dataSource;
    }

    @Bean 
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        JdbcTemplate template = new JdbcTemplate(dataSource);
        return template;
    }

    @Bean //向容器中注册事务管理器
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
        return transactionManager;
    }
}
```

### @Transactional

在需要开启事务的方法上添加

```java
@Transactional
public void insert(){
    personDao.insert();
    int i=1/0;
}
```

### 源码

```
//从启动注解进入
@EnableTransactionManagement
//导入了一个TransactionManagementConfigurationSelector类，进入该类
@Import(TransactionManagementConfigurationSelector.class)
public @interface EnableTransactionManagement {
	AdviceMode mode() default AdviceMode.PROXY;
}

//默认注册了两个类AutoProxyRegistrar，ProxyTransactionManagementConfiguration
public class TransactionManagementConfigurationSelector extends AdviceModeImportSelector<EnableTransactionManagement> {
	@Override
	protected String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {
			case PROXY:
				return new String[] {AutoProxyRegistrar.class.getName(), ProxyTransactionManagementConfiguration.class.getName()};
			case ASPECTJ:
				return new String[] {TransactionManagementConfigUtils.TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME};
			default:
				return null;
		}
	}
}

//AutoProxyRegistrar实现了ImportBeanDefinitionRegistrar
	向容器中又注册了internalAutoProxyCreator->InfrastructureAdvisorAutoProxyCreator

//InfrastructureAdvisorAutoProxyCreator同样是实现了
SmartInstantiationAwareBeanPostProcessor，BeanFactoryAware
这里和Aop的原理一样，是个后置处理器，在对象创建后，对其进行包装，返回一个代理对象，代理对象中包含了增强器方法。

//ProxyTransactionManagementConfiguration做什么的

@Configuration //同样是一个配置类
public class ProxyTransactionManagementConfiguration extends AbstractTransactionManagementConfiguration {
	
	//给容器中注册一个事务增强器
	@Bean(name = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME)
	@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
	public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor() {
		BeanFactoryTransactionAttributeSourceAdvisor advisor = new BeanFactoryTransactionAttributeSourceAdvisor();
		
		//事务增强器中包含了事务属性
		advisor.setTransactionAttributeSource(transactionAttributeSource());
		//添加了事务拦截器
		advisor.setAdvice(transactionInterceptor());
		advisor.setOrder(this.enableTx.<Integer>getNumber("order"));
		return advisor;
	}
}

进入事务拦截器中TransactionInterceptor中
找到一个执行方法invoke
	@Override
	public Object invoke(final MethodInvocation invocation) throws Throwable {
		Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);
		
		//在事务内进行执行，进入该方法
		return invokeWithinTransaction(invocation.getMethod(), targetClass, new InvocationCallback() {
			@Override
			public Object proceedWithInvocation() throws Throwable {
				return invocation.proceed();
			}
		});
	}
	
 1、先是获取到了事务的属性，及添加了@Transactional的方法
 2、获取了事务管理器
 3、TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);创建了一个事务
 4、retVal = invocation.proceedWithInvocation(); //执行目标方法
 5、如果发生了异常completeTransactionAfterThrowing(txInfo, ex);
 	利用的事务管理器进行的回滚操作
 	txInfo.getTransactionManager().rollback(txInfo.getTransactionStatus());
 6、没有异常，commitTransactionAfterReturning(txInfo);
 	txInfo.getTransactionManager().commit(txInfo.getTransactionStatus());
 	同样是利用事务管理器进行提交事务
```

## 扩展原理

### ApplicationListener

实现该接口，将其添加到容器中，即可作为一个监听器

```java
@Component
public class MyApplicationListener implements ApplicationListener {
    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        System.out.println("MyApplicationListener:"+event);
    }
}
```

**原理**

1. 在容器刷新的最后一步：finishRefresh();发布相应的事件

2. 进入该方法，第三步：publishEvent(new ContextRefreshedEvent(this));发布事件

3. 先是获取到事件多播器，然后派发事件

   getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);

4. 获得所有的监听器，然后遍历执行，invokeListener(listener, event);

### ApplicationEventMulticaster

事件多播器

refresh()刷新容器中的有一步是：initApplicationEventMulticaster();

进入该方法：

1. 判断工厂中是否有applicationEventMulticaster，有就直接取出
2. 没有，创建一个SimpleApplicationEventMulticaster，添加到容器中

### @EventListener

使用该注解可替代上面的实现接口的方式，来监听事件

```java
@EventListener
public void listen(ApplicationEvent event){
    System.out.println("event:"+event);
}
```

使用的是EventListenerMethodProcessor组件来处理该注解，实例了SmartInitializingSingleton接口。

**源码**

1. refresh();
2. finishBeanFactoryInitialization(beanFactory);
3. beanFactory.preInstantiateSingletons();
4. 在该preInstantiateSingletons方法中，实例化所有的单例bean后，在来了一次循环，来找出属于SmartInitializingSingleton类型实例，然后执行afterSingletonsInstantiated
5. 同样里面会遍历所有的beanNames,获取对应的类型type
6. processBean(factories, beanName, type);
7. 检查该类型中是否有对应的注解@EventListener
8. 如果存在，获取方法method,使用EventListenerFactory创建对应的applicationListener，加入到容器中

## Servlet3.0

### @WebServlet

### @WebListener

### @WebFilter

### ServletContainerInitializer @HandlesTypes

Shared libraries（共享库） / runtimes pluggability（运行时插件能力）

1. Servlet容器启动会扫描，当前应用的每一个jar包的ServletContainerInitializer的实现

2. 提供ServletContainerInitializer的实现类：必须写在该路径文件下

   META-INF/services/javax.servlet.ServletContainerInitializer，文件的内容就是实现类的全类名

3. ```java
   //传入感兴趣的类型
   //容器启动的时候，会将@HandlesTypes指定的类型下的子类（实现类，子接口）传进来
   @HandlesTypes(value={UserService.class})
   public class MyServletContainerInitializer implements ServletContainerInitializer{
   	//启动的时候，向ServletContext中添加组件，除了可以在
   	@Override
   	public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException {
           //打印感兴趣的类
   		for(Class clazz:c){
   			System.out.println(clazz);
   		}
   		//向容器中添加servlet
   		Dynamic dynamic = ctx.addServlet("myServlet", MyServlet.class);
   		dynamic.addMapping("/myServelt");
   		//添加监听器
   		ctx.addListener(MyListener.class);
   		//添加过滤器
   		javax.servlet.FilterRegistration.Dynamic filter = ctx.addFilter("myFilter", MyFilter.class);
   		
   		filter.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST),true, "/*");
   	}
   }
   ```

### 异步请求

1. 客户端发送请求
2. servlet容器分配一个线程并其中调用Servlet
3. Servlet调用request.startAsync(),保存AsyncContext,然后就返回
4. 容器线程一直退出状态，但是响应一直保持打开状态
5. 其它的线程使用保存AsyncContext来完成响应
6. 客户收到结果

```java
@WebServlet(urlPatterns="/async",asyncSupported=true)//开启支持异步
public class HelloAsyncServlet extends HttpServlet{
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		AsyncContext async = req.startAsync();
		System.out.println("主线程："+Thread.currentThread()+"==>"+System.currentTimeMillis());
		async.start(()->{
			System.out.println("子线程："+Thread.currentThread()+"==>"+System.currentTimeMillis());
			
			try {
				Thread.sleep(3000);
			} catch (Exception e) {
				e.printStackTrace();
			}
			//异步执行完毕
			async.complete();
			//获取响应
			ServletResponse response = async.getResponse();
			
			try {
				response.getWriter().write("hello async...");
			} catch (Exception e) {
				e.printStackTrace();
			}
			System.out.println("子线程："+Thread.currentThread()+"==>"+System.currentTimeMillis());
		});
		System.out.println("主线程："+Thread.currentThread()+"==>"+System.currentTimeMillis());
	}
}
```

## SpringMVC

### WebApplicationInitializer

web容器启动的时候会扫描每个jar包下的META-INF/services/javax.servlet.ServletContainerInitializer文件，加载文件中指定的对象org.springframework.web.SpringServletContainerInitializer

```java
@HandlesTypes(WebApplicationInitializer.class)//这里会引入该接口下的子类
public class SpringServletContainerInitializer implements ServletContainerInitializer {
	@Override
	public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
			throws ServletException {
		List<WebApplicationInitializer> initializers = new LinkedList<WebApplicationInitializer>();

		if (webAppInitializerClasses != null) {
			for (Class<?> waiClass : webAppInitializerClasses) {
				if (!waiClass.isInterface() && !Modifier.isAbstract(waiClass.getModifiers()) &&
						WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
					try {
						initializers.add((WebApplicationInitializer) waiClass.newInstance());
					}
					catch (Throwable ex) {
						throw new ServletException("Failed to instantiate WebApplicationInitializer class", ex);
					}
				}
			}
		}
		if (initializers.isEmpty()) {
			servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
			return;
		}
		servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
		AnnotationAwareOrderComparator.sort(initializers);
		for (WebApplicationInitializer initializer : initializers) {
            
			initializer.onStartup(servletContext);
		}
	}

}

```

spring应用一旦启动就会加载WebApplicationInitializer接口下的所有的组件

1. 抽象类AbstractContextLoaderInitializer实现了该接口

   createRootApplicationContext()：创建一个根容器对象

2. 抽象类AbstractDispatcherServletInitializer继承AbstractContextLoaderInitialize

   - createServletApplicationContext：创建一个web servlet的应用容器

   - createDispatcherServlet：创建一个DispatcherServlet

   - servletContext.addServlet(servletName, dispatcherServlet);将DispatcherServlet设置到Servlet中

   - ```java
     	protected void registerDispatcherServlet(ServletContext servletContext) {
     		String servletName = getServletName();
     		Assert.hasLength(servletName, "getServletName() must not return empty or null");
     
     		WebApplicationContext servletAppContext = createServletApplicationContext();
     		Assert.notNull(servletAppContext,
     				"createServletApplicationContext() did not return an application " +
     				"context for servlet [" + servletName + "]");
     //创建一个dispatcherServlet
     		FrameworkServlet dispatcherServlet = createDispatcherServlet(servletAppContext);
     		dispatcherServlet.setContextInitializers(getServletApplicationContextInitializers());
     
     		ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet);
     		Assert.notNull(registration,
     				"Failed to register servlet with name '" + servletName + "'." +
     				"Check if there is another servlet registered under the same name.");
     		//设置优先级
     		registration.setLoadOnStartup(1);
             //添加映射
     		registration.addMapping(getServletMappings());
             //设置支持异步
     		registration.setAsyncSupported(isAsyncSupported());
     
     		Filter[] filters = getServletFilters();
     		if (!ObjectUtils.isEmpty(filters)) {
     			for (Filter filter : filters) {
     				registerServletFilter(servletContext, filter);
     			}
     		}
     		customizeRegistration(registration);
     	}
     ```

3. AbstractAnnotationConfigDispatcherServletInitializer继承AbstractDispatcherServletInitializer

   注解的方式配置

   createRootApplicationContext：创建一个根容器

   getRootConfigClasses：加载根容器的配置

   createServletApplicationContext：创建web容器

   getServletConfigClasses：加载web容器的配置

```java
public class MyServletInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
	
	@Override
	public void onStartup(ServletContext servletContext) throws ServletException {
		super.onStartup(servletContext);
		//编码过滤器
		Dynamic encoderFilter = servletContext.addFilter("characterEncodingFilter", new CharacterEncodingFilter("UTF-8"));
		encoderFilter.addMappingForUrlPatterns(EnumSet.of(DispatcherType.REQUEST),true,"/*");
	}
	@Override
	protected Class<?>[] getRootConfigClasses() {
		return new Class[]{RootConfig.class};
	}
	@Override
	protected Class<?>[] getServletConfigClasses() {
		return new Class[]{AppConfig.class};
	}
	//添加拦截
	// /:这个代表拦截所有请求，但是不包括.jsp
	// /*:包括.jsp
	@Override
	protected String[] getServletMappings() {
		return new String[]{"/"};
	}
}
```

### 异步请求

#### DeferredResult

```java
@RequestMapping("/asyncResult")
@ResponseBody
public DeferredResult<?> getResult(){
    //创建一个延迟对象
	DeferredResult<Object> result = new DeferredResult<>(5000l,"fail...");
    //将延迟对象加入到队列中
	queue.add(result);
	return result;
}

@RequestMapping("/handler")
@ResponseBody
public String handlerReq(){
	DeferredResult<Object> result = (DeferredResult<Object>) queue.poll();
    //取出延迟对象，设置返回值
	result.setResult("diaoyuhang");
	return "success";
}
```
#### Callable

1. 控制器返回Callable
2. Springmvc调用request.startAsync并提交callable到一个TaskExecutor，用单独一个线程执行
3. 同时DispatcherServelt,所有Filter退出Servlet容器，但是响应保持打开状态
4. 最终callable返回一个结果并且Springmvc将请求重新分配回Servlet容器已完成处理
5. dispatcherServlet再次调用执行恢复从callable异步返回的结果

```java
@RequestMapping("/async")
@ResponseBody
public Callable<String> async01(){
	System.out.println("主线程："+Thread.currentThread()+" 时间"+System.currentTimeMillis());
	Callable<String> callable = new Callable<String>() {
		@Override
		public String call() throws Exception {
			System.out.println("子线程："+Thread.currentThread()+" 时间"+System.currentTimeMillis());
			Thread.sleep(3000);
			System.out.println("子线程："+Thread.currentThread()+" 时间"+System.currentTimeMillis());
			return "子线程返回结果";
		}
	};
	System.out.println("主线程："+Thread.currentThread()+" 时间"+System.currentTimeMillis());
	return callable;
}
```
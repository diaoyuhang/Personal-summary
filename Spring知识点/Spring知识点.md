## ApplicationContextAwareProcessor—bean后处理器

- 该处理器添加到beanFactory中的，发生在refresh的prepareBeanFactory阶段；
- 该处理器往实现了EnvironmentAware、EmbeddedValueResolverAware、ResourceLoaderAware、ApplicationEventPublisherAware、MessageSourceAware、ApplicationStartupAware、ApplicationContextAware这些接口的bean对象中设置ConfigurableEnvironment、StringValueResolver、ResourceLoader、ApplicationEventPublisher、MessageSource、ApplicationStartup、ApplicationContext；

## ConfigurationClassPostProcessor—bean工厂后处理器

发生在spring context 刷新过程中invokeBeanFactoryPostProcessors步骤(调用在上下文中已经注册成为bean的工厂处理器)；

**第一阶段解析ConfigurationClass, parser.parse(candidates);**

解析启动类上注解信息，处理@PropertySource、@ComponentScan、@Import、@ImportResource、@Bean注解信息；

在处理@Import阶段，获取到@EnableAutoConfiguration注解中的AutoConfigurationImportSelector、AutoConfigurationPackages.Registrar.class；

- 将AutoConfigurationPackages.Registrar.class加入到importBeanDefinitionRegistrars集合中，待后续加载bean定义；
- 如果是DeferredImportSelector实现，就在后续执行时加载AutoConfiguration.imports配置文件中的类，如果是ImportSelector当场就执行获取需要处理的bean字符串；

获取到ConfigurationClass集合；

**第二阶段读取bean定义信息，this.reader.loadBeanDefinitions(configClasses);**

遍历configclasses集合

加载bean定义从@importresouces中获取，loadBeanDefinitionsFromImportedResources；

加载bean定义实现了ImportBeanDefinitionRegistrar接口的实现类中，loadBeanDefinitionsFromRegistrars；



## AutoConfigurationImportSelector.AutoConfigurationGroup

会在ConfigurationClassPostProcessor中执行加载META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports文件中的配置类；

这配置文件中所写的都是自动配置类，在ConfigurationClassPostProcessor这个工厂后置处理器中将对应需要的@Bean标注得方法已bean定义信息注册到了bean工厂中的beanDefinitionMap中，待后续的bean实例化阶段执行这些方法；

## BeanPostProcessor后置处理器

在bean执行初始化方法前后执行postProcessBeforeInitialization，postProcessAfterInitialization方法；

CommonAnnotationBeanPostProcessor对类中的@Resources解析,

AutowiredAnnotationBeanPostProcessor对类中的@Autowired、@Value解析，

切面就是通过AnnotationAwareAspectJAutoProxyCreator返回代理；

## bean作用域

| 作用域         | 描述                                                         |
| :------------- | :----------------------------------------------------------- |
| singleton      | 在spring IoC容器仅存在一个Bean实例，Bean以单例方式存在，默认值 |
| prototype      | 每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行newXxxBean() |
| request        | 每次HTTP请求都会创建一个新的Bean，该作用域仅适用于WebApplicationContext环境 |
| session        | 同一个HTTP Session共享一个Bean，不同Session使用不同的Bean，仅适用于WebApplicationContext环境 |
| global-session | 一般用于Portlet应用环境，该作用域仅适用于WebApplicationContext环境 |

**但作用域为singleton的bean依赖作用域为request的bean的时候，需要将作用域为request的bean，创建aop代理<aop:scoped-proxy/>，不能直接注入到单例bean中，因为单例bean注入只发生一次**




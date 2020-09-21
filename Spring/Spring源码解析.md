# 	Spring

## 概览

![](images\Spring创建对象粗略图.png)

> ### Spring 给出了扩展：
>
> 1. ### 创建对象之前可以干些事
>
> 2. ### 容器初始化之前可以干点事
>
> 3. ### 在不同的阶段发出不同的时间 干点事
>
> 4. ### 抽象出各个接口
>
> 5. ### 面向接口编程

## 架构设计过程（抽象出接口）

![](images\架构设计过程.png)

## BeanFactory

```java
/**
用于访问spring容器的根接口
 * The root interface for accessing a Spring bean container.
 * This is the basic client view of a bean container;
 
 进一步的接口有ListableBeanFactory,ConfigurableBeanFactory,用于特定的目的
 
 * further interfaces such as {@link ListableBeanFactory} and
 * {@link org.springframework.beans.factory.config.ConfigurableBeanFactory}
 * are available for specific purposes.
 
   注意，通过setters或构造器配置应用对象，使用依赖注入通常会更好
 *<p>Note that it is generally better to rely on Dependency Injection
 * ("push" configuration) to configure application objects through setters
 * or constructors, rather than use any form of "pull" configuration like a
 
  spring的依赖注入功能是使用的beanfactory接口和其子接口实现的
 * BeanFactory lookup. Spring's Dependency Injection functionality is
 * implemented using this BeanFactory interface and its subinterfaces.
 
  * <p>In contrast to the methods in {@link ListableBeanFactory}, all of the
 * operations in this interface will also check parent factories if this is a
 * {@link HierarchicalBeanFactory}.
 
 如果bean没有在这个工厂对象中找到，当前的父类工厂将会被访问。当前工厂中beans会覆盖父类中的同名beans
 If a bean is not found in this factory instance,
 * the immediate parent factory will be asked. Beans in this factory instance
 * are supposed to override beans of the same name in any parent factory.
 
   bean工厂实现应该尽可能的支持标准的生命周期接口
 * <p>Bean factory implementations should support the standard bean lifecycle interfaces as far as possible.
 
  全套的初始化方法及其标准顺序：
 *The full set of initialization methods and their standard order is:
 * <ol>
 
 1、经过一系列的xxxAware把bean需要的信息调用setxxx给bean
 * <li>BeanNameAware's {@code setBeanName}
 * <li>BeanClassLoaderAware's {@code setBeanClassLoader}
 * <li>BeanFactoryAware's {@code setBeanFactory}
 * <li>EnvironmentAware's {@code setEnvironment}
 * <li>EmbeddedValueResolverAware's {@code setEmbeddedValueResolver}
 * <li>ResourceLoaderAware's {@code setResourceLoader}
 
    仅在应用上下文运行中适用
 * (only applicable when running in an application context)
 * <li>ApplicationEventPublisherAware's {@code setApplicationEventPublisher}
 * (only applicable when running in an application context)
 * <li>MessageSourceAware's {@code setMessageSource}
 * (only applicable when running in an application context)
 * <li>ApplicationContextAware's {@code setApplicationContext}
 * (only applicable when running in an application context)
 * <li>ServletContextAware's {@code setServletContext}
  
   仅在web应用上下文中运行适用
 * (only applicable when running in a web application context)
  调用beanPostProcessors beforeInitialization
 * <li>{@code postProcessBeforeInitialization} methods of BeanPostProcessors
 
 	初始化
 * <li>InitializingBean's {@code afterPropertiesSet}
    调用用户自定义的init-method方法
 * <li>a custom init-method definition
 
   调用beanPostProcessors afterInitializaion
 * <li>{@code postProcessAfterInitialization} methods of BeanPostProcessors
 * </ol>
 * 在关闭bean工厂时，遵循以下生命周期方法的
 * <p>On shutdown of a bean factory, the following lifecycle methods apply:
 * <ol>
 * <li>{@code postProcessBeforeDestruction} methods of DestructionAwareBeanPostProcessors
    	调用DisposableBean接口
 * <li>DisposableBean's {@code destroy}
     destroy-method方法
 * <li>a custom destroy-method definition
 * </ol>
 *
 */
```

### ListableBeanFactory

```java
/**
	BeanFactory的延伸，实现该接口能够列举出所有的bean实例
 * Extension of the {@link BeanFactory} interface to be implemented by bean factories
 * that can enumerate all their bean instances,
 * rather than attempting bean lookup
 * by name one by one as requested by clients. BeanFactory implementations that
 * preload all their bean definitions (such as XML-based factories) may implement
 * this interface.
```

### 	HierarchicalBeanFactory

```JAVA
/**
	实现该接口的bean工厂，会形成一个层级关系
 * Sub-interface implemented by bean factories that can be part
 * of a hierarchy.
 *
 * <p>The corresponding {@code setParentBeanFactory} method for bean
 * factories that allow setting the parent in a configurable
 * fashion can be found in the ConfigurableBeanFactory interface.
 *
```

## FactoryBean

```java
/**
	beanfactory中使用的对象实现了该接口，那么这些对象本身就是生产单个对象的工厂
 * Interface to be implemented by objects used within a {@link BeanFactory} which
 * are themselves factories for individual objects. 
 
 如果一个bean实现了这个接口，它通常作为作为一个工厂生产一个对象，不是直接作为一个bean实例暴露它自己
 *If a bean implements this
 * interface, it is used as a factory for an object to expose, not directly as a
 * bean instance that will be exposed itself.
 
 一个bean实现了这个接口不能作为一个正常的bean
  * <p><b>NB: A bean that implements this interface cannot be used as a normal bean.</b>
  factorybean被定义为一个bean的风格，但是暴露对象引用的对象始终是它创建的
 * A FactoryBean is defined in a bean style, but the object exposed for bean
 * references ({@link #getObject()}) is always the object that it creates.
 *
 FactoryBean能够支持单例和prototypes
  * <p>FactoryBeans can support singletons and prototypes, and can either create
 * objects lazily on demand or eagerly on startup. 
 
 SmartFactoryBean接口允许暴露更多的细粒度的行为原数据
 The {@link SmartFactoryBean}
 * interface allows for exposing more fine-grained behavioral metadata.
```

### SmartFactoryBean

## Environment

```java
/**
   表示当前应用程序运行环境的接口
 * Interface representing the environment in which the current application is running.
 
 为应用环境的两个方面准备：profiles和properties
 * Models two key aspects of the application environment: <em>profiles</em> and
 * <em>properties</em>.
 * Methods related to property access are exposed via the
 * {@link PropertyResolver} superinterface.
 	
 	只有当给定的profile是active，才向容器中注册的bean定义的逻辑组
  * <p>A <em>profile</em> is a named, logical group of bean definitions to be registered with the container only if the given profile is <em>active</em>. Beans may be assigned
 * to a profile whether defined in XML or via annotations; see the spring-beans 3.1 schema
 * or the {@link org.springframework.context.annotation.Profile @Profile} annotation for
 * syntax details. The role of the {@code Environment} object with relation to profiles is
 * in determining which profiles (if any) are currently {@linkplain #getActiveProfiles
 * active}, and which profiles (if any) should be {@linkplain #getDefaultProfiles active
 * by default}.
 	
 	properties在所有的应用中扮演者重要的角色，
  * <p><em>Properties</em> play an important role in almost all applications,
  来源于各种配置文件：配置文件，jvm系统配置，系统环境变量。。。。
  and may originate from a variety of sources: properties files, JVM system properties, system environment variables, JNDI, servlet context parameters, ad-hoc Properties objects, Maps, and so on.
  
  环境角色和配置文件的关系是为了提供方便的服务接口，以配置属性源和解析配置
  The role of the environment object with relation to properties is to provide the user with a convenient service interface for configuring property sources and resolving properties from them.
  
  在多数情况下，应用程序级的bean不需要直接和环境交互，但是有${}符号时替换成属性值用属性占位符配置器
   * <p>In most cases, however, application-level beans should not need to interact with the {@code Environment} directly but instead may have to have {@code ${...}} property values replaced by a property placeholder configurer 
 such as
 * {@link org.springframework.context.support.PropertySourcesPlaceholderConfigurer
 * PropertySourcesPlaceholderConfigurer}, which itself is {@code EnvironmentAware} and
 * as of Spring 3.1 is registered by default when using
 * {@code <context:property-placeholder/>}.
 
 环境对象的配置必须实现ConfigurableEnvironment接口，
  * <p>Configuration of the environment object must be done through the
 * {@code ConfigurableEnvironment} interface, 
 返回所有的子类方法通过通过AbstractApplicationContext的getEnvironment方法
 returned from all {@code AbstractApplicationContext} subclass {@code getEnvironment()} methods.
 
 See {@link ConfigurableEnvironment} Javadoc for usage examples demonstrating manipulation
 * of property sources prior to application context {@code refresh()}.
```

### ConfigurableEnvironment

```java
/**
	绝大多数类型都将实现配置接口
 * Configuration interface to be implemented by most if not all {@link Environment} types.
 
 提供了设置active和默认的配置并且操作基础的配置源
 * Provides facilities for setting active and default profiles and manipulating underlying property sources. 
 
 允许客户端设置验证所需的属性，自定义转化器服务通过ConfigurablePropertyResolver超级接口
 Allows clients to set and validate required properties, customize the
 * conversion service and more through the {@link ConfigurablePropertyResolver}
 * superinterface.
 
 操作属性源
 <h2>Manipulating property sources</h2>
 属性源可能被移除，重新排序，或者替换；并且可以从MutablePropertySources实例返回得到其他属性源
 * <p>Property sources may be removed, reordered, or replaced; and additional
     * property sources may be added using the {@link MutablePropertySources}
     * instance returned from {@link #getPropertySources()}.
    
  以下实例与ConfigurableEnvironment的实现StandardEnvironment相反，但是，通常应用任何实现，尽管特定的默认的属性源不同
    The following examples are against the {@link StandardEnvironment} implementation of {@code ConfigurableEnvironment}, but are generally applicable to any implementation,
 * though particular default property sources may differ.
 
 当Environment在applicationContext中使用时，它是非常重要的，因为PropertySource任何操作都在上下文刷新refresh之前被调用
  * When an {@link Environment} is being used by an {@code ApplicationContext}, it is
 * important that any such {@code PropertySource} manipulations be performed
 * <em>before</em> the context's {@link
 * org.springframework.context.support.AbstractApplicationContext#refresh() refresh()} method is called. 
 
 这确保了所有的属性源在容器引导过程中都是可用的
 This ensures that all property sources are available during the container bootstrap process, including use by {@linkplain
 * org.springframework.context.support.PropertySourcesPlaceholderConfigurer property
 * placeholder configurers}.
```



### PropertySourcesPlaceholderConfigurer

```java

```



### PropertiesLoaderSupport

```java
/**
	基础类，用于从一个或多个数据源中加载配置的javabean组件
 * Base class for JavaBean-style components that need to load properties
 * from one or more resources. Supports local properties as well, with
 * configurable overriding.
 *
```

##  BeanFactoryPostProcessor

```java
/**
	工厂挂钩允许自定义修改应用上下文的bean定义信息，调整上下文的基础bean属性值的工厂
 * Factory hook that allows for custom modification of an application context's
 * bean definitions, adapting the bean property values of the context's underlying
 * bean factory.
 
 针对自定义配置文件，这些配置会覆盖应用上下文中的bean配置
  * <p>Useful for custom config files targeted at system administrators that
 * override bean properties configured in the application context. See
 * {@link PropertyResourceConfigurer} and its concrete implementations for
 * out-of-the-box solutions that address such configuration needs.
 
   beanFactoryPostProcessor可以与bean定义交互修改，但是不能修改实例bean
  * <p>A {@code BeanFactoryPostProcessor} may interact with and modify bean
 * definitions, but never bean instances. Doing so may cause premature bean
 * instantiation, violating the container and causing unintended side-effects.
 
 如果需要bean实例的交互修改，考虑实现beanpostProcessor接口
 * If bean instance interaction is required, consider implementing
 * {@link BeanPostProcessor} instead.
```

## ApplicationContext应用上下文



![](D:\0_LeargingSummary\Spring\images\ApplicationContext继承体系结构.png)

```java
/**
	核心接口为应用提供配置。
 * Central interface to provide configuration for an application.
	 在应用运行时是只读的，但是如果实现了该接口则可以重新加载
 * This is read-only while the application is running, but may be
 * reloaded if the implementation supports this.
 
  * <p>An ApplicationContext provides:
 * <ul>
 	用于访问应用组件的bean工厂方法
 * <li>Bean factory methods for accessing application components.
 * Inherited from {@link org.springframework.beans.factory.ListableBeanFactory}.
 
 	以通用的方式加载文件资源的能力
  * <li>The ability to load file resources in a generic fashion.
 * Inherited from the {@link org.springframework.core.io.ResourceLoader} interface.
 
 	发布事件给注册的监听器的能力
  * <li>The ability to publish events to registered listeners.
 * Inherited from the {@link ApplicationEventPublisher} interface.
 
 	解析消息，支持国家化的能力
  * <li>The ability to resolve messages, supporting internationalization.
 * Inherited from the {@link MessageSource} interface.
 
 从父类容器中继承，后代中的定义总是保持着优先权。
  * <li>Inheritance from a parent context. Definitions in a descendant context
 * will always take priority.
 
这意味着，单个父上下文能够被整个应用使用，
 *This means, for example, that a single parent context can be used by an entire web application, 
 
 当每一个servlet有它自己的子上下文，该子上下文独立于其它servlet的子上下文
 *while each servlet has its own child context that is independent of that of any other servlet.
```

> ### 子上下文可以获得父上下文的BeanFactory---getParentBeanFactory

### AutowireCapableBeanFactory自动装配功能BeanFactory

```java
/**
	beanfactory的扩展接口，实现该接口的bean工厂能够具有自动装配的能力
 * Extension of the {@link org.springframework.beans.factory.BeanFactory}
 * interface to be implemented by bean factories that are capable of
 * autowiring, 
 
 前提是他们为现有的bean实例公开此功能
 provided that they want to expose this functionality for
 * existing bean instances.
 
	其它框架可以利用这个接口连接填充Spring无法控制的现有的bean实例的生命周期
  * <p>Integration code for other frameworks can leverage this interface to
 * wire and populate existing bean instances that Spring does not control
 * the lifecycle of. This is particularly useful for WebWork Actions and
 * Tapestry Page objects, for example.
 
 注意，这个接口没有被applicationcontext实现，它非常难用，
  * <p>Note that this interface is not implemented by
 * {@link org.springframework.context.ApplicationContext}  	 	,
 * as it is hardly ever used by application code.
 
 换句话说，它时可用的，通过ApplicationContext.getAutowireCapableBeanFactory获得
 That said, it is available
 * from an application context too, accessible through ApplicationContext's
 * {@link org.springframework.context.ApplicationContext#getAutowireCapableBeanFactory()}
 * method.
```

## AbstractApplicationContext抽象应用上下文

```java
 /*
 与普通的beanfactory相反，applicationcontext支持检测特殊的bean定义，在他的内部的bean工厂中
 *<p>In contrast to a plain BeanFactory, an ApplicationContext is supposed
 * to detect special beans defined in its internal bean factory:
 
 因此这些class自动注册
 * Therefore, this class automatically registers
 * {@link org.springframework.beans.factory.config.BeanFactoryPostProcessor BeanFactoryPostProcessors},
 * {@link org.springframework.beans.factory.config.BeanPostProcessor BeanPostProcessors},
 * and {@link org.springframework.context.ApplicationListener ApplicationListeners}
 * which are defined as beans in the context.
 
   messageSource可能作为一个bean在上下文中被提供，name为messgasource
  * <p>A {@link org.springframework.context.MessageSource} may also be supplied
 * as a bean in the context, with the name "messageSource"; 
 
 否则，信息解析被委托给父上下文中
 otherwise, message resolution is delegated to the parent context. 
 
   此外，应用事件多播器在上下文中由applicationEventMulticaster提供
 Furthermore, a multicaster for application events can be supplied as an "applicationEventMulticaster" bean
 * of type {@link org.springframework.context.event.ApplicationEventMulticaster}
 * in the context; 
 
 否则，就是默认的多播器simpleApplicationEventMulticaster
 otherwise, a default multicaster of type
 * {@link org.springframework.context.event.SimpleApplicationEventMulticaster} will be used.

  实现资源加载通过继承DefaultResourceLoader
 Implements resource loading by extending
 * {@link org.springframework.core.io.DefaultResourceLoader}.
 
 所以将非url资源路径作为类路径资源
 * Consequently treats non-URL resource paths as class path resources
 支持完整的类路径资源名包括包路径
 * (supporting full class path resource names that include the package path,
 * e.g. "mypackage/myresource.dat"), unless the {@link #getResourceByPath}
 * method is overridden in a subclass.
 */

//面试重点:分为12步
@Override
public void refresh() throws BeansException, IllegalStateException {
    //加锁防止多线程重复刷新
    synchronized (this.startupShutdownMonitor) {
        //为刷新准备上下文
        //标记当前上下文处于活动状态，active=true
        //初始化了上下文中属性源中的占位符
        //验证必须存在的属性不能为null
        //允许收据早期的applicationevents,一旦发布器可用就发布事件
        prepareRefresh();
		
        //告诉子类刷新内部的bean工厂
        //默认创建DefaultListableBeanFactory，如果有父上下文则取出父beanfactory
        //加载bean定义信息（使用XmlBeanDefinitionReader..），读取applicationcontext.xml,存放在beanDefinitionMap
        //将当前创建的beanfactory设置到当前上下文中
        //返回获得到该beanfactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        //准备在这个上下文中使用的bean工厂
        //设置classloader
        //设置表达式解析器StandardBeanExpressionResolver，用于解析Spel
     //添加bean后置处理器ApplicationContextAwareProcessor，涉及到bean生命周期的一些属性设置
// 1. 如果bean实现了EnvironmentAware接口，调用bean.setEnvironment
// 2. 如果bean实现了EmbeddedValueResolverAware接口，调用bean.setEmbeddedValueResolver
// 3. 如果bean实现了ResourceLoaderAware接口，调用bean.setResourceLoader
// 4. 如果bean实现了ApplicationEventPublisherAware接口，调用setApplicationEventPublisher
// 5. 如果bean实现了MessageSourceAware接口，调用bean.setMessageSource
// 6. 如果bean实现了ApplicationContextAware接口，调用bean.setApplicationContext

        //自动装配时忽略给定的依赖接口
        //用相应的自动装配值注册一个特殊的依赖类型。（注册一些特殊的bean）
        //注册早期的后置处理器用于检测bean为ApplicationListeners，
        //注册默认环境bean(environment，systemProperties，systemEnvironment)
        prepareBeanFactory(beanFactory);

        try {
            // 标准初始化后，修改applicationContext内部的beanfactory,所有的bean定义已经被加载，但是没有bean被实例化
            postProcessBeanFactory(beanFactory);

            //执行beanFactoryPostProcessor
            //首先执行容器初始化传入的BeanDefinitionRegistryPostProcessors,这个会向BeanDefinitionRegistry注册新的bean定义信息
         //1、筛选出BeanDefinitionRegistryPostProcessors实现了PriorityOrdered，排序执行
         //2、筛选出BeanDefinitionRegistryPostProcessors实现了Ordered，排序执行
         //3、筛选所有剩余的BeanDefinitionRegistryPostProcessors，执行
         	//最终执行容器初始化传入的常规的BeanFactoryPostProcessors
            
//---------------到这里容器初始化传入的的beanFactoryPostProcessor，和容器中的BeanDefinitionRegistryPostProcessors都执行结束--------------------------
            
    //执行容器中BeanFactoryPostProcessor，同样分优先级排序执行，PriorityOrdered，Ordered
            invokeBeanFactoryPostProcessors(beanFactory);

            //注册BeanPostProcessor.
            //同样这里也是分优先级，PriorityOrdered，Ordered，rest，分别排序执行
            //最后注册internal BeanPostProcessors (MergedBeanDefinitionPostProcessor):AutowiredAnnotationBeanPostProcessor用于 Autowired，Value
            //注册检测bean为applicationListeners的后置处理器
            registerBeanPostProcessors(beanFactory);

            //为这个上下文初始化消息源（用于国际化处理）
            initMessageSource();

            //初始化事件多播器，默认创建SimpleApplicationEventMulticaster（用于管理监听器，和发布事件给相应的监听器）
            initApplicationEventMulticaster();

            //初始化指定的子类上下文中的特殊bean
            onRefresh();

            //向ApplicationEventMulticaster中注册监听器
            registerListeners();

            //实例化所有非懒加载的单例bean
            //判断上下文中是否包含conversionService，存在就设置（用于数据类型转换，场景前后端数据的传送）
            //注册一个内嵌值解析器，解析占位符
            //提前实例化实现了LoadTimeWeaverAware的bean
            //停止使用临时的classloader
            //缓存所有的bean定义信息元数据
            //初始化所有剩余的非懒加载的单例
            finishBeanFactoryInitialization(beanFactory);

            //发布相应的事件
            //为这个上下文初始化生命周期
            //首先将刷新完毕事件传播到生命周期处理器
            //发布最终事件ContextRefreshedEvent，容器刷新完毕事件
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
 

/**
为刷新准备上下文，设置启动时间和激化标志以及执行任何初始化的属性源
	 * Prepare this context for refreshing, setting its startup date and
	 * active flag as well as performing any initialization of property sources.
	 */
protected void prepareRefresh() {
    // Switch to active.
    this.startupDate = System.currentTimeMillis();
    this.closed.set(false);
    this.active.set(true);

    if (logger.isDebugEnabled()) {
        if (logger.isTraceEnabled()) {
            logger.trace("Refreshing " + this);
        }
        else {
            logger.debug("Refreshing " + getDisplayName());
        }
    }

    // Initialize any placeholder property sources in the context environment.
    initPropertySources();

    // Validate that all properties marked as required are resolvable:
    // see ConfigurablePropertyResolver#setRequiredProperties
    getEnvironment().validateRequiredProperties();

    // Store pre-refresh ApplicationListeners...
    if (this.earlyApplicationListeners == null) {
        this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
    }
    else {
        // Reset local application listeners to pre-refresh state.
        this.applicationListeners.clear();
        this.applicationListeners.addAll(this.earlyApplicationListeners);
    }

    // Allow for the collection of early ApplicationEvents,
    // to be published once the multicaster is available...
    this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

### SimpleApplicationEventMulticaster简单应用的事件多播器

```java
/**
	applicationEventMulticaster的简单实现的接口
 * Simple implementation of the {@link ApplicationEventMulticaster} interface.
 
 	将所有的事件多播到所有已注册的监听器中，让监听器忽略不感兴趣的事件
  * <p>Multicasts all events to all registered listeners, leaving it up to
 * the listeners to ignore events that they are not interested in.
 
 	监听器将执行相应的instanceof，检查传入的对象
 * Listeners will usually perform corresponding {@code instanceof}
 * checks on the passed-in event object.
 
  默认情况下，所有的监听器在调用线程中被调用。
 * <p>By default, all listeners are invoked in the calling thread.
 
  这就会允许流氓监听器阻塞整个应用带来危险
 * This allows the danger of a rogue listener blocking the entire application,
 
 但是这增加最小的开销，指定一个额外的任务执行器在不同的线程中执行监听器，比如线程池
 * but adds minimal overhead. Specify an alternative task executor to have
 * listeners executed in different threads, for example from a thread pool.
 */
 public class SimpleApplicationEventMulticaster extends AbstractApplicationEventMulticaster {

	@Nullable
	private Executor taskExecutor;//如果为空，则是单线程执行监听器
```

### refresh方法

#### 1、prepareRefresh方法

```java
// Prepare this context for refreshing.为刷新准备上下文
prepareRefresh();
```

```java
/** Flag that indicates whether this context is currently active. */
private final AtomicBoolean active = new AtomicBoolean();

this.active.set(true);//标记该上下文目前是否处于活动状态
```

```java
// Initialize any placeholder property sources in the context environment.
initPropertySources();//在上下文环境中初始化任何占位符属性源
```

```java
//验证所有的被标记为必需的属性都是可以被解析的
// Validate that all properties marked as required are resolvable:
//查看配置属性解析器#设置需要的属性setRequiredProperties
// see ConfigurablePropertyResolver#setRequiredProperties
getEnvironment().validateRequiredProperties();


/**
	以可配置的形式返回这个应用上下文，允许进一步的自定义
	 * Return the {@code Environment} for this application context in configurable
	 * form, allowing for further customization.
	 
	 如果未指定，就会初始化一个默认的environment--StandardEnvironment
	 * <p>If none specified, a default environment will be initialized via
	 * {@link #createEnvironment()}.
	 */
@Override
public ConfigurableEnvironment getEnvironment() {
    if (this.environment == null) {
        this.environment = createEnvironment();
    }
    return this.environment;
}

/**
	验证每一个指定的属性是存在的并且解析不为NULL值
	 * Validate that each of the properties specified by
	 * {@link #setRequiredProperties} is present and resolves to a
	 * non-{@code null} value.
	 
	 否则抛出异常
	 * @throws MissingRequiredPropertiesException if any of the required
	 * properties are not resolvable.
	 */
	void validateRequiredProperties() throws MissingRequiredPropertiesException;
```

```java
// Store pre-refresh ApplicationListeners...存储提前刷新的applicationListeners
		if (this.earlyApplicationListeners == null) {
			this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
		}
		else {
			// Reset local application listeners to pre-refresh state.
			this.applicationListeners.clear();
			this.applicationListeners.addAll(this.earlyApplicationListeners);
		}

		// Allow for the collection of early ApplicationEvents,
		// to be published once the multicaster is available...
		this.earlyApplicationEvents = new LinkedHashSet<>();
```

#### 2、obtainFreshBeanFactory方法

```java
//告诉子类刷新内部的bean工厂
// Tell the subclass to refresh the internal bean factory.
ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
```

```java
/**
	 * Tell the subclass to refresh the internal bean factory.
	 * @return the fresh BeanFactory instance
	 * @see #refreshBeanFactory()
	 * @see #getBeanFactory()
	 */
	protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
		refreshBeanFactory();
		return getBeanFactory();
	}



/**
该实现执行一个实际刷新上下文的基础bean工厂，关闭前一个bean工厂（如果存在）并且为上下文生命周期的下一个阶段初始化一个新的bean工厂
	 * This implementation performs an actual refresh of this context's underlying
	 * bean factory, shutting down the previous bean factory (if any) and
	 * initializing a fresh bean factory for the next phase of the context's lifecycle.
	 */
	@Override
	protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		try {
            //默认创建返回bean工厂DefaultListableBeanFactory
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			beanFactory.setSerializationId(getId());
            //定制beanFactory
			customizeBeanFactory(beanFactory);
            //加载bean定义信息，这个时候环境已经有了，bean工厂也有了，就差生产bean的来源
			loadBeanDefinitions(beanFactory);
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}


/**
	为这个上下文创建一个内部bean工厂
	 * Create an internal bean factory for this context.
	 每次refresh都会调用该方法
	 * Called for each {@link #refresh()} attempt.
	 
	 默认实现创建一个DefaultListableBeanFactory，这个父上下文通过getInternalParentBeanFactory得到bean工厂作为父bean工厂。可以在子类中覆盖
	 * <p>The default implementation creates a
	 * {@link org.springframework.beans.factory.support.DefaultListableBeanFactory}
	 * with the {@linkplain #getInternalParentBeanFactory() internal bean factory} of this context's parent as parent bean factory. Can be overridden in subclasses,
	 * for example to customize DefaultListableBeanFactory's settings.
	 * @return the bean factory for this context
	 * @see org.springframework.beans.factory.support.DefaultListableBeanFactory#setAllowBeanDefinitionOverriding
	 * @see org.springframework.beans.factory.support.DefaultListableBeanFactory#setAllowEagerClassLoading
	 * @see org.springframework.beans.factory.support.DefaultListableBeanFactory#setAllowCircularReferences
	 * @see org.springframework.beans.factory.support.DefaultListableBeanFactory#setAllowRawInjectionDespiteWrapping
	 */
	protected DefaultListableBeanFactory createBeanFactory() {
		return new DefaultListableBeanFactory(getInternalParentBeanFactory());
	}



/**
如果它实现了ConfigurableApplicationContext则返回父上下文的内部bean工厂；否则返回父上下文自己
	 * Return the internal bean factory of the parent context if it implements
	 * ConfigurableApplicationContext; else, return the parent context itself.
	 * @see org.springframework.context.ConfigurableApplicationContext#getBeanFactory
	 */
	@Nullable
	protected BeanFactory getInternalParentBeanFactory() {
		return (getParent() instanceof ConfigurableApplicationContext ?
				((ConfigurableApplicationContext) getParent()).getBeanFactory() : getParent());
	}



/**
	定义这个上下文中的bean工厂
	 * Customize the internal bean factory used by this context.
	 * Called for each {@link #refresh()} attempt.
	 * <p>The default implementation applies this context's
	 * {@linkplain #setAllowBeanDefinitionOverriding "allowBeanDefinitionOverriding"}
	 * and {@linkplain #setAllowCircularReferences "allowCircularReferences"} settings,
	 * if specified. Can be overridden in subclasses to customize any of
	 * {@link DefaultListableBeanFactory}'s settings.
	 * @param beanFactory the newly created bean factory for this context
	 * @see DefaultListableBeanFactory#setAllowBeanDefinitionOverriding
	 * @see DefaultListableBeanFactory#setAllowCircularReferences
	 * @see DefaultListableBeanFactory#setAllowRawInjectionDespiteWrapping
	 * @see DefaultListableBeanFactory#setAllowEagerClassLoading
	 */
	protected void customizeBeanFactory(DefaultListableBeanFactory beanFactory) {
        //设置是否允许bean定义信息覆盖
		if (this.allowBeanDefinitionOverriding != null) {
			beanFactory.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
        //设置是否允许循环依赖
		if (this.allowCircularReferences != null) {
			beanFactory.setAllowCircularReferences(this.allowCircularReferences);
		}
	}
/**
设置允许beanfactory注册不同的定义信息相同的名称时覆盖bean定义信息，自动替换前者
	 * Set whether it should be allowed to override bean definitions by registering
	 * a different definition with the same name, automatically replacing the former.
	 如果不允许，一个异常将被抛出。这也适用于覆盖别名
	 * If not, an exception will be thrown. This also applies to overriding aliases.
	 * <p>Default is "true".
	 * @see #registerBeanDefinition
	 */
	public void setAllowBeanDefinitionOverriding(boolean allowBeanDefinitionOverriding) {
		this.allowBeanDefinitionOverriding = allowBeanDefinitionOverriding;
	}

/**
	设置是否允许多个bean之间存在循环依赖，并且自动解析他们。
	 * Set whether to allow circular references between beans - and automatically
	 * try to resolve them.
	注意：循环引用意味着所涉及的beans将会接收到另一个没有完全初始化的bean的引用
	 * <p>Note that circular reference resolution means that one of the involved beans
	 * will receive a reference to another bean that is not fully initialized yet.
	 这回导致初始化时产生微秒的负面影响
	 * This can lead to subtle and not-so-subtle side effects on initialization;
	 * it does work fine for many scenarios, though.
	 * <p>Default is "true". Turn this off to throw an exception when encountering
	 * a circular reference, disallowing them completely.
	 
	 注意：通常建议不要在多个beans之间依赖循环引用
	 * <p><b>NOTE:</b> It is generally recommended to not rely on circular references between your beans. 
	 
	 重构你的应用逻辑，使涉及的两个bean委托给封装了他们共同逻辑的第三个bean
	 Refactor your application logic to have the two beans
	 * involved delegate to a third bean that encapsulates their common logic.
	 */
	public void setAllowCircularReferences(boolean allowCircularReferences) {
		this.allowCircularReferences = allowCircularReferences;
	}


/**
	加载bean定义信息到给定的bean工厂，通常通过一个或多个beandefinitionreader
	 * Load bean definitions into the given bean factory, typically through
	 * delegating to one or more bean definition readers.
	 * @param beanFactory the bean factory to load bean definitions into
	 * @throws BeansException if parsing of the bean definitions failed
	 * @throws IOException if loading of bean definition files failed
	 * @see org.springframework.beans.factory.support.PropertiesBeanDefinitionReader
	 * @see org.springframework.beans.factory.xml.XmlBeanDefinitionReader
	 */
	protected abstract void loadBeanDefinitions(DefaultListableBeanFactory beanFactory)
			throws BeansException, IOException;
```

> #### AbstractXmlApplicationContext传统xml加载bean定义信息

```java
/**
通过XmlBeanDefinitionReader加载bean定义信息
 * Loads the bean definitions via an XmlBeanDefinitionReader.
 * @see org.springframework.beans.factory.xml.XmlBeanDefinitionReader
 * @see #initBeanDefinitionReader
 * @see #loadBeanDefinitions
 */
@Override
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    //给定一个beanfactory创建XmlBeanDefinitionReader
   // Create a new XmlBeanDefinitionReader for the given BeanFactory.
   XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

    //使用该上下文的资源加载环境配置bean定义阅读器
   // Configure the bean definition reader with this context's
   // resource loading environment.
   beanDefinitionReader.setEnvironment(this.getEnvironment());
   beanDefinitionReader.setResourceLoader(this);
   beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    //允许子类提供自定义初始化的reader，然后继续实际加载bean定义
   // Allow a subclass to provide custom initialization of the reader,
   // then proceed with actually loading the bean definitions.
   initBeanDefinitionReader(beanDefinitionReader);
   loadBeanDefinitions(beanDefinitionReader);
}

/**通过给定的XmlBeanDefinitionReader加载bean定义
	 * Load the bean definitions with the given XmlBeanDefinitionReader.
	 bean工厂的生命周期由refreshBeanFactory方法处理
	 * <p>The lifecycle of the bean factory is handled by the {@link #refreshBeanFactory} method;
     因此，该方法只是用于加载/注册bean定义信息
     hence this method is just supposed to load and/or register bean definitions.
	 * @param reader the XmlBeanDefinitionReader to use
	 * @throws BeansException in case of bean registration errors
	 * @throws IOException if the required XML document isn't found
	 * @see #refreshBeanFactory
	 * @see #getConfigLocations
	 * @see #getResources
	 * @see #getResourcePatternResolver
	 */
	protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
		Resource[] configResources = getConfigResources();
		if (configResources != null) {
			reader.loadBeanDefinitions(configResources);
		}
		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
			reader.loadBeanDefinitions(configLocations);
		}
	}
```

##### XmlBeanDefinitionReader

```java
/**
用于xml bean定义信息的bean定义阅读器
 * Bean definition reader for XML bean definitions.
 
 BeanDefinitionDocumentReader接口的实现，用于读取实际的xml文档
 * Delegates the actual XML document reading to an implementation
 * of the {@link BeanDefinitionDocumentReader} interface.
 *
 * <p>Typically applied to a
 * {@link org.springframework.beans.factory.support.DefaultListableBeanFactory}
 * or a {@link org.springframework.context.support.GenericApplicationContext}.
 *
 此类加载dom文档并且将BeanDefinitionDocumentReader应用于该文档
 * <p>This class loads a DOM document and applies the BeanDefinitionDocumentReader to it.
 文档阅读器为给定的bean工厂注册每一个bean定义，BeanDefinitionRegistry的实现
 * The document reader will register each bean definition with the given bean factory,
 * talking to the latter's implementation of the
 * {@link org.springframework.beans.factory.support.BeanDefinitionRegistry} interface.
 */
 public class XmlBeanDefinitionReader extends AbstractBeanDefinitionReader {}
```

> #### AnnotationConfigWebApplicationContext注解加载bean定义信息

```java
/**
给任何指定的类注册BeanDefinition，并且扫描任何指定的包。
 * Register a {@link org.springframework.beans.factory.config.BeanDefinition} for
 * any classes specified by {@link #register(Class...)} and scan any packages
 * specified by {@link #scan(String...)}.
 
 对于指定的任何路径值，尝试首次加载每一个路径作为一个类，如果类加载成功注册BeanDefinition,
 * <p>For any values specified by {@link #setConfigLocation(String)} or
 * {@link #setConfigLocations(String[])}, attempt first to load each location as a
 * class, registering a {@code BeanDefinition} if class loading is successful,
 
 并且如果一个类加载失败（ClassNotFoundException异常将会抛出）
 * and if class loading fails (i.e. a {@code ClassNotFoundException} is raised),
 假设该值是一个包并尝试去扫描它的组件类
 * assume the value is a package and attempt to scan it for component classes.
 
 启用默认的注解配置后置处理器集，例如@Autowired，@Required，关联的注解会被使用
 * <p>Enables the default set of annotation configuration post processors, such that
 * {@code @Autowired}, {@code @Required}, and associated annotations can be used.
 * <p>Configuration class bean definitions are registered with generated bean
 * definition names unless the {@code value} attribute is provided to the stereotype
 * annotation.
 * @param beanFactory the bean factory to load bean definitions into
 * @see #register(Class...)
 * @see #scan(String...)中·
 * @see #setConfigLocation(String)
 * @see #setConfigLocations(String[])
 * @see AnnotatedBeanDefinitionReader
 * @see ClassPathBeanDefinitionScanner
 */
@Override
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) {
   AnnotatedBeanDefinitionReader reader = getAnnotatedBeanDefinitionReader(beanFactory);
   ClassPathBeanDefinitionScanner scanner = getClassPathBeanDefinitionScanner(beanFactory);

   BeanNameGenerator beanNameGenerator = getBeanNameGenerator();
   if (beanNameGenerator != null) {
      reader.setBeanNameGenerator(beanNameGenerator);
      scanner.setBeanNameGenerator(beanNameGenerator);
      beanFactory.registerSingleton(AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR, beanNameGenerator);
   }

   ScopeMetadataResolver scopeMetadataResolver = getScopeMetadataResolver();
   if (scopeMetadataResolver != null) {
      reader.setScopeMetadataResolver(scopeMetadataResolver);
      scanner.setScopeMetadataResolver(scopeMetadataResolver);
   }

   if (!this.componentClasses.isEmpty()) {
      if (logger.isDebugEnabled()) {
         logger.debug("Registering component classes: [" +
               StringUtils.collectionToCommaDelimitedString(this.componentClasses) + "]");
      }
      reader.register(ClassUtils.toClassArray(this.componentClasses));
   }

   if (!this.basePackages.isEmpty()) {
      if (logger.isDebugEnabled()) {
         logger.debug("Scanning base packages: [" +
               StringUtils.collectionToCommaDelimitedString(this.basePackages) + "]");
      }
      scanner.scan(StringUtils.toStringArray(this.basePackages));
   }

   String[] configLocations = getConfigLocations();
   if (configLocations != null) {
      for (String configLocation : configLocations) {
         try {
            Class<?> clazz = ClassUtils.forName(configLocation, getClassLoader());
            if (logger.isTraceEnabled()) {
               logger.trace("Registering [" + configLocation + "]");
            }
            reader.register(clazz);
         }
         catch (ClassNotFoundException ex) {
            if (logger.isTraceEnabled()) {
               logger.trace("Could not load class for config location [" + configLocation +
                     "] - trying package scan. " + ex);
            }
            int count = scanner.scan(configLocation);
            if (count == 0 && logger.isDebugEnabled()) {
               logger.debug("No component classes found for specified class/package [" + configLocation + "]");
            }
         }
      }
   }
}
```

#### 3、prepareBeanFactory方法

```java
// Prepare the bean factory for use in this context.
 //准备在这个上下文中使用的bean工厂
prepareBeanFactory(beanFactory);
```

```java
/**
配置工厂的标准上下文特点
	 * Configure the factory's standard context characteristics,
	 比如上下文的类加载器和后置处理器
	 * such as the context's ClassLoader and post-processors.
	 * @param beanFactory the BeanFactory to configure
	 */
	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        //告诉内部bean工厂去使用上下文的Class loader等等
		// Tell the internal bean factory to use the context's class loader etc.
        //设置classloader
		beanFactory.setBeanClassLoader(getClassLoader());
        //设置表达式解析器，用于解析Spel
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
        //设置源解析
		beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

		// Configure the bean factory with context callbacks.
        //添加bean后置处理器
		beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
        //自动装配时忽略给定的依赖接口
		beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
		beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
		beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
		beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
		beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

		// BeanFactory interface not registered as resolvable type in a plain factory.
		// MessageSource registered (and found for autowiring) as a bean.
		beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
		beanFactory.registerResolvableDependency(ResourceLoader.class, this);
		beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
		beanFactory.registerResolvableDependency(ApplicationContext.class, this);

		// Register early post-processor for detecting inner beans as ApplicationListeners.
        //添加事件监听
		beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

        //如果侦测到LoadTimeWeaver，准备织入---aop
		// Detect a LoadTimeWeaver and prepare for weaving, if found.
		if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			// Set a temporary ClassLoader for type matching.
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}
		//注册进默认的环境bean
		// Register default environment beans.
		if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
		}
		if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
			beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
		}
	}
```

#### 4、postProcessBeanFactory

```java
//允许在上下文子类对bean工厂进行后置处理
// Allows post-processing of the bean factory in context subclasses.
postProcessBeanFactory(beanFactory);
```

```java
/**
	 * Register request/session scopes, a {@link ServletContextAwareProcessor}, etc.
	 */
	@Override
	protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
		beanFactory.addBeanPostProcessor(new ServletContextAwareProcessor(this.servletContext, this.servletConfig));
		beanFactory.ignoreDependencyInterface(ServletContextAware.class);
		beanFactory.ignoreDependencyInterface(ServletConfigAware.class);

		WebApplicationContextUtils.registerWebApplicationScopes(beanFactory, this.servletContext);
		WebApplicationContextUtils.registerEnvironmentBeans(beanFactory, this.servletContext, this.servletConfig);
	}
```

#### 5、invokeBeanFactoryPostProcessors

```java
//调用在上下文中注册为bean 的工厂处理器
// Invoke factory processors registered as beans in the context.
invokeBeanFactoryPostProcessors(beanFactory);
```

```java
/**
	实例化并调用所有的注册的BeanFactoryPostProcessor beans
	 * Instantiate and invoke all registered BeanFactoryPostProcessor beans,
	 如果有给定顺序，遵守顺序执行
	 * respecting explicit order if given.
	 必须在单例实例化之前调用（因为工厂增强是为了实例化用的）
	 * <p>Must be called before singleton instantiation.
	 */
	protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
 //代表AbstractApplicationContext的后置处理器执行       
        PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
//检测LoadTimeWeaver并准备编织（如果在此期间发现）
		// Detect a LoadTimeWeaver and prepare for weaving, if found in the meantime
		// (e.g. through an @Bean method registered by ConfigurationClassPostProcessor)
		if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
			beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
			beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
		}
	}
```

```java
public static void invokeBeanFactoryPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

		// Invoke BeanDefinitionRegistryPostProcessors first, if any.
		Set<String> processedBeans = new HashSet<>();
		//工厂默认创建的是DefaultListableBeanFactory，DefaultListableBeanFactory是BeanDefinitionRegistry实现类，bean定义信息都在BeanDefinitionRegistry中，工厂是包含BeanDefinitionRegistry
		if (beanFactory instanceof BeanDefinitionRegistry) {
			BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
            //这里准备对后置处理器进行分类，创建连个不同的集合
		List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();
			//循环后置处理器
		for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
            //判断后置处理器属于哪种，BeanDefinitionRegistryPostProcessor在BeanFactoryPostProcessor之前执行，可以用来注册其它的bean定义
				if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
					BeanDefinitionRegistryPostProcessor registryProcessor =
							(BeanDefinitionRegistryPostProcessor) postProcessor;
                    
       //BeanDefinitionRegistryPostProcessor就是在此执行的，注册其它bean定义信息
					registryProcessor.postProcessBeanDefinitionRegistry(registry);
					registryProcessors.add(registryProcessor);
				}
            //常规的BeanFactoryPostProcessor
				else {
					regularPostProcessors.add(postProcessor);
				}
			}
	//这里不要实例化工厂bean，需要保留所有的常规bean
			// Do not initialize FactoryBeans here: We need to leave all regular beans
        //未初始化，让bean工厂后置处理器应用他们
			// uninitialized to let the bean factory post-processors apply to them!
在实现PriorityOrdered，Ordered和其余优先级的BeanDefinitionRegistryPostProcessor之间分开。
			// Separate between BeanDefinitionRegistryPostProcessors that implement
			// PriorityOrdered, Ordered, and the rest.
			List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();
            
 //首先，调用实现PriorityOrdered的BeanDefinitionRegistryPostProcessors
			// First, invoke the BeanDefinitionRegistryPostProcessors that implement PriorityOrdered.
            
            //得到BeanDefinitionRegistryPostProcessor的实现类的名称
			String[] postProcessorNames =
					beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
				if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
         //将实例化的BeanDefinitionRegistryPostProcessor加入到currentRegistryProcessors
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
            //排序
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			currentRegistryProcessors.clear();
            
下一步，调用实现Ordered的BeanDefinitionRegistryPostProcessors
			// Next, invoke the BeanDefinitionRegistryPostProcessors that implement Ordered.
			postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
				if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			currentRegistryProcessors.clear();

            //最后，调用所有其他BeanDefinitionRegistryPostProcessor，直到不再出现
			// Finally, invoke all other BeanDefinitionRegistryPostProcessors until no further ones appear.
			boolean reiterate = true;
			while (reiterate) {
				reiterate = false;
				postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
				for (String ppName : postProcessorNames) {
					if (!processedBeans.contains(ppName)) {
						currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
						processedBeans.add(ppName);
						reiterate = true;
					}
				}
				sortPostProcessors(currentRegistryProcessors, beanFactory);
				registryProcessors.addAll(currentRegistryProcessors);
				invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
				currentRegistryProcessors.clear();
			}

			// Now, invoke the postProcessBeanFactory callback of all processors handled so far.
			invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
			invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
		}

		else {
			// Invoke factory processors registered with the context instance.
			invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
		}

		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let the bean factory post-processors apply to them!
		String[] postProcessorNames =
				beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

		// Separate between BeanFactoryPostProcessors that implement PriorityOrdered,
		// Ordered, and the rest.
		List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
		List<String> orderedPostProcessorNames = new ArrayList<>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
		for (String ppName : postProcessorNames) {
			if (processedBeans.contains(ppName)) {
				// skip - already processed in first phase above
			}
			else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		// First, invoke the BeanFactoryPostProcessors that implement PriorityOrdered.
		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

		// Next, invoke the BeanFactoryPostProcessors that implement Ordered.
		List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
		for (String postProcessorName : orderedPostProcessorNames) {
			orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

		// Finally, invoke all other BeanFactoryPostProcessors.
		List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
		for (String postProcessorName : nonOrderedPostProcessorNames) {
			nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

		// Clear cached merged bean definitions since the post-processors might have
		// modified the original metadata, e.g. replacing placeholders in values...
		beanFactory.clearMetadataCache();
	}
```

#### 6、registerBeanPostProcessors

```java
//注册拦截Bean创建的Bean处理器
// Register bean processors that intercept bean creation.
registerBeanPostProcessors(beanFactory);
```

#### 7、initMessageSource

```java
//为此上下文初始化消息源  典型应用国际化
// Initialize message source for this context.
				initMessageSource();
```

#### 8、initApplicationEventMulticaster

```java
//初始化事件多播器，用于通知所有的观察者对象
//ApplicationListener,监听容器中发布的事件，只要事件发生，就触发监听器的回调，来完成事件驱动开发。属于观察者设计模式中的Observer对象。
// Initialize event multicaster for this context.
				initApplicationEventMulticaster();
```

#### 9、onRefresh

```java
//在特定上下文子类中初始化其他特殊bean
// Initialize other special beans in specific context subclasses.
onRefresh();


/**
可以重写的模板方法以添加特定于上下文的刷新工作
	 * Template method which can be overridden to add context-specific refresh work.
	 
	 在实例化单例之前调用特殊bean的初始化
	 * Called on initialization of special beans, before instantiation of singletons.
	 * <p>This implementation is empty.
	 * @throws BeansException in case of errors
	 * @see #refresh()
	 */
	protected void onRefresh() throws BeansException {
		// For subclasses: do nothing by default.
	}
```

#### 10、registerListeners

```java
//检查监听器bean并注册它们
// Check for listener beans and register them.
registerListeners();
```

#### 11、finishBeanFactoryInitialization

```java
//实例化所有剩余的（非懒加载初始化）单例。
// Instantiate all remaining (non-lazy-init) singletons.
finishBeanFactoryInitialization(beanFactory);
```

```java
/**
 * Finish the initialization of this context's bean factory,
 * initializing all remaining singleton beans.
 */
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    //初始化转换服务
   // Initialize conversion service for this context.
   if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
         beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
      beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
   }

    //如果没有bean后处理器，则注册默认的嵌入式值解析器
   // Register a default embedded value resolver if no bean post-processor
   // (such as a PropertyPlaceholderConfigurer bean) registered any before:
    
    //此时主要用于解析注释属性值
   // at this point, primarily for resolution in annotation attribute values.
   if (!beanFactory.hasEmbeddedValueResolver()) {
      beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
   }

   // Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
   String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
   for (String weaverAwareName : weaverAwareNames) {
      getBean(weaverAwareName);
   }

   // Stop using the temporary ClassLoader for type matching.
   beanFactory.setTempClassLoader(null);

   // Allow for caching all bean definition metadata, not expecting further changes.
   beanFactory.freezeConfiguration();
	//实例化所有保留的单例
   // Instantiate all remaining (non-lazy-init) singletons.
   beanFactory.preInstantiateSingletons();
}
```

```java
@Override
public void preInstantiateSingletons() throws BeansException {
   if (logger.isTraceEnabled()) {
      logger.trace("Pre-instantiating singletons in " + this);
   }

    //遍历一个副本允许使用初始化方法，这些方法一次注册新的bean定义信息
   // Iterate over a copy to allow for init methods which in turn register new bean definitions.
   // While this may not be part of the regular factory bootstrap, it does otherwise work fine.
   List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
	//触发初始化所有非懒加载的单例beans
   // Trigger initialization of all non-lazy singleton beans...
   for (String beanName : beanNames) {
       //获得合并后的RootBeanDefinition，
      RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
       //如果这个bean定义不是抽象，是单例，不是懒加载的
      if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
          //判断是不是工厂bean
         if (isFactoryBean(beanName)) {
             //获取工厂bean
            Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
             //判断是不是属于FactoryBean
            if (bean instanceof FactoryBean) {
               final FactoryBean<?> factory = (FactoryBean<?>) bean;
               boolean isEagerInit;
               if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                  isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                              ((SmartFactoryBean<?>) factory)::isEagerInit,
                        getAccessControlContext());
               }
               else {
                  isEagerInit = (factory instanceof SmartFactoryBean &&
                        ((SmartFactoryBean<?>) factory).isEagerInit());
               }
               if (isEagerInit) {
                  getBean(beanName);
               }
            }
         }
         else {
             //核心
            getBean(beanName);
         }
      }
   }

   // Trigger post-initialization callback for all applicable beans...
   for (String beanName : beanNames) {
      Object singletonInstance = getSingleton(beanName);
      if (singletonInstance instanceof SmartInitializingSingleton) {
         final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
         if (System.getSecurityManager() != null) {
            AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
               smartSingleton.afterSingletonsInstantiated();
               return null;
            }, getAccessControlContext());
         }
         else {
            smartSingleton.afterSingletonsInstantiated();
         }
      }
   }
}
```

```java
/**
返回一个实例，该实例可以是指定bean的共享的或独立的
 * Return an instance, which may be shared or independent, of the specified bean.
 * @param name the name of the bean to retrieve
 * @param requiredType the required type of the bean to retrieve
 * @param args arguments to use when creating a bean instance using explicit arguments
 * (only applied when creating a new instance as opposed to retrieving an existing one)
 * @param typeCheckOnly whether the instance is obtained for a type check,
 * not for actual use
 * @return an instance of the bean
 * @throws BeansException if the bean could not be created
 */
@SuppressWarnings("unchecked")
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
      @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
	//解析为规范名称
   final String beanName = transformedBeanName(name);
   Object bean;

    //认真检查单例缓存是否有手动注册的单例。
   // Eagerly check singleton cache for manually registered singletons.
   Object sharedInstance = getSingleton(beanName);
   if (sharedInstance != null && args == null) {
      if (logger.isTraceEnabled()) {
         if (isSingletonCurrentlyInCreation(beanName)) {
            logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
                  "' that is not fully initialized yet - a consequence of a circular reference");
         }
         else {
            logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
         }
      }
      bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
   }

   else {
      // Fail if we're already creating this bean instance:
      // We're assumably within a circular reference.
      if (isPrototypeCurrentlyInCreation(beanName)) {
         throw new BeanCurrentlyInCreationException(beanName);
      }

      // Check if bean definition exists in this factory.
       //这里获取父Bean工厂
      BeanFactory parentBeanFactory = getParentBeanFactory();
      if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
         // Not found -> check parent.
         String nameToLookup = originalBeanName(name);
         if (parentBeanFactory instanceof AbstractBeanFactory) {
             //递归查找bean定义是否存在这个工厂
            return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
                  nameToLookup, requiredType, args, typeCheckOnly);
         }
         else if (args != null) {
            // Delegation to parent with explicit args.
            return (T) parentBeanFactory.getBean(nameToLookup, args);
         }
         else if (requiredType != null) {
            // No args -> delegate to standard getBean method.
            return parentBeanFactory.getBean(nameToLookup, requiredType);
         }
         else {
            return (T) parentBeanFactory.getBean(nameToLookup);
         }
      }

       //标记这个指定bean已经创建
       //Mark the specified bean as already created (or about to be created).
      if (!typeCheckOnly) {
         markBeanAsCreated(beanName);
      }

      try {
          //获取bean定义信息
         final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
          //检查给定的合并bean定义，有可能引发验证异常。
         checkMergedBeanDefinition(mbd, beanName, args);

          //确保当前bean依赖的bean的初始化
         // Guarantee initialization of beans that the current bean depends on.
         String[] dependsOn = mbd.getDependsOn();
         if (dependsOn != null) {
             //遍历所有依赖的bean,并初始化
            for (String dep : dependsOn) {
               if (isDependent(beanName, dep)) {
                  throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                        "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
               }
               registerDependentBean(dep, beanName);
               try {
                   //初始化
                  getBean(dep);
               }
               catch (NoSuchBeanDefinitionException ex) {
                  throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                        "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
               }
            }
         }

         // Create bean instance.
         if (mbd.isSingleton()) {
            sharedInstance = getSingleton(beanName, () -> {
               try {
               //createBean创建bean-->doCreateBean
                  return createBean(beanName, mbd, args);
               }
               catch (BeansException ex) {
                  // Explicitly remove instance from singleton cache: It might have been put there
                  // eagerly by the creation process, to allow for circular reference resolution.
                  // Also remove any beans that received a temporary reference to the bean.
                  destroySingleton(beanName);
                  throw ex;
               }
            });
            bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
         }

         else if (mbd.isPrototype()) {
            // It's a prototype -> create a new instance.
            Object prototypeInstance = null;
            try {
               beforePrototypeCreation(beanName);
               prototypeInstance = createBean(beanName, mbd, args);
            }
            finally {
               afterPrototypeCreation(beanName);
            }
            bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
         }

         else {
            String scopeName = mbd.getScope();
            final Scope scope = this.scopes.get(scopeName);
            if (scope == null) {
               throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
            }
            try {
               Object scopedInstance = scope.get(beanName, () -> {
                  beforePrototypeCreation(beanName);
                  try {
                     return createBean(beanName, mbd, args);
                  }
                  finally {
                     afterPrototypeCreation(beanName);
                  }
               });
               bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
            }
            catch (IllegalStateException ex) {
               throw new BeanCreationException(beanName,
                     "Scope '" + scopeName + "' is not active for the current thread; consider " +
                     "defining a scoped proxy for this bean if you intend to refer to it from a singleton",
                     ex);
            }
         }
      }
      catch (BeansException ex) {
         cleanupAfterBeanCreationFailure(beanName);
         throw ex;
      }
   }
```

#### 12、finishRefresh

```java
//发布相应事件
// Last step: publish corresponding event.
finishRefresh();


/**
结束此上下文的刷新，
	 * Finish the refresh of this context, invoking the LifecycleProcessor's
	 * onRefresh() method and publishing the
	 * {@link org.springframework.context.event.ContextRefreshedEvent}.
	 */
	protected void finishRefresh() {
		// Clear context-level resource caches (such as ASM metadata from scanning).
		clearResourceCaches();

		// Initialize lifecycle processor for this context.
		initLifecycleProcessor();

		// Propagate refresh to lifecycle processor first.
		getLifecycleProcessor().onRefresh();

		// Publish the final event.
		publishEvent(new ContextRefreshedEvent(this));

		// Participate in LiveBeansView MBean, if active.
		LiveBeansView.registerApplicationContext(this);
	}
```
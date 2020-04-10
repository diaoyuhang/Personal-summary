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
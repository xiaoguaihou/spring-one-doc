======================

   * [springboot框架关于bean装配](#springboot框架关于bean装配)
      * [1 装配的目的](#1-装配的目的)
      * [2 实现方式](#2-实现方式)
         * [2.1 bean加载的解耦](#21-bean加载的解耦)
         * [2.2 配置参数的处理](#22-配置参数的处理)
         * [2.3 xml vs java annotation](#23-xml-vs-java-annotation)
      * [3 详解](#3-详解)
         * [3.0 IoC Container](#30-ioc-container)
         * [3.1 bean的定义](#31-bean的定义)
            * [Class的实例化](#class的实例化)
            * [DI](#di)
            * [Bean Scopes](#bean-scopes)
            * [Autowiring Mode](#autowiring-mode)
         * [3.2 annotation-based container configuration](#32-annotation-based-container-configuration)
         * [3.3 Spring inject or JSR-330](#33-spring-inject-or-jsr-330)
         * [3.4 Java-based Container Configuration](#34-java-based-container-configuration)
      * [4 Environment Abstraction](#4-environment-abstraction)
      * [5 data binding](#5-data-binding)
         * [5.1 @Value](#51-value)
         * [5.2 @ConfigurationProperties](#52-configurationproperties)
	 * [5.3 Demo](#53-demo)
	 

先看[5.3 Demo](#53-demo),如果非常了解，那就不必看此篇文章了～～

# springboot框架关于bean装配
framework这东西就是定义一套规范，使用者遵循这套规范，将会等到很大好处：加快开发效率、提高代码可维护性...

所以，我们要使用好framework，务必要了解他的规则，和定义这套规则的目的。否则，用起来，总是处于东施效颦的状态

## 1 装配的目的
- bean加载的解耦
- 配置参数的处理

## 2 实现方式
### 2.1 bean加载的解耦
加载的可选方式
- 类工厂：属于基本的设计模式，实现范畴，不能支撑框架
- 配置文件，xml等。spring 开始的做法
- java的anotation。Annotation-based,spring 2.5.6. Java-based configuration, Spring 3.0
### 2.2 配置参数的处理
- 代码写死。太土，框架不考虑
- 配置文件。配置大量的xml文件。spring开始真的做法
- 配置文件+代码。到了springboot，就是一个applicaiton.yml加上java annotation就可以搞定了。

### 2.3 xml vs java annotation
- 二者最终的结果，是一摸一样的
- xml，对于人的阅读很友好，语义表达很充分。但是配置起来比较烦人
- java annotation，需要你花不少时间先去研究他的用法。研究明白后，用起来很简便。

## 3 详解
### 3.0 IoC Container
- *org.springframework.context.ApplicationContext* 是IoC容器。负责bean的初始化，配置和组装（instantiating, configuring, and assembling）
- 容器怎么知道要负责哪些，具体的配置参数是什么呢: configuration metadata。(The container gets its instructions on what objects to instantiate, configure, and assemble by reading configuration metadata)
- The configuration metadata is represented in 
  - XML
  - Java annotations. From Spring 2.5
  - Java-based configuration. From Spring 3.0
- 典型的XML based ApplicationContext
  - ClassPathXmlApplicationContext
  - FileSystemXmlApplicationContext
```
<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-
    value>
    </context-param>
    <listener>
        <listener-class>
    org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
```
```
     // create and configure beans
    ApplicationContext context = new ClassPathXmlApplicationContext("services.xml",
    "daos.xml");
    // retrieve configured instance
    PetStoreService service = context.getBean("petStore", PetStoreService.class);
    // use configured instance
    List<String> userList = service.getUsernameList();
```

```
                -----------------
                |                |
--------------->|   The Spring   |<----------------------------
Configuration   |    Container   | Your Business Objects(POJOS)
  Metadata      |________________|
                        |
                        | produces
            ____________V____________
           |                         |
           | Fully Configured System |
           |  Ready for Use          |
           |_________________________|             
                        
```
### 3.1 bean的定义

- Class: instantiating beans
- Name
- Scope
- Constructor arguments : Dependency Injection
- Properties : Dependency Injection
- Autowiring mode
- Lzay initialization mode
- Initialization method
- Destruction method

#### Class的实例化
- 构造方法
```
<bean id="exampleBean" class="examples.ExampleBean"/>
```
- 静态工厂方法
```
<bean id="clientService"
        class="examples.ClientService"
        factory-method="createInstance"/>
</bean>
```
```
public class ClientService {
        private static ClientService clientService = new ClientService();
        private ClientService() {}
        public static ClientService createInstance() {
            return clientService;
} }
```
- 实例的工厂类
```
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
        <!-- inject any dependencies required by this locator bean -->
</bean>
    <bean id="clientService"
        factory-bean="serviceLocator"
        factory-method="createClientServiceInstance"/>
    <bean id="accountService"
        factory-bean="serviceLocator"
        factory-method="createAccountServiceInstance"/>
...
```

```
public class DefaultServiceLocator {
        private static ClientService clientService = new ClientServiceImpl();
        private static AccountService accountService = new AccountServiceImpl();
        public ClientService createClientServiceInstance() {
            return clientService;
}
        public AccountService createAccountServiceInstance() {
            return accountService;
} }
```
#### DI
- Constructor-based Dependency Injection
```
reference

<beans>
        <bean id="thingOne" class="x.y.ThingOne">
            <constructor-arg ref="thingTwo"/>
            <constructor-arg ref="thingThree"/>
        </bean>
        <bean id="thingTwo" class="x.y.ThingTwo"/>
        <bean id="thingThree" class="x.y.ThingThree"/>
</beans>
```

```
by parameter type
----------------------

<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean

by parameter index
----------------
<bean id="exampleBean" class="examples.ExampleBean">
        <constructor-arg index="0" value="7500000"/>
        <constructor-arg index="1" value="42"/>
</bean>

by parameter name
----------------
<bean id="exampleBean" class="examples.ExampleBean">
        <constructor-arg name="years" value="7500000"/>
        <constructor-arg name="ultimateAnswer" value="42"/>
</bean>


use p, c namespace
-----------------
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:c="http://www.springframework.org/schema/c"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id="thingOne" class="x.y.ThingTwo"/>
        <bean id="thingTwo" class="x.y.ThingThree"/>
        <!-- traditional declaration -->
        <bean id="thingOne" class="x.y.ThingOne">
            <constructor-arg ref="thingTwo"/>
            <constructor-arg ref="thingThree"/>
            <constructor-arg value="something@somewhere.com"/>
</bean>
        <!-- c-namespace declaration -->
        <bean id="thingOne" class="x.y.ThingOne" c:thingTwo-ref="thingTwo"
    c:thingThree-ref="thingThree" c:email="something@somewhere.com"/>
</beans>
```
- Setter-based Dependency Injection
```
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-
    method="close">
        <!-- results in a setDriverClassName(String) call -->
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
        <property name="username" value="root"/>
        <property name="password" value="masterkaoli"/>
</bean>
```

```
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:p="http://www.springframework.org/schema/p"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
            destroy-method="close"
            p:driverClassName="com.mysql.jdbc.Driver"
            p:url="jdbc:mysql://localhost:3306/mydb"
            p:username="root"
            p:password="masterkaoli"/>
</beans>
```


#### Bean Scopes

Scope | explain
---|---
singleton | default
prototype | Scopes a single bean definition to any number of object instances.
request | Only valid in the context of a web-aware Spring ApplicationContext.
session | Scopes a single bean definition to the lifecycle of an HTTP Session. Only valid in the context of a web-aware Spring ApplicationContext.
application | Scopes a single bean definition to the lifecycle of a ServletContext. Only valid in the context of a web-aware Spring ApplicationContext.
websocket | Scopes a single bean definition to the lifecycle of a WebSocket. Only valid in the context of a web-aware Spring ApplicationContext.

==when a singleton bean has a prototype bean property!!!==

#### Autowiring Mode

Mode | Explaination
---|---
no | Default) No autowiring. Bean references must be defined by ref elements. Changing the default setting is not recommended for larger deployments, because specifying collaborators explicitly gives greater control and clarity. To some extent, it documents the structure of a system.
byName | Autowiring by property name. Spring looks for a bean with the same name as the property that needs to be autowired. For example, if a bean definition is set to autowire by name and it contains a master property (that is, it has a setMaster(..) method), Spring looks for a bean definition named master and uses it to set the property.
byType | Lets a property be autowired if exactly one bean of the property type exists in the container. If more than one exists, a fatal exception is thrown, which indicates that you may not use byType autowiring for that bean. If there are no matching beans, nothing happens (the property is not set).
constructor | Analogous to byType but applies to constructor arguments. If there is not exactly one bean of the constructor argument type in the container, a fatal error is raised.






### 3.2 annotation-based container configuration
- @Componet + @ComponentScan + @Configuration
```
@Configuration
    @ComponentScan(basePackages = "org.example",
            includeFilters = @Filter(type = FilterType.REGEX, pattern =
    ".*Stub.*Repository"),
            excludeFilters = @Filter(Repository.class))
    public class AppConfig {
 }

```
等价与
```
<beans>
    <context:component-scan base-package="org.example">
            <context:include-filter type="regex"
                    expression=".*Stub.*Repository"/>
            <context:exclude-filter type="annotation"
                    expression="org.springframework.stereotype.Repository"/>
        </context:component-scan>
</beans>
```

### 3.3 Spring inject or JSR-330

Spring  &nbsp; | javax.inject.*  &nbsp; | comments
---|---|---
@Autowired | @Inject | @Inject has no 'required' attribute. Can be used with Java 8’s Optional instead.
@Component | @Named / @ManagedBean | SR-330 does not provide a composable model, only a way to identify named components.
@Scope("singleton") | @Singleton | The JSR-330 default scope is like Spring’s prototype. However, in order to keep it consistent with Spring’s general defaults, a JSR- 330 bean declared in the Spring container is a singleton by default. In order to use a scope other than singleton, you should use Spring’s @Scope annotation. javax.inject also provides a @Scope annotation. Nevertheless, this one is only intended to be used for creating your own annotations.
@Qualifier | @Qualifier / @Named | javax.inject.Qualifier is just a meta-annotation for building custom qualifiers. Concrete String qualifiers (like Spring’s @Qualifier with a value) can be associated through javax.inject.Named.
@Value | - | -
@Required | - | -
@Lazy | - | -
ObjectFactory | Provider | javax.inject.Provider is a direct alternative to Spring’s ObjectFactory, only with a shorter get() method name. It can also be used in combination with Spring’s @Autowired or with non-annotated constructors and setter methods.

- Autowired vs Resource vs Inject
- 

Annotation | Target | source | rule
---|---|---|---
@Autowired | {ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE}| spring | type->qualifier->name
@Resource | {TYPE, FIELD, METHOD} | java | name->type->qualifier
@Inject | { METHOD, CONSTRUCTOR, FIELD } | Java need import jar|type->qualifier->name


Both @Autowired (or @Inject) and @Resource work equally well. But there is a conceptual difference or a difference in the meaning

- @Resource means get me a known resource by name. The name is extracted from the name of the annotated setter or field, or it is taken from the name-Parameter.

- @Inject or @Autowired try to wire in a suitable other component by type.

So, basically these are two quite distinct concepts. Unfortunately the Spring-Implementation of @Resource has a built-in fallback, which kicks in when resolution by-name fails. In this case, it falls back to the @Autowired-kind resolution by-type. While this fallback is convenient, IMHO it causes a lot of confusion, because people are unaware of the conceptual difference and tend to use @Resource for type-based autowiring.



### 3.4 Java-based Container Configuration

- AnnotationConfigApplicationContext

This allows for completely XML-free usage of the Spring container: @Configuration, and @Componet or JSR-330 annotated class.
```
public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl
    .class, Dependency1.class, Dependency2.class);
        MyService myService = ctx.getBean(MyService.class);
        myService.doStuff();
}

----
public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new
    AnnotationConfigApplicationContext();
        ctx.register(AppConfig.class, OtherConfig.class);
        ctx.register(AdditionalConfig.class);
        ctx.refresh();
        MyService myService = ctx.getBean(MyService.class);
        myService.doStuff();
    }
    
-----
public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new
    AnnotationConfigApplicationContext();
        ctx.scan("com.acme");
        ctx.refresh();
        MyService myService = ctx.getBean(MyService.class);
```

- @bean

```
@Configuration
public class AppConfig {
    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    } 
}
```
等同于 (transferService -> com.acme.TransferServiceImpl)
```
@Configuration
public class AppConfig {
    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    } 
}
```

- lifecycle callbacks

Any classes defined with the @Bean annotation support the regular lifecycle callbacks and can use the @PostConstruct and @PreDestroy annotations from JSR-250. See JSR-250 annotations for further details.

The regular Spring lifecycle callbacks are fully supported as well. If a bean implements InitializingBean, DisposableBean, or Lifecycle, their respective methods are called by the container.

The @Bean annotation supports specifying arbitrary initialization and destruction callback methods, much like Spring XML’s init-method and destroy-method attributes on the bean element, as the following example shows:

```
public class BeanOne {
    public void init() {
        // initialization logic
} }
public class BeanTwo {
    public void cleanup() {
        // destruction logic
} }

@Configuration
public class AppConfig {
    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }
    
    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
    
    // disable the default (inferred) mode
    @Bean(destroyMethod="")
    public DataSource dataSource() throws NamingException {
        return (DataSource) jndiTemplate.lookup("MyDS");
    }
    
    @Bean(name="myencryptor")
    @Scope("prototype")
    public Encryptor encryptor() {
    }
    
    @Bean(name = { "dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

- injecting inter-bean dependencies

SimpleBean is singleton.

If you use @Configuration, all methods marked as @Bean will be wrapped into a CGLIB wrapper which works as if it’s the first call of this method, then the original method’s body will be executed and the resulting object will be registered in the spring context. All further calls just return the bean retrieved from the context.

```
@Configuration
public static class Config {

    @Bean
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Bean
    public SimpleBeanConsumer simpleBeanConsumer() {
        return new SimpleBeanConsumer(simpleBean());
    }
}
```

==SimpleBean is not singleton.==

```
@Component
public static class Config {

    @Bean
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Bean
    public SimpleBeanConsumer simpleBeanConsumer() {
        return new SimpleBeanConsumer(simpleBean());
    }
}
```
In the second code block above, new SimpleBeanConsumer(simpleBean()) just calls a pure java method. To correct the second code block, we can do something like this:

```
@Component
public static class Config {
    @Autowired
    SimpleBean simpleBean;

    @Bean
    public SimpleBean simpleBean() {
        return new SimpleBean();
    }

    @Bean
    public SimpleBeanConsumer simpleBeanConsumer() {
        return new SimpleBeanConsumer(simpleBean);
    }
}
```

一句话概括就是 @Configuration 中所有带 @Bean 注解的方法都会被动态代理，因此调用该方法返回的都是同一个实例。

- import
```
@Configuration
    public class ConfigA {
@Bean
        public A a() {
            return new A();
} }
    @Configuration
    @Import(ConfigA.class)
    public class ConfigB {
@Bean
        public B b() {
            return new B();
} }

---
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);
    B b = ctx.getBean(B.class);
}
```

## 4 Environment Abstraction
The Environment interface is an abstraction integrated in the container that models two key aspects of the application environment: profiles and properties.

A profile is a named, logical group of bean definitions to be registered with the container only if the given profile is active. Beans may be assigned to a profile whether defined in XML or with annotations. The role of the Environment object with relation to profiles is in determining which profiles (if any) are currently active, and which profiles (if any) should be active by default.

Properties play an important role in almost all applications and may originate from a variety of sources: properties files, JVM system properties, system environment variables, JNDI, servlet context parameters, ad-hoc Properties objects, Map objects, and so on. The role of the Environment object with relation to properties is to provide the user with a convenient service interface for configuring property sources and resolving properties from them

In most cases, however, application-level beans should not need to interact with the Environment directly but instead may have to have ${...} property values replaced by a property placeholder configurer such as PropertySourcesPlaceholderConfigurer, which itself is EnvironmentAware and as of Spring 3.1 is registered by default when using <context:property-placeholder/>.


- PropertySource Abstraction
The @PropertySource annotation provides a convenient and declarative mechanism for adding a
PropertySource to Spring’s Environment.
```
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {
    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

- yml
@PropertySource 不支持yml
```
@Configuration
public class AppConfig {
    @Bean
    public static PropertySourcesPlaceholderConfigurer properties() {
        PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer = new PropertySourcesPlaceholderConfigurer();
        YamlPropertiesFactoryBean yaml = new YamlPropertiesFactoryBean();
        yaml.setResources(new ClassPathResource("you.yml"));
        propertySourcesPlaceholderConfigurer.setProperties(yaml.getObject());
        return propertySourcesPlaceholderConfigurer;
    }
}
```

## 5 data binding
### 5.1 @Value
```

```

### 5.2 @ConfigurationProperties
Springboot 2.0后更合理了。专注于data binding

```
pre spring 2.0
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ConfigurationProperties {

	/**
	 * The name prefix of the properties that are valid to bind to this object. Synonym
	 * for {@link #prefix()}.
	 * @return the name prefix of the properties to bind
	 */
	@AliasFor("prefix")
	String value() default "";

	/**
	 * The name prefix of the properties that are valid to bind to this object. Synonym
	 * for {@link #value()}.
	 * @return the name prefix of the properties to bind
	 */
	@AliasFor("value")
	String prefix() default "";

	/**
	 * Flag to indicate that when binding to this object invalid fields should be ignored.
	 * Invalid means invalid according to the binder that is used, and usually this means
	 * fields of the wrong type (or that cannot be coerced into the correct type).
	 * @return the flag value (default false)
	 */
	boolean ignoreInvalidFields() default false;

	/**
	 * Flag to indicate that when binding to this object fields with periods in their
	 * names should be ignored.
	 * @return the flag value (default false)
	 */
	boolean ignoreNestedProperties() default false;

	/**
	 * Flag to indicate that when binding to this object unknown fields should be ignored.
	 * An unknown field could be a sign of a mistake in the Properties.
	 * @return the flag value (default true)
	 */
	boolean ignoreUnknownFields() default true;

	/**
	 * Flag to indicate that an exception should be raised if a Validator is available and
	 * validation fails. If it is set to false, validation errors will be swallowed. They
	 * will be logged, but not propagated to the caller.
	 * @return the flag value (default true)
	 */
	boolean exceptionIfInvalid() default true;

	/**
	 * Optionally provide explicit resource locations to bind to. By default the
	 * configuration at these specified locations will be merged with the default
	 * configuration. These resources take precedence over any other property sources
	 * defined in the environment.
	 * @return the path (or paths) of resources to bind to
	 * @see #merge()
	 * @deprecated as of 1.4 in favor of configuring the environment directly with
	 * additional locations
	 */
	@Deprecated
	String[] locations() default {};

	/**
	 * Flag to indicate that configuration loaded from the specified locations should be
	 * merged with the default configuration.
	 * @return the flag value (default true)
	 * @see #locations()
	 * @deprecated as of 1.4 along with {@link #locations()} in favor of configuring the
	 * environment directly with additional locations
	 */
	@Deprecated
	boolean merge() default true;

}

```

```
spring 2.0

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ConfigurationProperties {
    @AliasFor("prefix")
    String value() default "";

    @AliasFor("value")
    String prefix() default "";

    boolean ignoreInvalidFields() default false;

    boolean ignoreUnknownFields() default true;
}
```

特别是在配置第三方的属性的时候，是不可以用@value的。
```
@Configuration
@ConfigurationProperties()
public class AppConfig {
    @Autowired
    private Environment env;

    public String getNote() {
        return note;
    }

    public void setNote(String  note) {
        this.note = note;
    }

    private String note; // must have setter. @Value do not need setter

    @Bean
    @ConfigurationProperties()
    public TestBean testBean() {
        TestBean t = new TestBean();
        String name = env.getProperty("data.server");
        t.setName(name);
        return t;
    }
}

---
public class TestBean {
    private String name;
    private int age;
}

```

### 5.3 Demo
- configuration file in resources direction

you.yml
```
my:
  name : mix
  age: 3
  remark: hello!
  keys:
    - key1
    - key2
```

application.yml
```
data:
  server: 117.30.29.78

logging:
  path: /data/log/java_web/spring_boot/
```
logback-spring.xml省略

- java code

TestBean
```
import lombok.Data;

import java.util.List;

/**
 * @author yuzhigang on 2/11/2018 11:29 AM.
 * @version 1.0
 * Description:
 */
@Data
public class TestBean {

    private String name;
    private int age;
    private String remark;
    private List<String> keys;
}
```

AppConfig
```
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.env.Environment;
import org.springframework.core.io.ClassPathResource;

/**
 * @author yuzhigang on 2/11/2018 10:46 AM.
 * @version 1.0
 * Description:
 */
@Configuration
@PropertySource(value = "classpath:/prop.properties")
@Slf4j
public class AppConfig {

    @Autowired
    private Environment env;

    /**
     * @Value do not needd public setter method
     */
    @Value("${data.server}")
    private String server;

    @Bean
    @ConfigurationProperties(prefix = "my")
    public TestBean testBean() {
        TestBean t = new TestBean();

        // since my.name is loaded by propertySourcesPlaceholderConfigurer
        // my.name is not a property in ENVIRONMENT
        String name = env.getProperty("my.name");
        log.info("my.name=" + name);

        // 1. since p.key loaded by @PropertySource, p.key has been add to ENVIRONMENT
        // 2. suppose TestBean is a third-part class, we can't use @Value to bind property.
        // alternatively, we have to get external property configuration and call setter method
        // e.g. t.setName(value)
        String value = env.getProperty("p.key");
        log.info("p.key=" + value);

        log.info("this.server=" + server);

        return t;
    }

    @Bean
    public static PropertySourcesPlaceholderConfigurer properties() {
        // here just to show how to load a yml configuration file.
        // but in practice, this is useless in most case
        // since you can put yml configuration into application.yml directly which loaded by framework automatically
        PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer = new PropertySourcesPlaceholderConfigurer();
        YamlPropertiesFactoryBean yaml = new YamlPropertiesFactoryBean();
        yaml.setResources(new ClassPathResource("you.yml"));
        propertySourcesPlaceholderConfigurer.setProperties(yaml.getObject());
        return propertySourcesPlaceholderConfigurer;
    }

}

```

Appliation 
```
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
@Slf4j
public class SpringBootSimpleApplication implements ApplicationRunner {

    @Autowired
	TestBean bean;
	
	public static void main(String[] args) {
		System.out.print(">>>> entry piont:");
		SpringApplication.run(SpringBootSimpleApplication.class, args);
    }
    
    @Override
	public void run(ApplicationArguments args) throws Exception {
		log.info("TestBean:" + this.bean);
	}
}
```

- output
```
2018-11-07 11:18:36.935 [main] INFO  com.apress.spring.SpringBootSimpleApplication - Starting SpringBootSimpleApplication on bogon with PID 76131 (/Users/yuzhigang/workspace/spring_boot/target/classes started by yuzhigang in /Users/yuzhigang/workspace/spring_boot)
2018-11-07 11:18:36.939 [main] INFO  com.apress.spring.SpringBootSimpleApplication - No active profile set, falling back to default profiles: default
2018-11-07 11:18:37.318 [main] INFO  com.apress.spring.conf.AppConfig - my.name=null
2018-11-07 11:18:37.319 [main] INFO  com.apress.spring.conf.AppConfig - p.key=value
2018-11-07 11:18:37.319 [main] INFO  com.apress.spring.conf.AppConfig - this.server=conf_server
2018-11-07 11:18:37.411 [main] INFO  com.apress.spring.SpringBootSimpleApplication - Started SpringBootSimpleApplication in 0.854 seconds (JVM running for 1.32)
2018-11-07 11:18:37.413 [main] INFO  com.apress.spring.SpringBootSimpleApplication - TestBean:TestBean(name=mix, age=3, remark=hello!, keys=[key1, key2])
```

本来是因为看现在好多的讲解不够清楚和本质。但是，自己说了一通后，觉得很多地方说的也不是很清楚，虽然心里清楚。加上都是挤时间写的。诶，要是能有人鼓励一下的话。我再继续吧

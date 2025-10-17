# Technical aspects: Spring (bean life cycle, creation, injection, scope)

**Original** **Jimoer** [Jimoer](javascript:void(0);)

 *September 16, 2025, 16:10*

## What is the life cycle of a Spring Bean?

**In the Spring container, a Bean goes through the following stages from creation to destruction: * *** `<span leaf="" class="js_darkmode__text__7">定义阶段</span>`* * (Bean Meta Information Configuration) = > `<span leaf="" class="js_darkmode__text__10">实例化阶段</span>`(Create a Bean object) = > `<span leaf="" class="js_darkmode__text__13">初始化阶段</span>`(Execute initialization logic) = > `<span leaf="" class="js_darkmode__text__16">使用阶段</span>`(Bean available) = > `<span leaf="" class="js_darkmode__text__19">销毁阶段</span>`(Release resources)

![](https://mmbiz.qpic.cn/mmbiz_png/51zXejl2tKzUbjVicOadtJvBnibS1JWkicpVRRImhhTfiboia9deMWVpCxPx4KwKZfYhqobuL4YuCqfZiaXqjsoBLvjw/640?wx_fmt=png&from=appmsg)

### The Definition stage (BeanDefinition parsing)

 **Spring parses the Bean's meta-data through configuration (XML, annotations, Java configuration) to generate `<span leaf="" class="js_darkmode__text__28">BeanDefinition</span>`Object ** **.** `<span leaf="" class="js_darkmode__text__32">BeanDefinition</span>`It was stored. `<span leaf="" class="js_darkmode__text__35">Bean的类名、作用域（scope）、依赖项（depends-on）、初始化方法、销毁方法</span>`Similar metadata. all `<span leaf="" class="js_darkmode__text__38">BeanDefinition</span>`Storage in a container `<span leaf="" class="js_darkmode__text__41">BeanDefinitionMap</span>`(a HashMap), the key is the Bean name and the value is `<span leaf="" class="js_darkmode__text__44">BeanDefinition</span>`Object . **Parser :**

* **XML configuration:** `<span leaf="" class="js_darkmode__text__53">XmlBeanDefinitionReader</span>`analysis `<span leaf="" class="js_darkmode__text__56"><bean></span>`The label.
* **Note Configuration:** `<span leaf="" class="js_darkmode__text__62">ClassPathBeanDefinitionScanner</span>`scanning `<span leaf="" class="js_darkmode__text__65">@Component</span>`Wait until the note.
* **Java configuration:** `<span leaf="" class="js_darkmode__text__71">ConfigurationClassPostProcessor</span>`analysis `<span leaf="" class="js_darkmode__text__74">@Bean</span>`The method.

### Instantiation phase (creating a Bean instance)

 **according to `<span leaf="" class="js_darkmode__text__83">BeanDefinition</span>`Creating by reflection or factory methods `<span leaf="" class="js_darkmode__text__86">Bean</span>`Instance （object）, but the attribute is not injected at this time ** **。Instantiated by default via a no-arg constructor （if not specified, Spring will force a no-arg constructor）.**

> **in** `<span leaf="" class="js_darkmode__text__93">AbstractAutowireCapableBeanFactory</span>`In a class `<span leaf="" class="js_darkmode__text__96">createBeanInstance</span>`Implemented in the method.

### Attribute value filling (dependency injection)

 **Sets a value for a bean 's property or injects a dependency ** **.**

* **adopt** `<span leaf="" class="js_darkmode__text__110">@Autowired</span>`、 `<span leaf="" class="js_darkmode__text__113">@Value</span>`、 `<span leaf="" class="js_darkmode__text__116">XML</span>`of `<span leaf="" class="js_darkmode__text__119"><property></span>`The following methods are used to inject attributes.
* **If the injected dependency is another Bean, it recursively triggers the life cycle of the dependent Bean.**
* **Loop dependency problem: Deal with loop dependencies during the attribute injection phase (solved by a layer 3 cache).**

> **in** `<span leaf="" class="js_darkmode__text__131">AbstractAutowireCapableBeanFactory</span>`of `<span leaf="" class="js_darkmode__text__134">populateBean</span>`Deal with it in a method.

### Aware interface callback settings

 **If the bean implements a specific Aware interface , Spring calls the corresponding method and injects the container-related object ** **.**

* `<span leaf="" class="js_darkmode__text__147">BeanNameAware</span>`: The name of the injected Bean in the container ( `<span leaf="" class="js_darkmode__text__150">setBeanName(String beanName)</span>`）。
* `<span leaf="" class="js_darkmode__text__155">BeanFactoryAware</span>`Is injected into the BeanFactory where the current Bean is `<span leaf="" class="js_darkmode__text__158">setBeanFactory(BeanFactory beanFactory)</span>`）。
* `<span leaf="" class="js_darkmode__text__163">ApplicationContextAware</span>`: If the container is ApplicationContext, inject the application context `<span leaf="" class="js_darkmode__text__166">setApplicationContext(ApplicationContext applicationContext)</span>`）。

> **in** `<span leaf="" class="js_darkmode__text__172">AbstractAutowireCapableBeanFactory</span>`of `<span leaf="" class="js_darkmode__text__175">initializeBean</span>`Call within the method.

### BeanPostProcessor Preprocessing

**Before Bean is first initialized, allow custom** `<span leaf="" class="js_darkmode__text__183">BeanPostProcessor</span>`Process the Bean instance. It was mainly a call. `<span leaf="" class="js_darkmode__text__186">BeanPostProcessor</span>`of `<span leaf="" class="js_darkmode__text__189">postProcessBeforeInitialization</span>`Methods . **Common implementation classes**

* `<span leaf="" class="js_darkmode__text__197">ApplicationContextAwareProcessor</span>`: Handling `<span leaf="" class="js_darkmode__text__200">ApplicationContextAware</span>`Interface.
* `<span leaf="" class="js_darkmode__text__205">InitDestroyAnnotationBeanPostProcessor</span>`: Handling `<span leaf="" class="js_darkmode__text__208">@PostConstruct</span>`Notes.

> **by** `<span leaf="" class="js_darkmode__text__214">AbstractAutowireCapableBeanFactory</span>`of `<span leaf="" class="js_darkmode__text__217">applyBeanPostProcessorsBeforeInitialization</span>`The method is executed.

### InitializingBean processing and custom init-method processing

**Execute the initialization logic of the Bean.** `<span leaf="" class="js_darkmode__text__225">InitializingBean</span>`Processing, initializing after all Bean properties are set. If Bean implements the `<span leaf="" class="js_darkmode__text__228">InitializingBean</span>`Interface, `<span leaf="" class="js_darkmode__text__231">InitializingBean</span>`of `<span leaf="" class="js_darkmode__text__234">afterPropertiesSet</span>`The method will be called.

**Custom init-method handling . This method is called if the bean has an init-method defined in the configuration file . **For example , configure init-method via XML or @ Bean （ initMethod = " xxx " ） via Java .

> **in** `<span leaf="" class="js_darkmode__text__244">AbstractAutowireCapableBeanFactory</span>`of `<span leaf="" class="js_darkmode__text__247">invokeInitMethods</span>`Calls in methods

### BeanPostProcessor Postprocessing

**After the initial Bean, allow custom** `<span leaf="" class="js_darkmode__text__255">BeanPostProcessor</span>`Process the Bean instance. `<span leaf="" class="js_darkmode__text__258">BeanPostProcessor</span>`of `<span leaf="" class="js_darkmode__text__261">postProcessAfterInitialization</span>`The method will be called.

**Common use cases :**  **AOP proxies （ such as AbstractAutoProxyCreator to create proxies for target objects at this stage ） ** **.**

> **by** `<span leaf="" class="js_darkmode__text__272">AbstractAutowireCapableBeanFactory</span>`of `<span leaf="" class="js_darkmode__text__275">applyBeanPostProcessorsAfterInitialization</span>`Method implementation

### Register **DisposableBean** callback

**If Bean implements the `<span leaf="" class="js_darkmode__text__284">DisposableBean</span>`Interface or a custom destruction method specified in the Bean definition, the Spring container registers a destruction callback for these Beans, ensuring that the resource is properly cleaned when the container is closed.**

> **in** `<span leaf="" class="js_darkmode__text__290">AbstractAutowireCapableBeanFactory</span>`In a class `<span leaf="" class="js_darkmode__text__293">registerDisposableBeanlfNecessary</span>`Implementation in the method

### Bean Usage Phase

**Bean is fully initialized and ready for use by the application . **Obtaining a Bean instance through dependency injection （ e.g. `<span leaf="" class="js_darkmode__text__303">@Autowired</span>`or `<span leaf="" class="js_darkmode__text__306">ApplicationContext.getBean()</span>`). At this stage, the Bean is "available" until the container is closed.

### Bean Destruction Phase

**When the container closes, the Bean resource is released. The main steps:**

* **Interface callback: If Bean implements** `<span leaf="" class="js_darkmode__text__318">DisposableBean</span>`Call `<span leaf="" class="js_darkmode__text__321">destroy</span>`The method.
* **Note: If the method is annotated,** `<span leaf="" class="js_darkmode__text__327">@PreDestroy</span>`, Spring calls this method.
* **Custom destruction method: configuration via XML** `<span leaf="" class="js_darkmode__text__333">destroy-method</span>`Or Java configuration `<span leaf="" class="js_darkmode__text__336">@Bean(destroyMethod="xxx")</span>`。
* **Resource release: such as closing the database connection, releasing the file handle, etc.**

> **in** `<span leaf="" class="js_darkmode__text__345">DisposableBeanAdapter</span>`of `<span leaf="" class="js_darkmode__text__348">destroy</span>`Implementation in the method

### summary

**From the source code, you can observe that the entire creation of the Bean depends on** `<span leaf="" class="js_darkmode__text__356">AbstractAutowireCapableBeanFactory</span>`This category, and destruction, depends primarily on it. `<span leaf="" class="js_darkmode__text__359">DisposableBeanAdapter</span>`This class is. `<span leaf="" class="js_darkmode__text__362">AbstractAutowireCapableBeanFactory</span>`At the entrance, `<span leaf="" class="js_darkmode__text__365">doCreateBean</span>`The core code is as follows, which includes **instantiation, setting the value of the attribute, initializing the Bean and registering the destruction callback **of the core methods。I won't paste the code here. If you want to see more details, you can go to the source code.

## What are the ways to create Beans in Spring?

### Automatic scanning based on annotations

**The class is annotated and automatically registered with component scanning . **Common annotations include `<span leaf="" class="js_darkmode__text__381">@Component</span>`, `<span leaf="" class="js_darkmode__text__384">@Service</span>`, `<span leaf="" class="js_darkmode__text__387">@Repository</span>`, `<span leaf="" class="js_darkmode__text__390">@Controller</span>`(and its derivative notes).

**Examples: When adding to a class** `<span leaf="" class="js_darkmode__text__395">@Component</span>`Before enabling component scanning in the configuration class or XML `<span leaf="" class="js_darkmode__text__398">@ComponentScan</span>`or `<span leaf="" class="js_darkmode__text__401"><context:component-scan></span>`）。This class is automatically scanned when the service starts and injected into the Spring container.

```
@Configuration
@ComponentScan("com.jimoer.service")
public class BeanConfig {

}
```

```
@Service
publicclass UserService {
    public void hello() {
        System.out.println("Hello from UserService");
    }
}

@Component
publicclass UserHandler {
    public void hello() {
        System.out.println("Hello from UserHandler");
    }
}

@Repository
publicclass UserRepository {
    public void hello() {
        System.out.println("Hello from UserRepository");
    }
}

@Controller
publicclass UserController {
    public void hello() {
        System.out.println("Hello from UserController");
    }
}
```

### Using @ Configuration and @ Bean annotations

**adopt `<span leaf="" class="js_darkmode__text__535">@Configuration</span>`The annotated configuration class that explicitly defines the creation logic for the bean . **Use when you need to precisely control the initialization logic of a bean （ such as when it depends on other beans or complex conditions ） .

```
@Configuration
public class AppConfig {
    @Bean
    public UserService userService() {
        return new UserService();
    }
}
```

### XML configuration file

**Beans are defined in xml. Before the popularity of SpringBoot, this method is quite a lot, after the popularity of SpringBoot, it is used less and less.**

```
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd"
    <bean id="userService" class="com.jimoer.demo.UserServiceImpl">
       <property name="message" value="Hello Spring!" />
    </bean>
</beans>
```

```
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
UserService userService = context.getBean("userService");
```

**Better for legacy projects or scenarios that require compatibility with non-annotated configurations.**

### Use the @Import annotation

**`<span leaf="" class="js_darkmode__text__654">@Import</span>`The role of the annotation is to quickly import one or more classes so that they can be loaded by Spring into the IOC container for management . **Let the class be Spring 's **IOC** Container management, this is also to create a bean, therefore, this method can also be considered as a way to create a bean.

```
@Import({UserServiceImpl.class})
@Configuration
public class UserBeanConfiguration {
}
```

### Custom Commentary

**By defining a kind of annotation, and then during the Spring application startup, through the custom** `<span leaf="" class="js_darkmode__text__684">BeanDefinitionRegistryPostProcessor</span>`and `<span leaf="" class="js_darkmode__text__687">BeanfactoryPostProcessor</span>`Scan the configuration package path to identify classes with custom annotations. These handlers parse attributes in the annotation (such as interface class, version number, timeout, etc.) and create Spring's `<span leaf="" class="js_darkmode__text__690">BeanDefinition</span>`。Example: The Dubbo framework uses the `<span leaf="" class="js_darkmode__text__693">@DubboService</span>`annotation

```
@DubboService("version=1.0.0")
public class UserServiceImpl implements UserFacadeService {

}
```

### Dynamic registration (runtime registration)

**Dynamically register the Bean at run time through BeanDefinitionRegistry.**

```
// 获取 BeanFactory
ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();

// 定义 Bean 的元数据
GenericBeanDefinition userDefinition = new GenericBeanDefinition();
userDefinition.setBeanClass(UserService.class);

// 注册 Bean
beanFactory.registerBeanDefinition("userService", userDefinition);
```

**For: Dynamically generate Beans based on runtime conditions (e.g. plug-in systems, dynamic configurations).**

## What are the injection methods of Spring Bean?

### Using an @Autowired annotation

`<span leaf="" class="js_darkmode__text__756">@Autowired</span>`The annotation is a annotation provided by the Spring framework that supports a variety of ways to automatically inject Spring beans into other beans.

#### Field injection

```
@Component
public class JimoerUserService {
    @Autowired
    private UserRepository userRepository;
}
```

#### Construction method injection

```
@Component
public class JimoerUserService {
    private final UserRepository userRepository;

    // Spring 4.3+ 可省略 @Autowired（单构造器）
    @Autowired
    public JimoerUserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

#### Setter injection

```
@Component
public class JimoerUserService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### Use @Resource and @Inject annotations

**In addition to the annotations provided by Spring, the JDK also provides annotations that can be injected into each other's Beans.** `<span leaf="" class="js_darkmode__text__857">@Resource</span>`and `<span leaf="" class="js_darkmode__text__860">@Inject</span>`

```
@Component
public class JimoerUserService {
    @Resource
    private UserRepository userRepository;
}

@Component
public class JiomerUserService {
    @Inject
    private UserRepository userRepository;
}
```

### Using XML Configuration Injection

**How not to use annotation injection, you can also use the configuration of the XML file for Bean injection to each other.**

```
<bean id="userRepository" class="com.jimoer.UserRepository"/>
<!-- 构造方法注入 -->
<bean id="userService" class="com.jiomer.UserService">
    <constructor-arg ref="userRepository"/>
</bean>
<!-- 字段注入 -->
<bean id="jimoerUserService" class="com.jiomer.JimoerUserService">
    <property name="userRepository" ref="userRepository"/>
</bean>
```

### Construction method automatically injects

**In fact, starting with Spring 4.3, you don't need to use a class unless at least two constructors are declared in a class.** `<span leaf="" class="js_darkmode__text__992">@Autowired</span>`Label a configuration function, which can also be directly injected into Bean.

```
@Component
public class JimoerUserService {
 private UserRepository userRepository;
 public JimoerUserService(UserRepository userRepository){
  this.userRepository=userRepository;
 }
}
```

## What are the scopes of Spring Bean?

**The scope of a Spring bean is the extent to which the bean can be used. Different scopes determine how Beans are created, managed and destroyed.**

**Common areas of action are** `<span leaf="" class="js_darkmode__text__1030">Singleton</span>`、 `<span leaf="" class="js_darkmode__text__1033">Prototype</span>`、 `<span leaf="" class="js_darkmode__text__1036">Request</span>`、 `<span leaf="" class="js_darkmode__text__1039">Session</span>`、 `<span leaf="" class="js_darkmode__text__1042">Application</span>`These are five kinds. In the code, you can define a Bean by `<span leaf="" class="js_darkmode__text__1045">@Scope</span>`Annotations specify his scope of action.

**If the scope of the Bean is not specified, the default is** `<span leaf="" class="js_darkmode__text__1050">Singleton</span>`(A single example).

### Singleton

* **Cycle: Instances are created when the Spring container is started and destroyed when the container is closed.**
* **Scope: For each Spring IOC container, only one Bean instance is created.**
* **Applies to: stateless services (such as tool classes, cache managers, database connection pools).**
* **Thread Safe: Note that if the Bean has mutable state (that is, there are thread-shared variables in the Bean), it needs to be handled through synchronization mechanisms or thread safe collections.**
* **Configuration methods:\**

```
@Component // 默认即为 singleton
public class SingletonBean {
}
```

### Propertype (prototype)

* **Cycle: Each call** `<span leaf="" class="js_darkmode__text__1094">getBean()</span>`Or a new instance is created during injection, and the container is not responsible for destroying it.
* **For: Stateful Beans (e.g. user session data, temporary objects).**
* **Thread security: Instances are independent, avoiding thread security issues.**
* **Configuration methods:\**

```
@Component
@Scope("prototype")
public class PrototypeBean {
}
```

### Request (HTTP request)

* **Cycle: Each HTTP request creates an instance, which is destroyed at the end of the request.**
* **Applicable to: request-level data sharing in Web applications (e.g. request logs, context information).**

> **Only available for web application environments.**

* **Configuration methods:\**

```
@Component
@Scope("request")
public class RequestBean {
}
```

### Session(HTTP 会话)

* **Period: Each user session (** `<span leaf="" class="js_darkmode__text__1168">HttpSession</span>`) Create an instance and destroy it at the end of the session.
* **Applies to: user session data (e.g. shopping carts, user preferences).**

> **Only available for web application environments.**

* **Configuration methods:\**

```
@Component
@Scope("session")
public class SessionBean {
}
```

### Application (Application)

* **Cycle: Instances are created when the web application starts and destroyed when the application closes.**
* **For: Global configuration or shared resources (such as application-level caching, configuration information). similar** `<span leaf="" class="js_darkmode__text__1210">singleton</span>`But it's tied to. `<span leaf="" class="js_darkmode__text__1213">ServletContext</span>`。

> **Only available in web environments**

* **Configuration methods:\**

```
@Component
@Scope("application")
public class ApplicationBean {
}
```

### Websocket(WebSocket 会话)

* **Period:** `<span leaf="" class="js_darkmode__text__1249">WebSocket</span>`An instance is created when the connection is established and destroyed when the link is closed.
* **Applies to:** `<span leaf="" class="js_darkmode__text__1255">WebSocket</span>`Session context data (such as real time communication status).

> **Only available for WebSocket applications.**

* **Configuration methods:\**

```
@Component
@Scope("websocket")
public class WebSocketBean {
}
```

### Custom scopes

**In general, during the development process, it is all about using** `<span leaf="" class="js_darkmode__text__1289">Singleton</span>`Scopes are also used sometimes. `<span leaf="" class="js_darkmode__text__1292">Propertype</span>`They don't use many of the other ones. But in addition to the six Spring-provided scopes listed above, you can define your own Bean scopes.

**Customize the scope of a Spring Bean, you need to implement** `<span leaf="" class="js_darkmode__text__1297">org.springframework.beans.factory.config.Scope</span>`Interface, mainly implements the following methods to manage the life cycle of the Bean.

```
package org.springframework.beans.factory.config;

import org.springframework.beans.factory.ObjectFactory;

publicinterface Scope {
    Object get(String var1, ObjectFactory<?> var2);

    Object remove(String var1);

    void registerDestructionCallback(String var1, Runnable var2);

    Object resolveContextualObject(String var1);

    String getConversationId();
}
```

**Customize a class and implement it** `<span leaf="" class="js_darkmode__text__1355">Scope</span>`Interface, to implement our own Bean scope.

```
public class JimoerScope implements Scope{
    @Override
    public Object get(String s, ObjectFactory<?> objectFactory) {
        // 获取Bean的逻辑
        return objectFactory.getObject();
    }

    @Override
    public Object remove(String s) {
        // 移除Bean的逻辑
        returnnull;
    }

    @Override
    public void registerDestructionCallback(String s, Runnable runnable) {
        // 注册Bean销毁时的回调
    }

    @Override
    public Object resolveContextualObject(String s) {
        // 解析上下文
        returnnull;
    }

    @Override
    public String getConversationId() {
        // 获取会话ID
        return"";
    }
}
```

**Next, we register this custom scope in the Spring configuration. This can be done through** `<span leaf="" class="js_darkmode__text__1464">ConfigurableBeanFactory.registerScope</span>`Method implementation.

```
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
publicclass AppConfig {

    @Bean
    public JimoerScope jimoerScope(ConfigurableBeanFactory beanFactory) {
        JimoerScope jimoerScope = new JimoerScope();
        beanFactory.registerScope("jimoer", jimoerScope);
        return jimoerScope;
    }

}
```

**Use the name of the custom scope in the Bean definition at this point** `<span leaf="" class="js_darkmode__text__1514">jimoer</span>`. The Spring container will create and manage these Beans based on your custom logic.

```
@Component
@Scope("jimoer")
public class CustomerScopeTest {
}
```

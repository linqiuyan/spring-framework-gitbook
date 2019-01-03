# Part II. Spring Framework 4.x新特性

本章概述了Spring Framework 4.3中引入的新功能和改进。 如果您对更多详细信息感兴趣，请参阅链接：

## 3 Spring Framework 4.0中的新功能和增强功能

Spring Framework于2004年首次发布; 从那以后出现了重大的重大修改：Spring 2.0提供了XML命名空间和AspectJ支持; Spring 2.5采用了注释驱动的配置; Spring 3.0在框架代码库中引入了强大的Java 5+基础，以及基于Java的@Configuration模型等功能。

4.0版是Spring Framework的最新主要版本，也是第一个完全支持Java 8功能的版本。 您仍然可以将Spring与旧版本的Java一起使用，但是，现在已将最低要求提升到Java SE 6.我们还利用主要版本的机会删除了许多已弃用的类和方法。

Spring Framework GitHub Wiki上提供了升级到Spring 4.0的迁移指南。[https://github.com/spring-projects/spring-framework/wiki](https://github.com/spring-projects/spring-framework/wiki)

### 3.1改进的入门体验

新的spring.io网站提供了一系列“入门”指南，以帮助您学习Spring。 您可以在本文档的第1章“Spring入门”部分中阅读有关这些指南的更多信息。 新网站还全面概述了在Spring下发布的许多其他项目。

如果您是Maven用户，您可能还对现在随每个Spring Framework版本发布的有用的物料清单POM文件感兴趣。

### 3.2删除了不推荐使用的包和方法

版本4.0已删除所有已弃用的软件包以及许多已弃用的类和方法。 如果要从以前版本的Spring升级，则应确保已修复对过时API所做的任何已弃用的调用。

有关完整的更改，请查看[API差异报告](https://docs.spring.io/spring-framework/docs/3.2.4.RELEASE_to_4.0.0.RELEASE/)。

请注意，可选的第三方依赖项已提升至2010/2011最低要求（即Spring 4通常仅支持2010年末或更晚版本发布的版本）：值得注意的是，Hibernate 3.6 +，EhCache 2.1 +，Quartz 1.8 +，Groovy 1.8+ 和Joda-Time 2.0+。 作为规则的一个例外，Spring 4需要最近的Hibernate Validator 4.3+，并且对Jackson的支持现在已经集中在2.0+上（暂时保留了Jackson 1.8 / 1.9支持Spring 3.2的支持;现在只是在弃用 形成）。

### 3.3 Java 8（以及6和7）

Spring Framework 4.0支持多种Java 8功能。 您可以使用Spring的回调接口来使用lambda表达式和方法引用。 对java.time（JSR-310）有一流的支持，并且几个现有的注释已被改装为@Repeatable。 您还可以使用Java 8的参数名称发现（基于-parameters编译器标志）作为编译代码并启用调试信息的替代方法。

Spring仍然与旧版本的Java和JDK兼容：具体而言，仍然完全支持Java SE 6（具体而言，相当于JDK 6更新18的最低级别，如2010年1月发布）及更高版本。 但是，对于基于Spring 4的新开发项目，我们建议使用Java 7或8。

_截至2017年底，JDK 6正在逐步淘汰，因此也是Spring的JDK 6支持。 Oracle和IBM将在2018年终止JDK 6的所有商业支持工作。虽然Spring将保留其整个4.3.x系列的JDK 6运行时兼容性，但我们需要升级到JDK 7或更高版本以获得超出此点的任何进一步支持 ：特别是对于JDK 6特定的错误修复或其他问题，升级到JDK 7解决了这个问题。_

### 3.4 JavaEE 6和7

Java EE版本6或更高版本现在被认为是Spring Framework 4的基准，JPA 2.0和Servlet 3.0规范特别相关。 为了与Google App Engine和旧应用程序服务器保持兼容，可以将Spring 4应用程序部署到Servlet 2.5环境中。 但是，强烈建议使用Servlet 3.0+，这是Spring test和mock的先决条件。

_如果您是WebSphere 7用户，请确保安装JPA 2.0功能部件包。 在WebLogic 10.3.4或更高版本上，安装随附的JPA 2.0修补程序。 这将这两代服务器转变为兼容Spring 4的部署环境。_

在更具前瞻性的说明中，Spring Framework 4.0现在支持Java EE 7级别的适用规范：特别是JMS 2.0，JTA 1.2，JPA 2.1，Bean Validation 1.1和JSR-236 Concurrency Utilities。 像往常一样，这种支持侧重于个人使用这些规范，例如 在Tomcat或独立环境中。 但是，将Spring应用程序部署到Java EE 7服务器时，它同样适用。

请注意，Hibernate 4.3是JPA 2.1提供程序，因此仅在Spring Framework 4.0中受支持。 这同样适用于Hibernate Validator 5.0作为Bean Validation 1.1提供程序。 Spring Framework 3.2都没有正式支持这两者。

### 3.5 Groovy Bean定义DSL

从Spring Framework 4.0开始，可以使用Groovy DSL定义外部bean配置。 这在概念上类似于使用XML bean定义，但允许更简洁的语法。 使用Groovy还允许您直接在引导代码中嵌入bean定义。 例如：

```
def reader = new GroovyBeanDefinitionReader(myApplicationContext)
reader.beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

有关更多信息，请参阅`GroovyBeanDefinitionReader`[javadocs](https://docs.spring.io/spring-framework/docs/4.3.21.RELEASE/javadoc-api/org/springframework/beans/factory/groovy/GroovyBeanDefinitionReader.html)。

### 3.6 核心容器改进

核心容器有几个改进：

* Spring现在在注入Beans时将泛型类型视为限定符的形式。 例如，如果您使用的是Spring Data Repository，现在可以轻松注入特定的实现：

  ```
  @Autowired Repository<Customer> customerRepository
  ```

* 如果使用Spring的元注释支持，现在可以开发自定义注释，从源注释中公开特定属性。

* 现在可以在将bean自动装入列表和数组时对其进行排序。 支持@Order注释和Ordered接口。

* @Lazy注释现在可用于注入点以及@Bean定义。

* 为使用基于Java的配置的开发人员引入了@Description批注。

* 通过@Conditional注释添加了有条件地过滤bean的通用模型。 这类似于@Profile支持，但允许以编程方式开发用户定义的策略。

* 基于CGLIB的代理类不再需要默认构造函数。 通过objenesis库提供支持，该库在内部重新打包并作为Spring Framework的一部分进行分发。 使用此策略，根本不再为代理实例调用构造函数。

* 现在，整个框架都有受管理的时区支持，例如： 在LocaleContext上。

### 3.7 一般网站改进

部署到Servlet 2.5服务器仍然是一个选项，但Spring Framework 4.0现在主要关注Servlet 3.0+环境。 如果您使用的是Spring MVC测试框架，则需要确保与测试类路径中的Servlet 3.0兼容的JAR。

除了后面提到的WebSocket支持之外，还对Spring的Web模块进行了以下一般性改进：

* 您可以将新的@RestController注释与Spring MVC应用程序一起使用，从而无需为每个@RequestMapping方法添加@ResponseBody。
* 添加了AsyncRestTemplate类，允许在开发REST客户端时提供非阻塞异步支持。
* Spring现在在开发Spring MVC应用程序时提供全面的时区支持。

### 3.8 WebSocket，SockJS和STOMP消息传递

新的spring-websocket模块为Web应用程序中客户端和服务器之间基于WebSocket的双向通信提供全面支持。 它与JSR-356，Java WebSocket API兼容，另外还提供基于SockJS的回退选项（即WebSocket仿真），用于尚不支持WebSocket协议的浏览器（例如Internet Explorer &lt;10）。

新的spring-messaging模块增加了对STOMP的支持，作为在应用程序中使用的WebSocket子协议以及用于从WebSocket客户端路由和处理STOMP消息的注释编程模型。 因此，@ Controller现在可以包含@RequestMapping和@MessageMapping方法，用于处理来自WebSocket连接的客户端的HTTP请求和消息。 新的spring-messaging模块还包含以前来自Spring Integration项目的关键抽象，如Message，MessageChannel，MessageHandler等，作为基于消息传递的应用程序的基础。

有关更多详细信息（包括更全面的介绍），请参阅第26章WebSocket支持部分。

### 3.9 Testing 改进

除了在spring-test模块中修剪已弃用的代码之外，Spring Framework 4.0还引入了几个用于单元和集成测试的新功能。

* spring-test模块中的几乎所有注释（例如，@ ContextConfiguration，@ WebAppConfiguration，@ ContextHierarchy，@ ActiveProfiles等）现在都可以用作元注释来创建自定义组合注释并减少测试套件中的配置重复。
* 现在可以通过编程方式解析活动bean定义配置文件，只需实现自定义ActiveProfilesResolver并通过@ActiveProfiles的解析程序属性注册它。
* spring-core模块中引入了一个新的SocketUtils类，使您可以在localhost上扫描免费的TCP和UDP服务器端口。 此功能并非特定于测试，但在编写需要使用套接字的集成测试时非常有用，例如启动内存中SMTP服务器，FTP服务器，Servlet容器等的测试。
* 从Spring 4.0开始，org.springframework.mock.web包中的模拟集现在基于Servlet 3.0 API。 此外，一些Servlet API模拟（例如，MockHttpServletRequest，MockServletContext等）已经更新，具有微小的增强和改进的可配置性。

## 4 Spring Framework 4.1中的新功能和增强功能

4.1版包含了许多改进，如以下部分所述：

* 第4.1节 “JMS改进”
* 第4.2节 “Caching改进”
* 第4.3节 “Web改进”
* 第4.4节 “WebSocket消息传递改进”
* 第4.5节 “Testing改进”

### 4.1 JMS改进

Spring 4.1引入了一个更简单的基础结构，通过使用@JmsListener注释bean方法来注册JMS侦听器端点。 XML命名空间已得到增强，可支持这种新样式（jms：annotation-driven），并且还可以使用Java配置（@EnableJms，JmsListenerContainerFactory）完全配置基础结构。 也可以使用JmsListenerConfigurer以编程方式注册侦听器端点。

Spring 4.1还调整了JMS支持，使您可以从4.0中引入的spring-messaging抽象中受益，即：

* 消息侦听器端点可以具有更灵活的签名，并从标准消息传递注释中受益，例如@ Payload，@ Header，@ Headers和@SendTo。 也可以使用标准Message代替javax.jms.Message作为方法参数。
* 新的JmsMessageOperations接口可用，并允许使用Message抽象的JmsTemplate操作。

最后，Spring 4.1提供了额外的其他改进：

* JmsTemplate中的同步请求 - 回复操作支持
* 可以根据&lt;jms:listener/&gt;元素指定侦听器优先级
* 可以使用BackOff实现配置消息侦听器容器的恢复选项
* 支持JMS 2.0共享使用者

### 4.2 Caching改进

Spring 4.1使用Spring现有的缓存配置和基础架构抽象支持JCache（JSR-107）注释; 使用标准注释不需要进行任何更改。

Spring 4.1改进了自己的缓存抽象：

* 可以使用CacheResolver在运行时解析缓存。 因此，定义要使用的缓存名称的value参数不再是必需的。
* 更多操作级自定义：缓存解析器，缓存管理器，密钥生成器
* 新的@CacheConfig类级别注释允许在类级别共享公共设置，而不启用任何高速缓存操作。
* 使用CacheErrorHandler更好地处理缓存方法

随着新的putIfAbsent方法的添加，Spring 4.1在Cache接口中也发生了重大变化。

### 4.3 Web改进

* 已经使用新的抽象ResourceResolver，ResourceTransformer和ResourceUrlProvider扩展了对基于ResourceHttpRequestHandler的资源处理的现有支持。许多内置实现提供对版本化资源URL的支持（用于有效的HTTP缓存），定位gzip压缩资源，生成HTML 5 AppCache清单等。请参见第22.16.9节“资源服务”。
* 现在，@ RequestParam，@ RequestHeader和@MatrixVariable控制器方法参数支持JDK 1.8的java.util.Optional。
* 支持ListenableFuture作为DeferredResult的返回值替代，其中底层服务（或者可能是对AsyncRestTemplate的调用）已经返回ListenableFuture。
* @ModelAttribute方法现在以遵循相互依赖关系的顺序调用。见SPR-6299。
* Jackson的@JsonView直接支持@ResponseBody和ResponseEntity控制器方法，用于序列化同一POJO的不同数量的详细信息（例如摘要与详细信息页面）。通过在特殊键下添加序列化视图类型作为模型属性，基于视图的渲染也支持此功能。有关详细信息，请参阅“Jackson序列化视图支持”一节。
* Jackson现在支持JSONP。请参阅“Jackson JSONP支持”一节。
* 一个新的生命周期选项可用于在控制器方法返回之后和写入响应之前拦截@ResponseBody和ResponseEntity方法。利用声明一个实现ResponseBodyAdvice的@ControllerAdvice bean。对@JsonView和JSONP的内置支持利用了这一点。请参见第22.4.1节“使用HandlerInterceptor拦截请求”。
* 有三个新的HttpMessageConverter选项：
  * Gson  - 比Jackson更轻; 已经在Spring Android中使用。
  * Google协议缓冲区 - 作为企业内的服务间通信数据协议高效且有效，但也可以作为浏览器的JSON和XML公开。
  * 现在通过jackson-dataformat-xml扩展支持基于Jackson的XML序列化。 当使用@EnableWebMvc或&lt;mvc:annotation-driven/&gt;时，如果jackson-dataformat-xml在类路径中，则默认使用它而不是JAXB2。
* JSP之类的视图现在可以通过按名称引用控制器映射来构建到控制器的链接。每个@RequestMapping都分配了一个默认名称。例如，带有方法handleFoo的FooController被命名为“FC＃handleFoo”。命名策略是可插拔的。也可以通过其name属性显式命名@RequestMapping。 Spring JSP标记库中的新mvcUrl函数使其易于在JSP页面中使用。请参见第22.7.3节“从视图构建控制器和方法的URI”。
* ResponseEntity提供了一个构建器式API，用于指导控制器方法准备服务器端响应，例如， ResponseEntity.ok\(\)。
* RequestEntity是一种新类型，它提供构建器样式的API，以指导客户端REST代码准备HTTP请求。
* MVC Java配置和XML命名空间：
  * 现在可以配置视图解析器，包括支持内容协商，请参见第22.16.8节“查看解析器”。
  * View Controllers现在具有内置的重定向支持和设置响应状态。应用程序可以使用它来配置重定向URL，使用视图呈现404响应，发送“无内容”响应等。此处列出了一些用例。
  * 路径匹配自定义经常使用，现在是内置的。请参见第22.16.11节“路径匹配”。
* Groovy标记模板支持（基于Groovy 2.3）。查看GroovyMarkupConfigurer并重新选择ViewResolver和\`View'实现。

### 4.4 WebSocket消息传递改进

* SockJS（Java）客户端支持。 请参阅同一包中的SockJsClient和类。
* STOMP客户端订阅和取消订阅时发布的新应用程序上下文事件SessionSubscribeEvent和SessionUnsubscribeEvent。
* 新的“websocket”范围。 请参见第26.4.16节“WebSocket范围”。
* @SendToUser只能定位一个会话，不需要经过身份验证的用户。
* @MessageMapping方法可以使用点“。” 而不是斜杠“/”作为路径分隔符。 见SPR-11660。
* 收集并记录STOMP / WebSocket监控信息。 请参见第26.4.18节“监控”。
* 显着优化和改进的日志记录，即使在DEBUG级别，仍应保持可读性和紧凑性。
* 优化的消息创建，包括支持临时消息可变性和避免自动消息ID和时间戳创建。 请参阅MessageHeaderAccessor的Javadoc。
* 关闭在建立WebSocket会话后60秒内没有活动的STOMP / WebSocket连接。 见SPR-11884。

### 4.5 Testing改进

* Groovy脚本现在可用于配置为TestContext框架中的集成测试加载的ApplicationContext。
  * 有关详细信息，请参阅“使用Groovy脚本进行上下文配置”一节。
* 现在可以通过新的TestTransaction API以编程方式启动并在事务测试方法中结束测试管理的事务
  * 有关详细信息，请参阅“程序化事务管理”一节。
* 现在可以通过基于每个类或每个方法的新@Sql和@SqlConfig注释以声明方式配置SQL脚本执行
  * 有关详细信息，请参见第15.5.8节“执行SQL脚本”。
* 可以通过新的@TestPropertySource批注配置自动覆盖系统和应用程序属性源的测试属性源
  * 有关详细信息，请参阅“使用测试属性源进行上下文配置”一节。
* 现在可以自动发现默认的TestExecutionListeners。
  * 有关详细信息，请参阅“自动发现默认TestExecutionListeners”一节。
* 自定义TestExecutionListeners现在可以自动与默认侦听器合并。
  * 有关详细信息，请参阅“合并TestExecutionListeners”一节。
* TestContext框架中的事务测试支持文档已通过更详尽的解释和其他示例进行了改进
  * 有关详细信息，请参见第15.5.7节“事务管理”。
* 对MockServletContext，MockHttpServletRequest和其他Servlet API模拟的各种改进。
* AssertThrows已被重构以支持Throwable而不是Exception。
* 在Spring MVC Test中，可以使用JSON Assert声明JSON响应，作为使用JSONPath的额外选项，就像使用XMLUnit可以为XML做的那样。
* 现在可以在MockMvcConfigurer的帮助下创建MockMvcBuilder配方。添加此选项是为了便于应用Spring Security设置，但可用于封装任何第三方框架或项目内的常见设置。
* MockRestServiceServer现在支持AsyncRestTemplate进行客户端测试。

## 5 Spring Framework 4.2中的新功能和增强功能

版本4.2包含许多改进，如以下部分所述：

* 第5.1节“核心容器改进”
* 第5.2节“数据访问改进”
* 第5.3节“JMS改进”
* 第5.4节“Web改进”
* 第5.5节“WebSocket消息传递改进”
* 第5.6节“测试改进”

6 




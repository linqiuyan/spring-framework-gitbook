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

### 5.1  核心容器改进

* 诸如@Bean之类的注释也可以在Java 8默认方法上检测和处理，允许使用默认的@Bean方法从接口编写配置类。
* 配置类现在可以使用常规组件类声明@Import，允许混合使用导入的配置类和组件类。
* 即使通过类路径扫描检测到，配置类也可以声明@Order值，以相应的顺序进行处理（例如，按名称覆盖bean）。
* @Resource注入点支持@Lazy声明，类似于@Autowired，接收所请求目标bean的延迟初始化代理。
* 应用程序事件基础结构现在提供基于注释的模型以及发布任意事件的能力。
  * 托管bean中的任何公共方法都可以使用@EventListener进行注释以使用事件。
  * @TransactionalEventListener提供事务绑定事件支持。
* Spring Framework 4.2引入了一流的支持，用于声明和查找注释属性的别名。 新的@AliasFor注释可用于在单个注释中声明一对别名属性，或者将自定义组合注释中的一个属性的别名声明为元注释中的属性。
  * 以下注释已经使用@AliasFor支持进行了改进，以便为其值属性提供有意义的别名：@Cacheable, @CacheEvict, @CachePut, @ComponentScan, @ComponentScan.Filter, @ImportResource, @Scope, @ManagedResource, @Header, @Payload, @SendToUser, @ActiveProfiles, @ContextConfiguration, @Sql, @TestExecutionListeners, @TestPropertySource, @Transactional, @ControllerAdvice, @CookieValue, @CrossOrigin, @MatrixVariable, @RequestHeader, @RequestMapping, @RequestParam, @RequestPart, @ResponseStatus, @SessionAttributes, @ActionMapping, @RenderMapping, @EventListener, @TransactionalEventListener
  * 例如，spring-test模块中的@ContextConfiguration现在声明如下：
    ```
    public @interface ContextConfiguration {

        @AliasFor("locations")
        String[] value() default {};

        @AliasFor("value")
        String[] locations() default {};

        // ...
    }
    ```
  * 类似地，覆盖元注释中的属性的组合注释现在可以使用@AliasFor来精确控制注释层次结构中的哪些属性被覆盖。 实际上，现在可以为元注释的value属性声明别名。

  * 例如，现在可以使用自定义属性覆盖开发组合注释，如下所示。

  ```
  @ContextConfiguration
  public @interface MyTestConfig {

      @AliasFor(annotation = ContextConfiguration.class, attribute = "value")
      String[] xmlFiles();

      // ...
  }
  ```

  * 详见[spring annotation编程模型](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#annotation-programming-model)

* 对用于查找元注释的Spring搜索算法进行了大量改进。例如，本地声明的组合注释现在比继承注释更受青睐。

* 现在可以在接口和抽象，桥接和接口方法以及类，标准方法，构造函数和字段上发现从元注释覆盖属性的组合注释。
* 表示注释属性（和AnnotationAttributes实例）的地图可以合成（即，转换）为注释。
* 基于字段的数据绑定（DirectFieldAccessor）的功能已与当前基于属性的数据绑定（BeanWrapper）对齐。特别是，基于字段的绑定现在支持集合，数组和映射的导航。
* DefaultConversionService现在为Stream，Charset，Currency和TimeZone提供开箱即用的转换器。这些转换器也可以单独添加到任何任意的ConversionService中。
* DefaultFormattingConversionService为JSR-354 Money＆Currency中的值类型提供了开箱即用的支持（如果类路径中存在'javax.money'API）：即MonetaryAmount和CurrencyUnit。这包括支持应用@NumberFormat。
* @NumberFormat现在可以用作元注释。
* JavaMailSenderImpl有一个新的testConnection（）方法，用于检查与服务器的连接。
* ScheduledTaskRegistrar公开计划任务。
* Apache commons-pool2现在支持池AOP CommonsPool2TargetSource。
* 将StandardScriptFactory引入为基于JSR-223的脚本bean机制，通过XML中的lang：std元素公开。例如支持JavaScript和JRuby。 （注意：JRubyScriptFactory和lang：jruby现在已弃用，支持使用JSR-223。）

### 5.2  数据访问改进

* 现在通过AspectJ支持javax.transaction.Transactional。
* SimpleJdbcCallOperations现在支持命名绑定。
* 完全支持Hibernate ORM 5.0：作为JPA提供程序（自动调整）以及通过其原生API（由新的org.springframework.orm.hibernate5包覆盖）。
* 现在可以为嵌入式数据库自动分配唯一的名称，&lt;jdbc：embedded-database&gt;支持新的database-name属性。 有关详细信息，请参阅下面的“测试改进”。

### 5.3  JMS改进

* autoStartup属性可以通过JmsListenerContainerFactory控制。
* 现在可以为每个侦听器容器配置回复目标的类型。
* @SendTo注释的值现在可以使用SpEL表达式。
* 可以使用JmsResponse在运行时计算响应目标
* @JmsListener现在是一个可重复的注释，用于在同一个方法上声明几个JMS容器（如果你还没有使用Java8，请使用新引入的@JmsListeners）。

### 5.4  网站改进

* HTTP Streaming和Server-Sent Events支持，请参阅“HTTP Streaming”一节。
* 内置对CORS的支持，包括全局（MVC Java配置和XML命名空间）和本地（例如@CrossOrigin）配置。 有关详细信息，请参见第27章CORS支持。
* HTTP缓存更新：
  * 新的CacheControl构建器; 插入ResponseEntity，WebContentGenerator，ResourceHttpRequestHandler。
  * 改进了WebRequest中的ETag / Last-Modified支持。
* 自定义映射注释，使用@RequestMapping作为元注释。
* AbstractHandlerMethodMapping中的公共方法，用于在运行时注册和取消注册请求映射。
* AbstractDispatcherServletInitializer中受保护的createDispatcherServlet方法，以进一步自定义要使用的DispatcherServlet实例。
* HandlerMethod作为@ExceptionHandler方法的方法参数，在@ControllerAdvice组件中特别方便。
* java.util.concurrent.CompletableFuture作为@Controller方法的返回值类型。
* HttpHeaders中的字节范围请求支持以及服务静态资源。
* 在嵌套异常上检测到@ResponseStatus。
* RestTemplate中的UriTemplateHandler扩展点。
  * DefaultUriTemplateHandler公开baseUrl属性和路径段编码选项。
  * 扩展点也可用于插入任何URI模板库。
* OkHTTP与RestTemplate集成。
* 自定义baseUrl替代MvcUriComponentsBuilder中的方法。
* 序列化/反序列化异常消息现在以WARN级别记录。
* 默认JSON前缀已从“{} &&”更改为更安全的“）\]}'，”one。
* 新的RequestBodyAdvice扩展点和内置实现，以支持Jackson的@JsonView @RequestBody方法参数。
* 使用GSON或Jackson 2.6+时，处理程序方法返回类型用于改进List &lt;Foo&gt;等参数化类型的序列化。
* 将ScriptTemplateView引入为基于JSR-223的脚本Web视图机制，重点关注Nashorn（JDK 8）上的JavaScript视图模板。

### 5.5  WebSocket消息传递改进

* 公开有关已连接用户和订阅的状态信息：
  * 新的SimpUserRegistry公开为名为“userRegistry”的bean。
  * 跨服务器群集共享在线信息（请参阅代理中继配置选项）。

* 跨服务器群集解析用户目标（请参阅代理中继配置选项）。
* StompSubProtocolErrorHandler扩展点用于自定义和控制客户端的STOMP ERROR帧。
* 通过@ControllerAdvice组件的全局@MessageExceptionHandler方法。
* Heart-beats和SpEL表达式'selector'标头，用于使用SimpleBrokerMessageHandler进行订阅。
* 用于TCP和WebSocket的STOMP客户端;请参见第26.4.15节“STOMP客户端”。
* @SendTo和@SendToUser可以包含目标变量占位符。
* Jackson的@JsonView支持@MessageMapping和@SubscribeMapping方法的返回值。
* ListenableFuture和CompletableFuture作为@MessageMapping和@SubscribeMapping方法的返回值类型。
* 用于XML有效负载的MarshallingMessageConverter。

### 5.6  测试改进

* 现在可以使用JUnit规则而不是SpringJUnit4ClassRunner来执行基于JUnit的集成测试。这允许基于Spring的集成测试与JUnit的参数化或第三方运行程序（如MockitoJUnitRunner）等替代运行程序一起运行。
* * 有关详细信息，请参阅“Spring JUnit 4规则”一节。
* Spring MVC Test框架现在为HtmlUnit提供一流的支持，包括与Selenium的WebDriver集成，允许基于页面的Web应用程序测试，而无需部署到Servlet容器。
* * 有关详细信息，请参见第15.6.2节“HtmlUnit集成”。
* AopTestUtils是一个新的测试实用程序，它允许开发人员获取对隐藏在一个或多个Spring代理后面的底层目标对象的引用。
* * 有关详细信息，请参见第14.2.1节“常规测试实用程序”。
* ReflectionTestUtils现在支持设置和获取静态字段，包​​括常量。
* 现在保留通过@ActiveProfiles声明的bean定义配置文件的原始顺序，以支持Spring Boot的ConfigFileApplicationListener等用例，后者根据活动配置文件的名称加载配置文件。
* @DirtiesContext支持新的BEFORE\_METHOD，BEFORE\_CLASS和BEFORE\_EACH\_TEST\_METHOD模式，用于在测试之前关闭ApplicationContext  - 例如，如果大型测试套件中的某些恶意（即尚未确定）测试损坏了ApplicationContext的原始配置。
* @Commit是一个新的注释，可以用作@Rollback的直接替换（false）。
* @Rollback现在可用于配置类级别的默认回滚语义。
* * 因此，@ TransactionConfiguration现已弃用，将在后续版本中删除。
* @Sql现在支持通过新的语句属性执行内联SQL语句。
* 用于在测试之间缓存ApplicationContexts的ContextCache现在是一个公共API，其默认实现可以替换为自定义缓存需求。
* DefaultTestContext，DefaultBootstrapContext和DefaultCacheAwareContextLoaderDelegate现在是支持子包中的公共类，允许自定义扩展。
* TestContextBootstrappers现在负责构建TestContext。
* 在Spring MVC Test框架中，MvcResult详细信息现在可以在DEBUG级别记录或写入自定义OutputStream或Writer。有关详细信息，请参阅MockMvcResultHandlers中的新log（），print（OutputStream）和print（Writer）方法。
* JDBC XML命名空间在&lt;jdbc：embedded-database&gt;中支持新的database-name属性，允许开发人员为嵌入式数据库设置唯一的名称 - 例如，通过SpEL表达式或受当前活动bean影响的属性占位符定义简介。
* 现在可以为嵌入式数据库自动分配一个唯一的名称，从而允许在测试套件中的不同ApplicationContexts中重用常见的测试数据库配置。
* * 有关详细信息，请参见第19.8.6节“为嵌入式数据库生成唯一名称”。
* MockHttpServletRequest和MockHttpServletResponse现在通过getDateHeader和setDateHeader方法为日期标题格式提供更好的支持。

## 6 Spring Framework 4.3中的新功能和增强功能

版本4.3包括许多改进，如以下部分所述：

* 第6.1节“核心容器改进”
* 第6.2节“数据访问改进”
* 第6.3节“缓存改进”
* 第6.4节“JMS改进”
* 第6.5节“Web改进”
* 第6.6节“WebSocket消息传递改进”
* 第6.7节“测试改进”
* 第6.8节“支持新的库和服务器代”

### 6.1  核心容器改进

* 核心容器异常提供更丰富的元数据，以便以编程方式进
* Java 8默认方法被检测为bean属性getter / setters。
* 在注入主bean的情况下，不会创建惰性候选bean。
* 如果目标bean只定义了一个构造函数，则不再需要指定@Autowired批注。
* @Configuration类支持构造函数注入。
* 用于指定@EventListener条件的任何SpEL表达式现在都可以引用bean（例如@ beanName.method（））。
* 组合注释现在可以使用数组的组件类型的单个元素覆盖元注释中的数组属性。例如，@ RequestMapping的String \[\] path属性可以用组合注释中的String路径覆盖。
* @ PersistenceContext / @ PersistenceUnit选择一个主要的EntityManagerFactory bean，如果这样声明的话。
* @Scheduled和@Schedules现在可以用作元注释来创建具有属性覆盖的自定义组合注释。
* 任何范围的bean都适当支持@Scheduled。

### 6.2  数据访问改进

jdbc：initialize-database和jdbc：embedded-database支持可应用于每个脚本的可配置分隔符。

### 6.3  缓存改进

* Spring 4.3允许对给定键的并发调用进行同步，以便仅计算一次值。 这是一个选择加入功能，应通过@Cacheable上的新同步属性启用。 此功能引入了Cache接口的重大更改，因为添加了get（Object key，Callable &lt;T&gt; valueLoader）方法。

* Spring 4.3还改进了缓存抽象，如下所示：
  * 缓存相关注释中的SpEL表达式现在可以引用bean（即@ beanName.method（））。
  * ConcurrentMapCacheManager和ConcurrentMapCache现在支持通过新的storeByValue属性对缓存条目进行序列化。
  * @Cacheable，@ CacheEvict，@ CachePut和@Caching现在可以用作元注释来创建具有属性覆盖的自定义组合注释。

### 6.4  JMS改进

* 现在可以在类级别指定@SendTo以共享公共回复目标。
* @JmsListener和@JmsListeners现在可以用作元注释来创建具有属性覆盖的自定义组合注释。

### 6.5  网站改进

* 内置支持HTTP HEAD和HTTP OPTIONS。
* 新的@ GetMapping，@ PostMapping，@ PutMapping，@ DeleMapping和@PatchMapping为@RequestMapping组成了注释。

* 有关详细信息，请参阅Composed @RequestMapping Variants。
* 新的@RequestScope，@ SessionScope和@ApplicationScope组成了Web范围的注释。

* 有关详细信息，请参阅请求范围，会话范围和应用程序范围
* 新的@RestControllerAdvice注释与@ControllerAdvice和@ResponseBody语义相结合。
* @ResponseStatus现在在类级别受支持，并由所有方法继承。
* 用于访问会话属性的新@SessionAttribute注释（请参阅示例）。
* 用于访问请求属性的新@RequestAttribute批注（参见示例）。
* @ModelAttribute允许通过binding = false属性阻止数据绑定（请参阅参考资料）。
* @PathVariable可以声明为可选（用于@ModelAttribute方法）。
* 将错误和自定义Throwables一致地暴露给MVC异常处理程序。
* HTTP消息转换器中一致的字符集处理，包括多部分文本内容的UTF-8默认值。
* 静态资源处理使用配置的ContentNegotiationManager进行媒体类型确定。
* RestTemplate和AsyncRestTemplate通过DefaultUriTemplateHandler支持严格的URI变量编码。
* AsyncRestTemplate支持请求拦截。

### 6.6  WebSocket消息传递改进

* 现在可以在类级别指定@SendTo和@SendToUser来共享公共目标。

### 6.7  测试改进

* Spring TestContext Framework中的JUnit支持现在需要JUnit 4.12或更高版本。
* SpringJUnit4ClassRunner的新SpringRunner别名。
* 现在可以在接口上声明测试相关的注释 - 例如，用于使用基于Java 8的接口默认方法的测试接口。
* 如果检测到默认XML文件，Groovy脚本或@Configuration类，则现在可以完全省略@ContextConfiguration的空声明。
* @Transactional测试方法不再需要公开（例如，在TestNG和JUnit 5中）。
* @BeforeTransaction和@AfterTransaction方法不再需要公开，现在可以在基于Java 8的接口默认方法上声明。
* Spring TestContext Framework中的ApplicationContext缓存现在受限于默认最大大小32和最近最少使用的驱逐策略。可以通过设置JVM系统属性或名为spring.test.context.cache.maxSize的Spring属性来配置最大大小。
* 新的ContextCustomizer API，用于在将bean定义加载到上下文之后但在刷新上下文之前自定义测试ApplicationContext。定制程序可以由第三方在全球注册，但需要实现自定义ContextLoader。
* @Sql和@SqlGroup现在可以用作元注释来创建具有属性覆盖的自定义组合注释。
* 现在，ReflectionTestUtils会在设置或获取字段时自动解包代理。
* 服务器端Spring MVC测试支持对具有多个值的响应头的期望。
* 服务器端Spring MVC Test解析表单数据请求内容并填充请求参数。
* 服务器端Spring MVC Test支持调用的处理程序方法的类似于断言的断言。
* 客户端REST测试支持允许指示期望请求的次数以及是否应忽略期望声明的顺序（请参见第15.6.3节“客户端REST测试”）。
* 客户端REST测试支持对请求正文中的表单数据的期望。

### 6.8  支持新library和server generations

* Hibernate ORM 5.2（仍然支持4.2 / 4.3和5.0 / 5.1，现在已经弃用了3.6）
* Hibernate Validator 5.3（最小值保持为4.3）
* Jackson2.8（截至4.3春季，最低升至Jackson2.6+）
* OkHttp 3.x（仍然支持OkHttp 2.x并排）
* Tomcat 8.5以及9.0里程碑
* Netty 4.1
* Undertow 1.4
* WildFly 10.1

此外，Spring Framework 4.3在spring-core.jar中嵌入了更新的ASM 5.1，CGLIB 3.2.4和Objenesis 2.4。




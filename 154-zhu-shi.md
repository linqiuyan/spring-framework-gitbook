## 15.4 注释

### 15.4.1 spring测试注释

Spring Framework提供了以下一组特定于Spring的注释，您可以在单元和集成测试中结合TestContext框架使用它们。 有关更多信息，请参阅相应的javadoc，包括默认属性值，属性别名等。

#### @BootstrapWith

@BootstrapWith是一个类级别的注释，用于配置Spring TestContext Framework的引导方式。 具体来说，@ BootstrapWith用于指定自定义TestContextBootstrapper。 有关更多详细信息，请参阅Bootstrapping TestContext框架部分。

#### @ContextConfiguration

@ContextConfiguration定义类级元数据，用于确定如何为集成测试加载和配置ApplicationContext。 具体来说，@ ContextConfiguration声明应用程序上下文资源位置或将用于加载上下文的带注释的类。

资源位置通常是位于类路径中的XML配置文件或Groovy脚本; 而注释类通常是@Configuration类。 但是，资源位置也可以引用文件系统中的文件和脚本，带注释的类可以是组件类等。

```
@ContextConfiguration("/test-config.xml")
public class XmlApplicationContextTests {
    // class body...
}
```

```
@ContextConfiguration(classes = TestConfig.class)
public class ConfigClassApplicationContextTests {
    // class body...
}
```

作为声明资源位置或带注释的类的替代或补充，@ ContextConfiguration可用于声明ApplicationContextInitializer类。

```
@ContextConfiguration(initializers = CustomContextIntializer.class)
public class ContextInitializerTests {
    // class body...
}
```

@ContextConfiguration也可以选择用于声明ContextLoader策略。 但请注意，您通常不需要显式配置加载器，因为默认加载器支持资源位置或带注释的类以及初始化器。

```
@ContextConfiguration(locations = "/test-context.xml", loader = CustomContextLoader.class)
public class CustomLoaderXmlApplicationContextTests {
    // class body...
}
```

\[注意\]

@ContextConfiguration默认支持继承资源位置或配置类以及超类声明的上下文初始值设定项。

有关更多详细信息，请参见第15.5.4节“上下文管理”和@ContextConfiguration javadocs。

#### @WebAppConfiguration

@WebAppConfiguration是一个类级别注释，用于声明为集成测试加载的ApplicationContext应该是WebApplicationContext。 仅在测试类上存在@WebAppConfiguration可确保为测试加载WebApplicationContext，使用默认值“file：src / main / webapp”作为Web应用程序根目录的路径（即资源） 基本路径）。 在后台使用资源基本路径来创建MockServletContext，它充当测试的WebApplicationContext的ServletContext。

```
@ContextConfiguration
@WebAppConfiguration
public class WebAppTests {
    // class body...
}
```

#### @ContextHierarchy

#### @ActiveProfiles

#### @TestPropertySource

#### @DirtiesContext

#### @TestExecutionListeners

#### @承诺

#### @Rollback

#### @BeforeTransaction

#### @AfterTransaction

#### @sql

#### @SqlConfig

#### @SqlGroup

### 15.4.2 标准注释支持

对于Spring TestContext Framework的所有配置，标准语义支持以下注释。 请注意，这些注释并非特定于测试，可以在Spring Framework中的任何位置使用。

* @Autowired
* @Qualifier
* @Resource（javax.annotation）如果存在JSR-250
* @ManagedBean（javax.annotation）如果存在JSR-250
* @Inject（javax.inject）如果存在JSR-330
* @Named（javax.inject）如果存在JSR-330
* @PersistenceContext（javax.persistence）如果存在JPA
* @PersistenceUnit（javax.persistence）如果存在JPA
* @Required
* @Transactional

### 15.4.3  Spring JUnit 4测试注释

同时@IfProfileValue

@ProfileValueSourceConfiguration

@Timed

@Repeat

### 15.4.4 测试的元注释支持






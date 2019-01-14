## 7.13 环境抽象

环境是集成在容器中的抽象，它模拟了应用程序环境的两个关键方面：配置文件和属性。

配置文件是仅在给定配置文件处于活动状态时才向容器注册的bean定义的命名逻辑组。 可以将Bean分配给配置文件，无论是以XML还是通过注释定义。 与配置文件相关的Environment对象的作用是确定哪些配置文件（如果有）当前处于活动状态，以及默认情况下哪些配置文件（如果有）应处于活动状态。

属性在几乎所有应用程序中都发挥着重要作用，并且可能源自各种源：属性文件，JVM系统属性，系统环境变量，JNDI，servlet上下文参数，ad-hoc属性对象，映射等。 与属性相关的Environment对象的作用是为用户提供方便的服务接口，用于配置属性源和从中解析属性。

### 7.13.1  Bean定义配置文件

Bean定义配置文件是核心容器中的一种机制，允许在不同环境中注册不同的bean。 单词环境对不同的用户来说意味着不同的东西，这个功能可以帮助许多用例，包括：

* 在开发中使用内存数据源，在QA或生产环境中查找来自JNDI的相同数据源
* 仅在将应用程序部署到性能环境时注册监视基础结构
* 为客户A和客户B部署注册bean的自定义实现

让我们考虑一个需要DataSource的实际应用程序中的第一个用例。 在测试环境中，配置可能如下所示：

```
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```

现在让我们考虑如何将此应用程序部署到QA或生产环境中，假设应用程序的数据源将在生产应用程序服务器的JNDI目录中注册。 我们的dataSource bean现在看起来像这样：

```
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

问题是如何根据当前环境在使用这两种变体之间切换。 随着时间的推移，Spring用户已经设计了许多方法来完成这项工作，通常依赖于系统环境变量和包含$ {placeholder}标记的XML &lt;import /&gt;语句的组合，这些标记根据值解析为正确的配置文件路径 一个环境变量。 Bean定义配置文件是核心容器功能，可为此问题提供解决方案。

如果我们概括上面特定于环境的bean定义的示例用例，我们最终需要在某些上下文中注册某些bean定义，而不是在其他上下文中。 您可以说您希望在情境A中注册某个bean定义的配置文件，在情况B中注册不同的配置文件。让我们首先看看如何更新我们的配置以反映这种需求。

#### @Profile

@Profile注释允许您在一个或多个指定的配置文件处于活动状态时指示组件符合注册条件。 使用上面的示例，我们可以重写dataSource配置，如下所示：

```
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```

```
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

如前所述，使用@Bean方法，您通常会选择使用编程JNDI查找：使用Spring的JndiTemplate / JndiLocatorDelegate帮助程序或上面显示的直接JNDI InitialContext用法，但不会强制您将返回类型声明为JndiObjectFactoryBean变量 FactoryBean类型。

@Profile可以用作元注释，用于创建自定义组合注释。 以下示例定义了一个自定义@Production批注，可用作@Profile（“生产”）的替代品：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

如果使用@Profile标记@Configuration类，则将绕过与该类关联的所有@Bean方法和@Import注释，除非一个或多个指定的配置文件处于活动状态。 如果使用@Profile（{“p1”，“p2”}）标记@Component或@Configuration类，则除非已激活配置文件“p1”和/或“p2”，否则不会注册/处理该类。 如果给定的配置文件以NOT运算符（！）作为前缀，则如果配置文件未处于活动状态，则将注册带注释的元素。 例如，给定@Profile（{“p1”，“！p2”}），如果配置文件“p1”处于活动状态或配置文件“p2”未激活，则会进行注册。



@Profile也可以在方法级别声明，以仅包括配置类的一个特定bean，例如 对于特定bean的替代变体：

```
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development")
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production")
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

对于@Bean方法的@Profile，可能会应用一个特殊的场景：对于相同Java方法名称的重载@Bean方法（类似于构造函数重载），需要在所有重载方法上一致地声明@Profile条件。如果条件不一致，则只有重载方法中第一个声明的条件才重要。因此，@ Profile不能用于选择具有特定参数签名的重载方法而不是另一个;同一个bean的所有工厂方法之间的分辨率遵循Spring的构造函数解析算法在创建时。

如果要定义具有不同配置文件条件的备用Bean，请使用通过@Bean name属性指向同一bean名称的不同Java方法名称，如上例所示。如果参数签名都是相同的（例如，所有变体都具有no-arg工厂方法），这是首先在有效的Java类中表示这种排列的唯一方法（因为只有一种方法可以特定的名称和参数签名）。

#### XML bean定义配置文件

XML对应物是&lt;beans&gt;元素的profile属性。 我们上面的示例配置可以在两个XML文件中重写，如下所示：

```
<beans profile="development"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>
```

```
<beans profile="production"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```

也可以避免在同一文件中使用split和nest &lt;beans /&gt;元素：

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="development">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
</beans>
```

spring-bean.xsd已被约束为仅允许这些元素作为文件中的最后一个元素。 这应该有助于提供灵活性，而不会在XML文件中引起混乱。

#### 激活profile

现在我们已经更新了配置，我们仍然需要指示Spring哪个配置文件处于活动状态。 如果我们现在开始我们的示例应用程序，我们会看到抛出NoSuchBeanDefinitionException，因为容器找不到名为dataSource的Spring bean。

激活配置文件可以通过多种方式完成，但最直接的方法是通过ApplicationContext以环境API编程方式：

```
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

此外，还可以通过spring.profiles.active属性以声明方式激活配置文件，该属性可以通过系统环境变量，JVM系统属性，web.xml中的servlet上下文参数或甚至作为JNDI中的条目来指定（请参阅第7.13节。 2，“PropertySource抽象”）。 在集成测试中，可以通过spring-test模块中的@ActiveProfiles注释声明活动配置文件（请参阅“使用环境配置文件配置上下文”一节）。

请注意，配置文件不是“任何 - 或”命题; 可以一次激活多个配置文件。 以编程方式，只需为setActiveProfiles（）方法提供多个配置文件名称，该方法接受String ... varargs：

```
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```

声明性地，spring.profiles.active可以接受以逗号分隔的配置文件名称列表：

```
-Dspring.profiles.active="profile1,profile2"
```

#### 默认profile

默认配置文件表示默认启用的配置文件。 考虑以下：

```
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

如果没有激活配置文件，将创建上面的dataSource; 这可以看作是为一个或多个bean提供默认定义的一种方法。 如果启用了任何配置文件，则默认配置文件将不适用。

可以使用环境上的setDefaultProfiles（）或声明性地使用spring.profiles.default属性更改默认配置文件的名称。

### 7.13.2  PropertySource抽象

Spring的Environment抽象通过可配置的属性源层次结构提供搜索操作。 要完整解释，请考虑以下事项：

```
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsFoo = env.containsProperty("foo");
System.out.println("Does my environment contain the 'foo' property? " + containsFoo);
```

在上面的代码片段中，我们看到了一种向Spring询问是否为当前环境定义foo属性的高级方法。 要回答此问题，Environment对象将对一组PropertySource对象执行搜索。 PropertySource是对任何键值对源的简单抽象，Spring的StandardEnvironment配置有两个PropertySource对象 - 一个表示JVM系统属性集（一个是System.getProperties（）），另一个表示系统环境集。 变量（a System.getenv（））。

这些默认属性源存在于StandardEnvironment中，以便在独立应用程序中使用。 StandardServletEnvironment填充了其他默认属性源，包括servlet配置和servlet上下文参数。 StandardPortletEnvironment类似地可以访问portlet配置和portlet上下文参数作为属性源。 两者都可以选择启用JndiPropertySource。 有关详细信息，请参阅javadocs。

具体地说，当使用StandardEnvironment时，如果运行时存在foo系统属性或foo环境变量，则对env.containsProperty（“foo”）的调用将返回true。

执行的搜索是分层的。 默认情况下，系统属性优先于环境变量，因此如果在调用env.getProperty（“foo”）期间恰好在两个位置都设置了foo属性，则系统属性值将为“win”并优先返回 环境变量。 请注意，属性值不会被合并，而是被前面的条目完全覆盖。

对于常见的StandardServletEnvironment，完整层次结构如下所示，顶部的最高优先级条目：

* ServletConfig参数（如果适用，例如在DispatcherServlet上下文的情况下）
* ServletContext参数（web.xml context-param条目）
* JNDI环境变量（“java：comp / env /”条目）
* JVM系统属性（“-D”命令行参数）
* JVM系统环境（操作系统环境变量）

最重要的是，整个机制是可配置的。 也许您有自定义的属性源，您希望将其集成到此搜索中。 没问题 - 只需实现并实例化您自己的PropertySource并将其添加到当前Environment的PropertySources集合中：

```
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

在上面的代码中，MyPropertySource在搜索中添加了最高优先级。 如果它包含foo属性，它将被检测并在任何其他PropertySource中的任何foo属性之前返回。 MutablePropertySources API公开了许多方法，允许精确操作属性源集。

### 7.13.3  @PropertySource

@PropertySource注释提供了一种方便的声明式机制，用于向Spring的环境添加PropertySource。

给定包含键/值对testbean.name = myTestBean的文件“app.properties”，以下@Configuration类使用@PropertySource，以便调用testBean.getName（）将返回“myTestBean”。

```
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
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

@PropertySource资源位置中存在的任何$ {...}占位符将根据已针对环境注册的属性源集合进行解析。 例如：

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

假设“my.placeholder”存在于已经注册的一个属性源中，例如 系统属性或环境变量，占位符将被解析为相应的值。 如果没有，则“default / path”将用作默认值。 如果未指定缺省值且无法解析属性，则将抛出IllegalArgumentException。

@PropertySource注释可根据Java 8约定重复。 但是，所有这些@PropertySource注释都需要在同一级别声明：直接在配置类上或在同一自定义注释中的元注释。 不建议混合直接注释和元注释，因为直接注释将有效地覆盖元注释。



### 7.13.4 占位符决议在statements \(Placeholder resolution in statements\)

从历史上看，元素中占位符的值只能针对JVM系统属性或环境变量进行解析。 情况不再如此。 因为环境抽象集成在整个容器中，所以很容易通过它来解决占位符的分辨率。 这意味着您可以以任何您喜欢的方式配置解析过程：更改搜索系统属性和环境变量的优先级，或者完全删除它们; 根据需要将您自己的属性源添加到混合中。

具体而言，只要在环境中可用，无论客户属性在何处定义，以下语句都可以工作：

```
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```




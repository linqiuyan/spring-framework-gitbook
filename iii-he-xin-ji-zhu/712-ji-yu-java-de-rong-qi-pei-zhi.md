## 7.12 基于Java的容器配置

### 7.12.1 基本概念：@Bean和@Configuration

Spring新的Java配置支持中的中心工件是@Configuration-annotated类和@ Bean-annotated方法。

@Bean注释用于指示方法实例化，配置和初始化由Spring IoC容器管理的新对象。 对于那些熟悉Spring的&lt;beans /&gt; XML配置的人来说，@ Boan注释扮演的角色与&lt;bean /&gt;元素相同。 您可以将@Bean带注释的方法与任何Spring @Component一起使用，但是，它们最常用于@Configuration bean。

使用@Configuration注释类表示其主要目的是作为bean定义的源。 此外，@ Configuration类允许通过简单地调用同一个类中的其他@Bean方法来定义bean间依赖关系。 最简单的@Configuration类如下：

```
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

上面的AppConfig类将等效于以下Spring &lt;beans /&gt; XML：

```
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

---

完整@Configuration vs 'lite'@Bean模式？

当@Bean方法在未使用@Configuration注释的类中声明时，它们被称为以'lite'模式处理。 在@Component或甚至普通旧类中声明的Bean方法将被视为'lite'，其中包含类的主要目的不同，而@Bean方法只是一种奖励。 例如，服务组件可以通过在每个适用的组件类上使用额外的@Bean方法将管理视图公开给容器。 在这种情况下，@ Bean方法是一种简单的通用工厂方法机制。

与完整的@Configuration不同，lite @Bean方法不能声明bean间依赖关系。 相反，它们对其包含组件的内部状态进行操作，并且可选地对它们可以声明的参数进行操作。 因此，这样的@Bean方法不应该调用其他@Bean方法; 每个这样的方法实际上只是一个特定bean引用的工厂方法，没有任何特殊的运行时语义。 这里的积极副作用是不必在运行时应用CGLIB子类，因此在类设计方面没有限制（即包含类可能是最终的等等）。

在常见的场景中，@ Bean方法将在@Configuration类中声明，确保始终使用“完整”模式，因此交叉方法引用将被重定向到容器的生命周期管理。 这将防止通过常规Java调用意外地调用相同的@Bean方法，这有助于减少在“精简”模式下操作时难以跟踪的细微错误。

---

@Bean和@Configuration注释将在下面的部分中进行深入讨论。 首先，我们将介绍使用基于Java的配置创建弹簧容器的各种方法。

### 7.12.2 使用AnnotationConfigApplicationContext实例化Spring容器

下面的部分记录了Spring的AnnotationConfigApplicationContext，Spring 3.0中的新增内容。 这个多功能的ApplicationContext实现不仅能够接受@Configuration类作为输入，还能接受使用JSR-330元数据注释的普通@Component类和类。

当@Configuration类作为输入提供时，@ Configuration类本身被注册为bean定义，并且类中所有声明的@Bean方法也被注册为bean定义。

当提供@Component和JSR-330类时，它们被注册为bean定义，并且假设在必要时在这些类中使用诸如@Autowired或@Inject之类的DI元数据。

#### 结构简单

与实例化ClassPathXmlApplicationContext时将Spring XML文件用作输入的方式大致相同，在实例化AnnotationConfigApplicationContext时，可以将@Configuration类用作输入。 这允许Spring容器的完全无XML使用：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

如上所述，AnnotationConfigApplicationContext不仅限于使用@Configuration类。 任何@Component或JSR-330带注释的类都可以作为输入提供给构造函数。 例如：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

以上假设MyServiceImpl，Dependency1和Dependency2使用Spring依赖注入注释，例如@Autowired。

#### 使用register（Class &lt;？&gt; ...）以编程方式构建容器

可以使用no-arg构造函数实例化AnnotationConfigApplicationContext，然后使用register（）方法进行配置。 在以编程方式构建AnnotationConfigApplicationContext时，此方法特别有用。

```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

#### 使用扫描启用组件扫描（String ...）

要启用组件扫描，只需注释@Configuration类，如下所示：

```
@Configuration
@ComponentScan(basePackages = "com.acme")
public class AppConfig  {
    ...
}
```

有经验的Spring用户将熟悉与Spring的上下文相同的XML声明：namespace

```
<beans>
    <context:component-scan base-package="com.acme"/>
</beans>
```

在上面的示例中，将扫描com.acme包，查找任何@ Component-annotated类，这些类将在容器中注册为Spring bean定义。 AnnotationConfigApplicationContext公开scan（String ...）方法以允许相同的组件扫描功能：

```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

请记住@Configuration类是使用@Component进行元注释的，因此它们是组件扫描的候选者！ 在上面的例子中，假设AppConfig是在com.acme包（或下面的任何包）中声明的，它将在调用scan（）期间被拾取，并且在refresh（）时，它的所有@Bean方法都将被处理并且 在容器中注册为bean定义。

#### 使用AnnotationConfigWebApplicationContext支持Web应用程序

AnnotationConfigApplicationContext的WebApplicationContext变体与AnnotationConfigWebApplicationContext一起提供。 配置Spring ContextLoaderListener servlet侦听器，Spring MVC DispatcherServlet等时可以使用此实现。以下是配置典型Spring MVC Web应用程序的web.xml代码段。 注意contextClass context-param和init-param的使用：

```
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

### 7.12.3 使用@Bean注释

@Bean是方法级注释，是XML &lt;bean /&gt;元素的直接模拟。 注释支持&lt;bean /&gt;提供的一些属性，例如：init-method，destroy-method，autowiring和name。

您可以在@Configuration-annotated或@Component-annotated类中使用@Bean批注。

#### 声明一个bean

要声明bean，只需使用@Bean批注注释方法即可。 您可以使用此方法在指定为方法返回值的类型的ApplicationContext中注册bean定义。 默认情况下，bean名称将与方法名称相同。 以下是@Bean方法声明的简单示例：

```
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

前面的配置完全等同于以下Spring XML：

```
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

这两个声明都在ApplicationContext中创建一个名为transferService的bean，绑定到TransferServiceImpl类型的对象实例：

```
transferService -> com.acme.TransferServiceImpl
```

您还可以使用接口（或基类）返回类型声明您的@Bean方法：

```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

但是，这会将高级类型预测的可见性限制为指定的接口类型（TransferService），然后，只有在实例化受影响的单例bean后，容器才知道完整类型（TransferServiceImpl）。 非延迟单例bean根据其声明顺序进行实例化，因此您可能会看到不同的类型匹配结果，具体取决于另一个组件何时尝试通过非声明类型进行匹配（例如@Autowired TransferServiceImpl，它只会解析一次“transferService” bean已被实例化）。

_如果您始终通过声明的服务接口引用您的类型，则@Bean返回类型可以安全地加入该设计决策。 但是，对于实现多个接口的组件或可能由其实现类型引用的组件，更安全地声明可能的最具体的返回类型（至少与引用您的bean的注入点所需的具体相同）。_

#### Bean依赖项

@Bean注释方法可以有任意数量的参数来描述构建该bean所需的依赖关系。 例如，如果我们的TransferService需要AccountRepository，我们可以通过方法参数实现该依赖：

```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

解析机制与基于构造函数的依赖注入非常相似，有关详细信息，请参阅相关部分。

#### 接收生命周期回调

使用@Bean批注定义的任何类都支持常规生命周期回调，并且可以使用JSR-250中的@PostConstruct和@PreDestroy批注，有关详细信息，请参阅JSR-250批注。

完全支持常规的Spring生命周期回调。 如果bean实现InitializingBean，DisposableBean或Lifecycle，则它们各自的方法由容器调用。

还完全支持标准的\* Aware接口集，例如BeanFactoryAware，BeanNameAware，MessageSourceAware，ApplicationContextAware等。

@Bean注释支持指定任意初始化和销毁回调方法，就像bean元素上的Spring XML的init-method和destroy-method属性一样：

```
public class Foo {

    public void init() {
        // initialization logic
    }
}

public class Bar {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public Foo foo() {
        return new Foo();
    }

    @Bean(destroyMethod = "cleanup")
    public Bar bar() {
        return new Bar();
    }
}
```

默认情况下，使用Java配置定义的具有公共关闭或关闭方法的bean会自动使用销毁回调登记。 如果您有一个公共关闭或关闭方法，并且您不希望在容器关闭时调用它，只需将@Bean（destroyMethod =“”）添加到您的bean定义中以禁用默认（推断）模式。

默认情况下，您可能希望对通过JNDI获取的资源执行此操作，因为其生命周期在应用程序外部进行管理。 特别是，请确保始终为DataSource执行此操作，因为已知它在Java EE应用程序服务器上存在问题。

```
@Bean(destroyMethod="")
public DataSource dataSource() throws NamingException {
    return (DataSource) jndiTemplate.lookup("MyDS");
}
```

此外，使用@Bean方法，您通常会选择使用编程JNDI查找：使用Spring的JndiTemplate / JndiLocatorDelegate帮助程序或直接JNDI InitialContext用法，但不使用JndiObjectFactoryBean变量，它会强制您将返回类型声明为FactoryBean类型而不是 实际的目标类型，使其更难用于其他打算在此处引用所提供资源的@Bean方法中的交叉引用调用。

当然，在上面的Foo的情况下，在构造期间直接调用init（）方法同样有效：

```
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        Foo foo = new Foo();
        foo.init();
        return foo;
    }

    // ...
}
```

当您直接使用Java工作时，您可以使用对象执行任何您喜欢的操作，而不必总是依赖于容器生命周期！

#### 指定bean范围

##### 使用@Scope注释

您可以指定使用@Bean批注定义的bean应具有特定范围。 您可以使用Bean Scopes部分中指定的任何标准作用域。

默认范围是单例，但您可以使用@Scope批注覆盖它：

```
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```

##### @Scope和scoped-proxy

Spring提供了一种通过作用域代理处理作用域依赖项的便捷方式。 使用XML配置时创建此类代理的最简单方法是&lt;aop：scoped-proxy /&gt;元素。 使用@Scope批注在Java中配置bean提供了与proxyMode属性的等效支持。 默认值为无代理（ScopedProxyMode.NO），但您可以指定ScopedProxyMode.TARGET\_CLASS或ScopedProxyMode.INTERFACES。

如果使用Java将XML参考文档（参见前面的链接）中的作用域代理示例移植到我们的@Bean，它将如下所示：

```
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```

#### 自定义bean命名

默认情况下，配置类使用@Bean方法的名称作为结果bean的名称。 但是，可以使用name属性覆盖此功能。

```
@Configuration
public class AppConfig {

    @Bean(name = "myFoo")
    public Foo foo() {
        return new Foo();
    }
}
```

#### Bean别名

正如第7.3.1节“命名bean”中所讨论的，有时需要为单个bean提供多个名称，也称为bean别名。 @Bean批注的name属性为此接受String数组。

```
@Configuration
public class AppConfig {

    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

#### bean的描述

有时提供bean的更详细的文本描述是有帮助的。 当bean暴露（可能通过JMX）用于监视目的时，这可能特别有用。

要向@Bean添加描述，可以使用@Description注释：

```
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Foo foo() {
        return new Foo();
    }
}
```

### 7.12.4 使用@Configuration注释

@Configuration是一个类级注释，指示对象是bean定义的源。 @Configuration类通过公共@Bean注释方法声明bean。 在@Configuration类上调用@Bean方法也可用于定义bean间依赖项。 有关一般介绍，请参见第7.12.1节“基本概念：@Bean和@Configuration”。

#### 注入bean间依赖关系

当@Beans彼此依赖时，表达该依赖关系就像让一个bean方法调用另一个一样简单：

```
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        return new Foo(bar());
    }

    @Bean
    public Bar bar() {
        return new Bar();
    }
}
```

在上面的示例中，foo bean通过构造函数注入接收对bar的引用。

这种声明bean间依赖关系的方法仅在@Configuration类中声明@Bean方法时才有效。 您不能使用普通的@Component类声明bean间依赖项。

#### 查找方法注入

如前所述，查找方法注入是一项很少使用的高级功能。 在单例范围的bean依赖于原型范围的bean的情况下，它很有用。 将Java用于此类配置提供了实现此模式的自然方法。

```
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

使用Java配置支持，您可以创建CommandManager的子类，其中覆盖抽象createCommand（）方法，使其查找新的（原型）命令对象：

```
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with command() overridden
    // to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```

#### 有关基于Java的配置如何在内部工作的更多信息

以下示例显示了被调用两次的@Bean注释方法：

```
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```

clientDao（）在clientService1（）中调用一次，在clientService2（）中调用一次。 由于此方法创建了ClientDaoImpl的新实例并将其返回，因此通常需要2个实例（每个服务一个）。 这肯定会有问题：在Spring中，实例化的bean默认具有单例范围。 这就是魔术所带来的：所有@Configuration类在启动时都使用CGLIB进行子类化。 在子类中，子方法在调用父方法并创建新实例之前，首先检查容器是否有任何缓存（作用域）bean。 请注意，从Spring 3.2开始，不再需要将CGLIB添加到类路径中，因为CGLIB类已在org.springframework.cglib下重新打包并直接包含在spring-core JAR中。

根据bean的范围，行为可能会有所不同。 我们在这里谈论单例。

由于CGLIB在启动时动态添加功能，因此存在一些限制，特别是配置类不能是最终的。 但是，从4.3开始，配置类允许使用任何构造函数，包括使用@Autowired或单个非默认构造函数声明进行默认注入。

如果您希望避免任何CGLIB强加的限制，请考虑在非@Configuration类上声明您的@Bean方法，例如 而是在普通的@Component类上。 然后，@ Bean方法之间的跨方法调用不会被截获，因此您必须在构造函数或方法级别专门依赖依赖注入。

### 7.12.5 编写基于Java的配置

#### 使用@Import注释

就像在Spring XML文件中使用&lt;import /&gt;元素来帮助模块化配置一样，@ Immort注释允许从另一个配置类加载@Bean定义：

```
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```

现在，在实例化上下文时，不需要同时指定ConfigA.class和ConfigB.class，只需要显式提供ConfigB：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

这种方法简化了容器实例化，因为只需要处理一个类，而不是要求开发人员在构造期间记住可能大量的@Configuration类。

从Spring Framework 4.2开始，@ Immort还支持对常规组件类的引用，类似于AnnotationConfigApplicationContext.register方法。 如果您想避免组件扫描，使用一些配置类作为显式定义所有组件的入口点，这将特别有用。

#### 注入对导入的@Bean定义的依赖关系

上面的例子有效，但很简单。 在大多数实际情况中，bean将跨配置类相互依赖。 使用XML时，这本身不是问题，因为不涉及编译器，可以简单地声明ref =“someBean”并相信Spring会在容器初始化期间解决它。 当然，在使用@Configuration类时，Java编译器会对配置模型施加约束，因为对其他bean的引用必须是有效的Java语法。

幸运的是，解决这个问题很简单。 正如我们已经讨论过的，@ Node方法可以有任意数量的参数来描述bean的依赖关系。 让我们考虑一个更真实的场景，其中包含几个@Configuration类，每个类都取决于其他类中声明的bean：

```
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

还有另一种方法可以达到相同的效果。 请记住，@ Configuration类最终只是容器中的另一个bean：这意味着他们可以像任何其他bean一样利用@Autowired和@Value注入等！

确保以这种方式注入的依赖项只是最简单的。 @Configuration类在上下文初始化期间很早就被处理，并且强制以这种方式注入依赖项可能导致意外的早期初始化。 尽可能采用基于参数的注入，如上例所示。

此外，通过@Bean特别注意BeanPostProcessor和BeanFactoryPostProcessor定义。 这些通常应该声明为静态@Bean方法，而不是触发其包含配置类的实例化。 否则，@ Autowired和@Value将无法在配置类本身上工作，因为它过早地被创建为bean实例。

```
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    @Autowired
    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

仅在Spring Framework 4.3中支持@Configuration类中的构造函数注入。 另请注意，如果目标bean只定义了一个构造函数，则无需指定@Autowired; 在上面的示例中，RepositoryConfig构造函数中不需要@Autowired。

在上面的场景中，使用@Autowired可以很好地工作并提供所需的模块化，但确定声明自动装配的bean定义的确切位置仍然有些模棱两可。 例如，作为查看ServiceConfig的开发人员，您如何确切地知道@Autowired AccountRepository bean的声明位置？ 它在代码中并不明确，这可能就好了。 请记住，Spring Tool Suite提供的工具可以呈现图表，显示所有内容的连线方式 - 这可能就是您所需要的。 此外，您的Java IDE可以轻松找到AccountRepository类型的所有声明和用法，并将快速显示返回该类型的@Bean方法的位置。

如果这种歧义是不可接受的，并且您希望从IDE中从一个@Configuration类直接导航到另一个@Configuration类，请考虑自动对配置类进行自动装配：

```
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}
```

在上面的情况中，它是完全明确的，其中定义了AccountRepository。 但是，ServiceConfig现在与RepositoryConfig紧密耦合; 这是权衡。 通过使用基于接口的或基于类的抽象@Configuration类，可以在某种程度上减轻这种紧密耦合。 考虑以下：

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();
}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }
}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class})  // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

现在，ServiceConfig与具体的DefaultRepositoryConfig松散耦合，内置的IDE工具仍然很有用：开发人员很容易获得RepositoryConfig实现的类型层次结构。 通过这种方式，导航@Configuration类及其依赖项与导航基于接口的代码的常规过程没有什么不同。

如果您想影响某些bean的启动创建顺序，可以考虑将其中一些声明为@Lazy（在首次访问时创建而不是在启动时创建）或在某些其他bean上声明为@DependsOn（确保其他特定bean将是 在当前bean之前创建，超出后者直接依赖所暗示的内容）。

#### 有条件地包括@Configuration类或@Bean方法

基于某些任意系统状态，有条件地启用或禁用完整的@Configuration类，甚至单独的@Bean方法通常很有用。 一个常见的例子是只有在Spring环境中启用了特定的配置文件时才使用@Profile注释来激活bean（有关详细信息，请参见第7.13.1节“Bean定义配置文件”）。

@Profile注释实际上是使用一个名为@Conditional的更灵活的注释实现的。 @Conditional批注指示在注册@Bean之前应该参考的特定org.springframework.context.annotation.Condition实现。

Condition接口的实现只提供一个返回true或false的matches（...）方法。 例如，以下是用于@Profile的实际Condition实现：

```
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    if (context.getEnvironment() != null) {
        // Read the @Profile annotation attributes
        MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
        if (attrs != null) {
            for (Object value : attrs.get("value")) {
                if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                    return true;
                }
            }
            return false;
        }
    }
    return true;
}
```

有关更多详细信息，请参阅@Conditional javadocs。

#### 结合Java和XML配置

Spring的@Configuration类支持并非旨在成为Spring XML的100％完全替代品。 诸如Spring XML命名空间之类的一些工具仍然是配置容器的理想方式。 在XML方便或必要的情况下，您可以选择：使用例如ClassPathXmlApplicationContext以“以XML为中心”的方式实例化容器，或者使用AnnotationConfigApplicationContext和@ImportResource注释以“以Java为中心”的方式实例化容器 根据需要导入XML。

#### 以XML为中心的@Configuration类的使用

最好从XML引导Spring容器，并以ad-hoc方式包含@Configuration类。 例如，在使用Spring XML的大型现有代码库中，根据需要创建@Configuration类并将其包含在现有XML文件中会更容易。 下面你将找到在这种“以XML为中心”的情况下使用@Configuration类的选项。

请记住，@ Configuration类最终只是容器中的bean定义。 在此示例中，我们创建一个名为AppConfig的@Configuration类，并将其作为&lt;bean /&gt;定义包含在system-test-config.xml中。 由于&lt;context：annotation-config /&gt;已打开，容器将识别@Configuration批注并正确处理AppConfig中声明的@Bean方法。

```
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }
}
```

**system-test-config.xml**:

```
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

**jdbc.properties**:

```
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=
```

```
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

在上面的system-test-config.xml中，AppConfig &lt;bean /&gt;不声明id元素。 虽然这样做是可以接受的，但是没有其他bean可以引用它，并且不太可能通过名称从容器中显式获取它。 与DataSource bean类似 - 它只是按类型自动装配，因此不严格要求显式bean ID。

因为@Configuration是使用@Component进行元注释的，所以@Configuration-annotated类自动成为组件扫描的候选者。 使用与上面相同的方案，我们可以重新定义system-test-config.xml以利用组件扫描。 请注意，在这种情况下，我们不需要显式声明&lt;context：annotation-config /&gt;，因为&lt;context：component-scan /&gt;启用相同的功能。

**system-test-config.xml**:

```
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

#### @Configuration以类为中心使用带@ImportResource的XML

在@Configuration类是配置容器的主要机制的应用程序中，仍然可能需要使用至少一些XML。 在这些场景中，只需使用@ImportResource并仅根据需要定义尽可能多的XML。 这样做可以实现“以Java为中心”的方法来配置容器并将XML保持在最低限度。

```
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}
```

```
properties-config.xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```

```
jdbc.properties
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=
```

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```




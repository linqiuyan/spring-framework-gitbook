## 7.6 自定义bean的本质

### 7.6.1 生命周期回调

要与容器的bean生命周期管理进行交互，可以实现Spring InitializingBean和DisposableBean接口。 容器为前者调用afterPropertiesSet\(\)，为后者调用destroy\(\)以允许bean在初始化和销毁bean时执行某些操作。

JSR-250 @PostConstruct和@PreDestroy注释通常被认为是在现代Spring应用程序中接收生命周期回调的最佳实践。 使用这些注释意味着您的bean不会耦合到Spring特定的接口。 有关详细信息，请参见第7.9.8节“@ PostConstruct和@PreDestroy”。

在内部，Spring Framework使用BeanPostProcessor实现来处理它可以找到的任何回调接口并调用适当的方法。 如果您需要自定义功能或Spring不提供的其他生命周期行为，您可以自己实现BeanPostProcessor。 有关更多信息，请参见第7.8节“容器扩展点”。

除了初始化和销毁回调之外，Spring管理的对象还可以实现Lifecycle接口，以便这些对象可以参与由容器自身生命周期驱动的启动和关闭过程。

本节描述了生命周期回调接口。

#### 初始化回调

org.springframework.beans.factory.InitializingBean接口允许bean在容器设置bean的所有必要属性后执行初始化工作。 InitializingBean接口指定一个方法：

```
void afterPropertiesSet() throws Exception;
```

建议您不要使用InitializingBean接口，因为它会不必要地将代码耦合到Spring。 或者，使用@PostConstruct注释或指定POJO初始化方法。 对于基于XML的配置元数据，可以使用init-method属性指定具有void无参数签名的方法的名称。 使用Java配置，您可以使用@Bean的initMethod属性，请参阅“接收生命周期回调”一节。 例如，以下内容：

```
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

......与......完全一样

```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```
public class AnotherExampleBean implements InitializingBean {

    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

但不会将代码耦合到Spring。

#### 销毁回调

实现org.springframework.beans.factory.DisposableBean接口允许bean在包含它的容器被销毁时获得回调。 DisposableBean接口指定一个方法：

```
void destroy() throws Exception;
```

建议您不要使用DisposableBean回调接口，因为它会不必要地将代码耦合到Spring。 或者，使用@PreDestroy批注或指定bean定义支持的泛型方法。 使用基于XML的配置元数据，可以在&lt;bean /&gt;上使用destroy-method属性。 使用Java配置，您可以使用@Bean的destroyMethod属性，请参阅“接收生命周期回调”一节。 例如，以下定义：

```
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

与以下内容完全相同：

```
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```
public class AnotherExampleBean implements DisposableBean {

    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

但不会将代码耦合到Spring。

_可以为&lt;bean&gt;元素的destroy-method属性分配一个特殊的（推断的）值，该值指示Spring自动检测特定bean类（任何实现java.lang.AutoCloseable或java的类）的公共关闭或关闭方法。 io.Closeable因此匹配）。 此特殊（推断）值也可以在&lt;beans&gt;元素的default-destroy-method属性上设置，以将此行为应用于整个bean集（请参阅“默认初始化和销毁方法”一节）。 请注意，这是Java配置的默认行为。_

#### 默认初始化和销毁​​方法

当您编写初始化和销毁​​不使用特定于Spring的InitializingBean和DisposableBean回调接口的方法回调时，通常会编写名称为init（），initialize（），dispose（）等的方法。理想情况下，此类生命周期回调方法的名称在项目中是标准化的，以便所有开发人员使用相同的方法名称并确保一致性。

您可以配置Spring容器以查找命名初始化并销毁每个bean上的回调方法名称。这意味着，作为应用程序开发人员，您可以编写应用程序类并使用名为init（）的初始化回调，而无需为每个bean定义配置init-method =“init”属性。 Spring IoC容器在创建bean时调用该方法（并且符合前面描述的标准生命周期回调契约）。此功能还强制执行初始化和销毁​​方法回调的一致命名约定。

假设您的初始化回调方法名为init（），而destroy回调方法名为destroy（）。您的类将类似于以下示例中的类。

```
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

```
<beans default-init-method="init">

    <bean id="blogService" class="com.foo.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

顶级&lt;beans /&gt;元素属性上存在default-init-method属性会导致Spring IoC容器将bean上的init方法识别为初始化方法回调。当bean被创建和组装时，如果bean类具有这样的方法，则在适当的时候调用它。

您可以通过在顶级&lt;beans /&gt;元素上使用default-destroy-method属性来类似地配置destroy方法回调（在XML中）。

如果现有bean类已经具有以约定方式命名的回调方法，则可以通过使用init-method和&lt;bean /&gt;的destroy-method属性指定（在XML中，即）方法名称来覆盖缺省值。本身。

Spring容器保证在为bean提供所有依赖项之后立即调用已配置的初始化回调。因此，在原始bean引用上调用初始化回调，这意味着AOP拦截器等尚未应用于bean。首先完全创建目标bean，然后应用带有拦截器链的AOP代理（例如）。如果目标bean和代理是分开定义的，那么您的代码甚至可以绕过代理与原始目标bean进行交互。因此，将拦截器应用于init方法是不一致的，因为这样做会将目标bean的生命周期与其代理/拦截器耦合在一起，并在代码直接与原始目标bean交互时留下奇怪的语义。

#### 结合生命周期机制

从Spring 2.5开始，您有三个控制bean生命周期行为的选项：InitializingBean和DisposableBean回调接口; 自定义init（）和destroy（）方法; 以及@PostConstruct和@PreDestroy注释。 您可以组合这些机制来控制给定的bean。

_如果为bean配置了多个生命周期机制，并且每个机制都配置了不同的方法名称，则每个配置的方法都按照下面列出的顺序执行。 但是，如果为初始化方法配置了相同的方法名称（例如，init（） - 对于多个这些生命周期机制，该方法将执行一次，如上一节中所述。_

为同一个bean配置的多个生命周期机制，具有不同的初始化方法，如下所示：

* 使用@PostConstruct注释的方法
* afterPropertiesSet（）由InitializingBean回调接口定义
* 自定义配置的init（）方法

Destroy方法以相同的顺序调用：

* 用@PreDestroy注释的方法
* destroy（），由DisposableBean回调接口定义
* 自定义配置的destroy（）方法

#### 启动和关闭回调

Lifecycle接口为任何具有自己生命周期要求的对象定义基本方法（例如，启动和停止某些后台进程）：

```
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

任何Spring管理的对象都可以实现该接口。 然后，当ApplicationContext本身接收到开始和停止信号时，例如， 对于运行时的停止/重新启动方案，它会将这些调用级联到该上下文中定义的所有生命周期实现。 它通过委托给LifecycleProcessor来实现：

```
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

请注意，LifecycleProcessor本身是Lifecycle接口的扩展。 它还添加了另外两种方法来响应刷新和关闭的上下文。

_请注意，常规org.springframework.context.Lifecycle接口只是显式启动/停止通知的简单合约，并不意味着在上下文刷新时自动启动。 考虑实现org.springframework.context.SmartLifecycle，以便对特定bean的自动启动进行细粒度控制（包括启动阶段）。 此外，请注意，在销毁之前不保证停止通知：在常规关闭时，所有Lifecycle bean将首先在传播一般销毁回调之前收到停止通知; 但是，在上下文生命周期中的热刷新或中止刷新尝试时，只会调用destroy方法。_

启动和关闭调用的顺序非常重要。 如果任何两个对象之间存在“依赖”关系，则依赖方将在其依赖之后启动，并且它将在其依赖之前停止。 但是，有时直接依赖性是未知的。 您可能只知道某种类型的对象应该在另一种类型的对象之前开始。 在这些情况下，SmartLifecycle接口定义了另一个选项，即在其超级接口Phased上定义的getPhase（）方法。

```
public interface Phased {

    int getPhase();
}
```

```
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

启动时，具有最低相位的对象首先开始，停止时，遵循相反的顺序。因此，实现SmartLifecycle并且其getPhase\(\)方法返回Integer.MIN\_VALUE的对象将是第一个开始和最后一个停止的对象。在频谱的另一端，相位值Integer.MAX\_VALUE将指示对象应该最后启动并首先停止（可能是因为它依赖于正在运行的其他进程）。在考虑相位值时，同样重要的是要知道任何未实现SmartLifecycle的“正常”生命周期对象的默认阶段都是0.因此，任何负相位值都表示对象应该在这些标准组件之前启动（和在他们之后停止），反之亦然，任何正相位值。

如您所见，SmartLifecycle定义的stop方法接受回调。在该实现的关闭过程完成之后，任何实现都必须调用该回调的run\(\)方法。这样就可以在必要时启用异步关闭，因为LifecycleProcessor接口的默认实现DefaultLifecycleProcessor将等待每个阶段内对象组的超时值来调用该回调。默认的每阶段超时为30秒。您可以通过在上下文中定义名为“lifecycleProcessor”的bean来覆盖缺省生命周期处理器实例。如果您只想修改超时，那么定义以下内容就足够了：

```
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

如上所述，LifecycleProcessor接口还定义了用于刷新和关闭上下文的回调方法。后者将简单地驱动关闭过程，就好像已经显式调用了stop（），但是当上下文关闭时会发生。另一方面，“刷新”回调启用了SmartLifecycle bean的另一个功能。刷新上下文（在所有对象都已实例化并初始化之后），将调用该回调，此时默认生命周期处理器将检查每个SmartLifecycle对象的isAutoStartup（）方法返回的布尔值。如果为“true”，那么该对象将在该点启动，而不是等待显式调用上下文或其自己的start（）方法（与上下文刷新不同，上下文启动不会自动发生在标准上下文实现中） 。 “阶段”值以及任何“依赖”关系将以与上述相同的方式确定启动顺序。

#### 在非Web应用程序中正常关闭Spring IoC容器

本节仅适用于非Web应用程序。 Spring的基于Web的ApplicationContext实现已经具有代码，可以在关闭相关Web应用程序时正常关闭Spring IoC容器。

如果您在非Web应用程序环境中使用Spring的IoC容器; 例如，在富客户端桌面环境中; 您使用JVM注册了一个关闭钩子。 这样做可确保正常关闭并在单例bean上调用相关的destroy方法，以便释放所有资源。 当然，您仍然必须正确配置和实现这些destroy回调。

要注册关闭挂钩，请调用ConfigurableApplicationContext接口上声明的registerShutdownHook\(\)方法：

```
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

### 7.6.2  ApplicationContextAware和BeanNameAware

当ApplicationContext创建实现org.springframework.context.ApplicationContextAware接口的对象实例时，将为该实例提供对该ApplicationContext的引用。

```
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

因此，bean可以通过ApplicationContext接口以编程方式操作创建它们的ApplicationContext，或者通过将引用强制转换为此接口的已知子类（例如ConfigurableApplicationContext）来公开其他功能。 一种用途是对其他bean进行编程检索。 有时这种能力很有用; 但是，通常你应该避免它，因为它将代码耦合到Spring并且不遵循Inversion of Control样式，其中协作者作为属性提供给bean。 ApplicationContext的其他方法提供对文件资源的访问，发布应用程序事件和访问MessageSource。 第7.15节“ApplicationContext的附加功能”中介绍了这些附加功能

从Spring 2.5开始，自动装配是另一种获取ApplicationContext引用的替代方法。 “传统”构造函数和byType自动装配模式（如第7.4.5节“自动装配协作者”中所述）可以分别为构造函数参数或setter方法参数提供ApplicationContext类型的依赖关系。 要获得更大的灵活性，包括自动装配字段和多参数方法的能力，请使用基于注释的新自动装配功能。 如果这样做，ApplicationContext将自动装入一个字段，构造函数参数或方法参数，如果相关的字段，构造函数或方法带有@Autowired注释，则该参数需要ApplicationContext类型。 有关更多信息，请参见第7.9.2节“@Autowired”。

当ApplicationContext创建实现org.springframework.beans.factory.BeanNameAware接口的类时，将为该类提供对其关联对象定义中定义的名称的引用。

```
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

在普通bean属性填充之后但在初始化回调之前调用回调，例如InitializingBean afterPropertiesSet或自定义init方法。

### 7.6.3 其他Aware接口

除了上面讨论的ApplicationContextAware和BeanNameAware之外，Spring还提供了一系列Aware接口，允许bean向容器指示它们需要某种基础结构依赖性。 最重要的Aware接口总结如下 - 作为一般规则，名称是依赖类型的良好指示：

| Name | Injected Dependency | Explained in…​ |
| :--- | :--- | :--- |
| `ApplicationContextAware` | 声明ApplicationContext | [Section 7.6.2, “ApplicationContextAware and BeanNameAware”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-aware) |
| `ApplicationEventPublisherAware` | Event publisher of the enclosing`ApplicationContext` | [Section 7.15, “Additional capabilities of the ApplicationContext”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#context-introduction) |
| `BeanClassLoaderAware` | 用于加载bean类的类加载器。 | [Section 7.3.2, “Instantiating beans”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-class) |
| `BeanFactoryAware` | 声明BeanFactory | [Section 7.6.2, “ApplicationContextAware and BeanNameAware”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-aware) |
| `BeanNameAware` | 声明bean的名称 | [Section 7.6.2, “ApplicationContextAware and BeanNameAware”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-aware) |
| `BootstrapContextAware` | 资源adapterBootstrapContext容器运行。通常仅在JCA awareApplicationContexts中可用 | [Chapter 32,JCA CCI](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#cci) |
| `LoadTimeWeaverAware` | 定义的weaver用于在加载时处理类定义 | [Section 11.8.4, “Load-time weaving with AspectJ in the Spring Framework”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#aop-aj-ltw) |
| `MessageSourceAware` | 用于解析消息的已配置策略（支持参数化和国际化） | [Section 7.15, “Additional capabilities of the ApplicationContext”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#context-introduction) |
| `NotificationPublisherAware` | Spring JMX通知发布者 | [Section 31.7, “Notifications”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#jmx-notifications) |
| `PortletConfigAware` | CurrentPortletConfigthe容器运行。仅在支持Web的SpringApplicationContext中有效 | [Chapter 25,Portlet MVC Framework](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#portlet) |
| `PortletContextAware` | CurrentPortletContext容器运行。仅在支持Web的SpringApplicationContext中有效 | [Chapter 25,Portlet MVC Framework](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#portlet) |
| `ResourceLoaderAware` | 配置的加载程序，用于对资源进行低级访问 | [Chapter 8,Resources](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#resources) |
| `ServletConfigAware` | CurrentServletConfigthe容器运行。仅在支持Web的SpringApplicationContext中有效 | [Chapter 22,Web MVC framework](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#mvc) |
| `ServletContextAware` | CurrentServletContext容器运行。仅在支持Web的SpringApplicationContext中有效 | [Chapter 22,Web MVC framework](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#mvc) |

再次注意，这些接口的使用将您的代码与Spring API联系起来，并且不遵循Inversion of Control样式。 因此，建议将它们用于需要以编程方式访问容器的基础结构bean。

## 




## 7.15  ApplicationContext的其他功能

正如章节介绍中所讨论的，org.springframework.beans.factory包提供了管理和操作bean的基本功能，包括以编程方式。 org.springframework.context包添加了ApplicationContext接口，该接口扩展了BeanFactory接口，此外还扩展了其他接口，以更面向应用程序框架的方式提供其他功能。 许多人以完全声明的方式使用ApplicationContext，甚至不以编程方式创建它，而是依赖于诸如ContextLoader之类的支持类来自动实例化ApplicationContext，作为Java EE Web应用程序的正常启动过程的一部分。

为了以更加面向框架的样式增强BeanFactory功能，上下文包还提供以下功能：

* 通过MessageSource接口访问i18n风格的消息。
* 通过ResourceLoader接口访问URL和文件等资源。
* 事件发布，即通过使用ApplicationEventPublisher接口实现ApplicationListener接口的bean。
* 加载多个（分层）上下文，允许每个上下文通过HierarchicalBeanFactory接口聚焦于一个特定层，例如应用程序的Web层。

### 7.15.1 使用MessageSource进行国际化

ApplicationContext接口扩展了一个名为MessageSource的接口，因此提供了国际化（i18n）功能。 Spring还提供了HierarchicalMessageSource接口，它可以分层次地解析消息。 这些接口共同提供了Spring影响消息解析的基础。 这些接口上定义的方法包括：

* String getMessage（String code，Object \[\] args，String default，Locale loc）：用于从MessageSource检索消息的基本方法。 如果未找到指定区域设置的消息，则使用默认消息。 传入的任何参数都使用标准库提供的MessageFormat功能成为替换值。
* String getMessage（String code，Object \[\] args，Locale loc）：基本上与前一个方法相同，但有一点不同：不能指定默认消息; 如果找不到该消息，则抛出NoSuchMessageException。
* String getMessage（MessageSourceResolvable resolvable，Locale locale）：前面方法中使用的所有属性也包装在名为MessageSourceResolvable的类中，您可以将此方法用于此类。

加载ApplicationContext时，它会自动搜索上下文中定义的MessageSource bean。 bean必须具有名称messageSource。 如果找到这样的bean，则对前面方法的所有调用都被委托给消息源。 如果未找到任何消息源，ApplicationContext将尝试查找包含具有相同名称的bean的父级。 如果是，它将该bean用作MessageSource。 如果ApplicationContext找不到任何消息源，则会实例化一个空的DelegatingMessageSource，以便能够接受对上面定义的方法的调用。

Spring提供了两个MessageSource实现，ResourceBundleMessageSource和StaticMessageSource。 两者都实现HierarchicalMessageSource以执行嵌套消息传递。 StaticMessageSource很少使用，但提供了以编程方式向源添加消息。 ResourceBundleMessageSource显示在以下示例中：

```
<beans>
    <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>format</value>
                <value>exceptions</value>
                <value>windows</value>
            </list>
        </property>
    </bean>
</beans>
```

在该示例中，假设您在类路径中定义了三个资源包，称为格式，异常和窗口。 任何解决消息的请求都将以JDK标准方式处理，通过ResourceBundles解析消息。 出于示例的目的，假设上述两个资源包文件的内容是......

```
# in format.properties
message=Alligators rock!
```

```
# in exceptions.properties
argument.required=The {0} argument is required.
```

下一个示例中显示了执行MessageSource功能的程序。 请记住，所有ApplicationContext实现也都是MessageSource实现，因此可以强制转换为MessageSource接口。

```
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", null);
    System.out.println(message);
}
```

上述程序产生的结果将是......

```
Alligators rock!
```

总而言之，MessageSource是在名为beans.xml的文件中定义的，该文件存在于类路径的根目录中。 messageSource bean定义通过其basenames属性引用许多资源包。 在列表中传递给basenames属性的三个文件作为类路径根目录下的文件存在，分别称为format.properties，exceptions.properties和windows.properties。

下一个示例显示传递给消息查找的参数; 这些参数将转换为字符串并插入到查找消息中的占位符中。

```
<beans>

    <!-- this MessageSource is being used in a web application -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="exceptions"/>
    </bean>

    <!-- lets inject the above MessageSource into this POJO -->
    <bean id="example" class="com.foo.Example">
        <property name="messages" ref="messageSource"/>
    </bean>

</beans>
```

```
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", null);
        System.out.println(message);
    }
}
```

调用execute（）方法得到的结果将是......

```
The userDao argument is required.
```

关于国际化（i18n），Spring的各种MessageSource实现遵循与标准JDK ResourceBundle相同的区域设置解析和回退规则。 简而言之，继续前面定义的示例messageSource，如果要根据British（en-GB）语言环境解析消息，则应分别创建名为format\_en\_GB.properties，exceptions\_en\_GB.properties和windows\_en\_GB.properties的文件。

通常，区域设置解析由应用程序的周围环境管理。 在此示例中，将手动指定将解析（英国）消息的区域设置。

```
# in exceptions_en_GB.properties
argument.required=Ebagum lad, the {0} argument is required, I say, required.
```

```
public static void main(final String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("argument.required",
        new Object [] {"userDao"}, "Required", Locale.UK);
    System.out.println(message);
}
```

运行上述程序产生的结果将是......

```
Ebagum lad, the 'userDao' argument is required, I say, required.
```

您还可以使用MessageSourceAware接口获取对已定义的任何MessageSource的引用。 在创建和配置bean时，应用程序上下文的MessageSource会注入实现MessageSourceAware接口的ApplicationContext中定义的任何bean。

作为ResourceBundleMessageSource的替代，Spring提供了一个ReloadableResourceBundleMessageSource类。 此变体支持相同的bundle文件格式，但比基于标准JDK的ResourceBundleMessageSource实现更灵活。 特别是，它允许从任何Spring资源位置（不仅仅是从类路径）读取文件，并支持bundle属性文件的热重新加载（同时有效地在它们之间缓存它们）。 有关详细信息，请查看ReloadableResourceBundleMessageSource javadocs。

### 7.15.2 标准和自定义活动

ApplicationContext中的事件处理是通过ApplicationEvent类和ApplicationListener接口提供的。 如果将实现ApplicationListener接口的bean部署到上下文中，则每次将ApplicationEvent发布到ApplicationContext时，都会通知该bean。 从本质上讲，这是标准的Observer设计模式。

从Spring 4.2开始，事件基础结构得到了显着改进，并提供了基于注释的模型以及发布任意事件的能力，这是一个不一定从ApplicationEvent扩展的对象。 当这样的对象发布时，我们将它包装在一个事件中。

Spring提供以下标准事件：

**Built-in Events**

| Event |
| :--- |


|  | Explanation |
| :--- | :--- |
| `ContextRefreshedEvent` | Published when the`ApplicationContext`is initialized or refreshed, for example, using the`refresh()`method on the`ConfigurableApplicationContext`interface. "Initialized" here means that all beans are loaded, post-processor beans are detected and activated, singletons are pre-instantiated, and the`ApplicationContext`object is ready for use. As long as the context has not been closed, a refresh can be triggered multiple times, provided that the chosen`ApplicationContext`actually supports such "hot" refreshes. For example,`XmlWebApplicationContext`supports hot refreshes, but`GenericApplicationContext`does not. |
| `ContextStartedEvent` | Published when the`ApplicationContext`is started, using the`start()`method on the`ConfigurableApplicationContext`interface. "Started" here means that all`Lifecycle`beans receive an explicit start signal. Typically this signal is used to restart beans after an explicit stop, but it may also be used to start components that have not been configured for autostart , for example, components that have not already started on initialization. |
| `ContextStoppedEvent` | Published when the`ApplicationContext`is stopped, using the`stop()`method on the`ConfigurableApplicationContext`interface. "Stopped" here means that all`Lifecycle`beans receive an explicit stop signal. A stopped context may be restarted through a`start()`call. |
| `ContextClosedEvent` | Published when the`ApplicationContext`is closed, using the`close()`method on the`ConfigurableApplicationContext`interface. "Closed" here means that all singleton beans are destroyed. A closed context reaches its end of life; it cannot be refreshed or restarted. |
| `RequestHandledEvent` | A web-specific event telling all beans that an HTTP request has been serviced. This event is published\_after\_the request is complete. This event is only applicable to web applications using Spring’s`DispatcherServlet`. |

您还可以创建和发布自己的自定义事件。 这个例子演示了一个扩展Spring的ApplicationEvent基类的简单类：

```
public class BlackListEvent extends ApplicationEvent {

    private final String address;
    private final String content;

    public BlackListEvent(Object source, String address, String content) {
        super(source);
        this.address = address;
        this.content = content;
    }

    // accessor and other methods...
}
```

要发布自定义ApplicationEvent，请在ApplicationEventPublisher上调用publishEvent（）方法。 通常，这是通过创建一个实现ApplicationEventPublisherAware并将其注册为Spring bean的类来完成的。 以下示例演示了这样一个类：

```
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blackList;
    private ApplicationEventPublisher publisher;

    public void setBlackList(List<String> blackList) {
        this.blackList = blackList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String content) {
        if (blackList.contains(address)) {
            publisher.publishEvent(new BlackListEvent(this, address, content));
            return;
        }
        // send email...
    }
}
```

在配置时，Spring容器将检测到EmailService实现ApplicationEventPublisherAware并将自动调用setApplicationEventPublisher（）。 实际上，传入的参数将是Spring容器本身; 您只需通过其ApplicationEventPublisher接口与应用程序上下文进行交互。

要接收自定义ApplicationEvent，请创建一个实现ApplicationListener的类并将其注册为Spring bean。 以下示例演示了这样一个类：

```
public class BlackListNotifier implements ApplicationListener<BlackListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    public void onApplicationEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

请注意，ApplicationListener通常使用自定义事件BlackListEvent的类型进行参数化。 这意味着onApplicationEvent（）方法可以保持类型安全，从而避免任何向下转换的需要。 您可以根据需要注册任意数量的事件侦听器，但请注意，默认情况下，事件侦听器会同步接收事件。 这意味着publishEvent（）方法将阻塞，直到所有侦听器都已完成对事件的处理。 这种同步和单线程方法的一个优点是，当侦听器接收到事件时，如果事务上下文可用，它将在发布者的事务上下文内运行。 如果需要另一个事件发布策略，请参阅Spring的ApplicationEventMulticaster接口的javadoc。

以下示例显示了用于注册和配置上述每个类的bean定义：

```
<bean id="emailService" class="example.EmailService">
    <property name="blackList">
        <list>
            <value>known.spammer@example.org</value>
            <value>known.hacker@example.org</value>
            <value>john.doe@example.org</value>
        </list>
    </property>
</bean>

<bean id="blackListNotifier" class="example.BlackListNotifier">
    <property name="notificationAddress" value="blacklist@example.org"/>
</bean>
```

总而言之，当调用emailService bean的sendEmail（）方法时，如果有任何应该被列入黑名单的电子邮件，则会发布BlackListEvent类型的自定义事件。 blackListNotifier bean被注册为ApplicationListener，因此接收BlackListEvent，此时它可以通知相关方。

_Spring的事件机制是为在同一应用程序上下文中的Spring bean之间的简单通信而设计的。 但是，对于更复杂的企业集成需求，单独维护的Spring Integration项目为构建基于众所周知的Spring编程模型的轻量级，面向模式，事件驱动的体系结构提供了完整的支持。_

#### 基于注释的事件侦听器

从Spring 4.2开始，可以通过EventListener注释在托管bean的任何公共方法上注册事件侦听器。 BlackListNotifier可以重写如下：

```
public class BlackListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlackListEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

如您所见，方法签名再次声明它侦听的事件类型，但这次使用灵活的名称并且没有实现特定的侦听器接口。 只要实际事件类型在其实现层次结构中解析通用参数，也可以通过泛型缩小事件类型。

如果您的方法应该监听多个事件，或者您想要根据任何参数进行定义，那么也可以在注释本身上指定事件类型：

```
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    ...
}
```

还可以通过注释的condition属性添加额外的运行时过滤，该注释定义应该匹配的SpEL表达式以实际调用特定事件的方法。

例如，如果事件的content属性等于foo，则可以重写我们的通知程序以仅调用：

```
@EventListener(condition = "#blEvent.content == 'foo'")
public void processBlackListEvent(BlackListEvent blEvent) {
    // notify appropriate parties via notificationAddress...
}
```

每个SpEL表达式都针对专用上下文进行评估。 下一个表列出了可用于上下文的项目，因此可以将它们用于条件事件处理：

事件SpEL可用元数据

| Name |
| :--- |


|  | Location | Description | Example |
| :--- | :--- | :--- | :--- |
| Event | root object | The actual`ApplicationEvent` | `#root.event` |
| Arguments array | root object | The arguments \(as array\) used for invoking the target | `#root.args[0]` |
| _Argument name_ | evaluation context | Name of any of the method arguments. If for some reason the names are not available \(e.g. no debug information\), the argument names are also available under the`#a<#arg>`where\_\#arg\_stands for the argument index \(starting from 0\). | `#blEvent`or`#a0`\(one can also use`#p0`or`#p<#arg>`notation as an alias\). |

请注意，＃root.event允许您访问基础事件，即使您的方法签名实际上引用了已发布的任意对象。

如果您需要发布一个事件作为处理另一个事件的结果，只需更改方法签名以返回应该发布的事件，例如：

```
@EventListener
public ListUpdateEvent handleBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress and
    // then publish a ListUpdateEvent...
}
```

异步侦听器不支持此功能。

这个新方法将为上面方法处理的每个BlackListEvent发布一个新的ListUpdateEvent。 如果您需要发布多个事件，请返回一个事件集合。

#### 异步监听器

如果您希望特定侦听器异步处理事件，只需重用常规@Async支持：

```
@EventListener
@Async
public void processBlackListEvent(BlackListEvent event) {
    // BlackListEvent is processed in a separate thread
}
```

使用异步事件时请注意以下限制：

1. 如果事件侦听器抛出异常，它将不会传播给调用者，请检查AsyncUncaughtExceptionHandler以获取更多详细信息。
2. 此类事件监听器无法发送回复。 如果您需要作为处理结果发送另一个事件，请注入ApplicationEventPublisher以手动发送事件。

#### Ordering listeners

如果需要在另一个之前调用侦听器，只需将@Order注释添加到方法声明中：

```
@EventListener
@Order(42)
public void processBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress...
}
```

#### 通用事件\(Generic events\)

您还可以使用泛型来进一步定义事件的结构。 考虑一个EntityCreatedEvent &lt;T&gt;，其中T是创建的实际实体的类型。 您可以创建以下侦听器定义以仅接收Person的EntityCreatedEvent：

```
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    ...
}
```

由于类型擦除，这只有在被触发的事件解析事件侦听器在其上过滤的泛型参数时才会起作用（类似于

class PersonCreatedEvent extends EntityCreatedEvent&lt;Person&gt; { …​ }）

在某些情况下，如果所有事件都遵循相同的结构（这应该是上述事件的情况），这可能会变得相当繁琐。 在这种情况下，您可以实现ResolvableTypeProvider来指导框架超出运行时环境提供的范围：

```
public class EntityCreatedEvent<T> extends ApplicationEvent implements ResolvableTypeProvider {

    public EntityCreatedEvent(T entity) {
        super(entity);
    }

    @Override
    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(getClass(),
                ResolvableType.forInstance(getSource()));
    }
}
```

这不仅适用于ApplicationEvent，也适用于您作为事件发送的任意对象。

### 7.15.3 方便地访问low-level资源

为了最佳地使用和理解应用程序上下文，用户通常应该熟悉Spring的资源抽象，如第8章“资源”一章所述。

应用程序上下文是ResourceLoader，可用于加载资源。 Resource本质上是JDK类java.net.URL的功能更丰富的版本，实际上，Resource的实现包装了适当的java.net.URL实例。 资源可以透明的方式从几乎任何位置获取低级资源，包括从类路径，文件系统位置，任何可用标准URL描述的位置，以及一些其他变体。 如果资源位置字符串是没有任何特殊前缀的简单路径，那么这些资源来自特定且适合于实际应用程序上下文类型。

您可以配置部署到应用程序上下文中的bean，以实现特殊的回调接口ResourceLoaderAware，在初始化时自动回调，应用程序上下文本身作为ResourceLoader传入。 您还可以公开Resource类型的属性，以用于访问静态资源; 它们将像任何其他属性一样注入其中。 您可以将这些Resource属性指定为简单的String路径，并依赖于上下文自动注册的特殊JavaBean PropertyEditor，以便在部署Bean时将这些文本字符串转换为实际的Resource对象。

提供给ApplicationContext构造函数的位置路径实际上是资源字符串，并且以简单的形式适当地处理特定的上下文实现。 ClassPathXmlApplicationContext将简单的位置路径视为类路径位置。 您还可以使用具有特殊前缀的位置路径（资源字符串）来强制从类路径或URL加载定义，而不管实际的上下文类型如何。

### 7.15.4 方便的Web应用程序的ApplicationContext实例化

您可以使用例如ContextLoader以声明方式创建ApplicationContext实例。 当然，您也可以使用其中一个ApplicationContext实现以编程方式创建ApplicationContext实例。

您可以使用ContextLoaderListener注册ApplicationContext，如下所示：

```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

listener检查contextConfigLocation参数。 如果参数不存在，则侦听器将/WEB-INF/applicationContext.xml用作默认值。 当参数确实存在时，侦听器使用预定义的分隔符（逗号，分号和空格）分隔String，并将值用作将搜索应用程序上下文的位置。 还支持Ant样式的路径模式。 名称以“Context.xml”结尾的所有文件的/WEB-INF/\*Context.xml，驻留在“WEB-INF”目录中，/ WEB-INF/\*\*/\*Context.xml表示所有文件 这些文件位于“WEB-INF”的任何子目录中。

### 7.15.5 将Spring ApplicationContext部署为Java EE RAR文件

可以将Spring ApplicationContext部署为RAR文件，将上下文及其所有必需的bean类和库JAR封装在Java EE RAR部署单元中。 这相当于引导一个独立的ApplicationContext，它只是托管在Java EE环境中，能够访问Java EE服务器设施。 RAR部署是部署无头WAR文件的场景的更自然的替代方案，实际上是没有任何HTTP入口点的WAR文件，仅用于在Java EE环境中引导Spring ApplicationContext。

RAR部署非常适用于不需要HTTP入口点但仅包含消息端点和预定作业的应用程序上下文。 在这样的上下文中的Bean可以使用应用程序服务器资源，例如JTA事务管理器和JNDI绑定的JDBC DataSources和JMS ConnectionFactory实例，也可以通过Spring的标准事务管理以及JNDI和JMX支持工具向平台的JMX服务器注册。 应用程序组件还可以通过Spring的TaskExecutor抽象与应用程序服务器的JCA WorkManager进行交互。

查看SpringContextResourceAdapter类的javadoc，了解RAR部署中涉及的配置详细信息。

对于将Spring ApplicationContext简单部署为Java EE RAR文件：将所有应用程序类打包到RAR文件中，该文件是具有不同文件扩展名的标准JAR文件。 将所有必需的库JAR添加到RAR存档的根目录中。 添加“META-INF / ra.xml”部署描述符（如SpringContextResourceAdapters javadoc中所示）和相应的Spring XML bean定义文件（通常为“META-INF / applicationContext.xml”），并删除生成的RAR文件 进入应用程序服务器的部署目录。

_**这种RAR部署单元通常是独立的; 它们不会将组件暴露给外部世界，甚至不会暴露给同一应用程序的其他模块。 与基于RAR的ApplicationContext的交互通常通过与其他模块共享的JMS目标进行。 例如，基于RAR的ApplicationContext还可以调度一些作业，对文件系统中的新文件（或类似物）作出反应。 如果它需要允许来自外部的同步访问，它可以例如导出RMI端点，当然可以由同一机器上的其他应用程序模块使用。**_




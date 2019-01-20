## 11.8 在Spring应用程序中使用AspectJ

到目前为止，我们在本章中介绍的所有内容都是纯粹的Spring AOP。在本节中，我们将讨论如何使用AspectJ编译器/编织器代替Spring AOP，或者除了Spring AOP之外，如果您的需求超出Spring AOP提供的功能。

Spring附带了一个小的AspectJ方面库，它可以作为spring-aspects.jar在您的发行版中独立使用;您需要将其添加到类路径中才能使用其中的方面。第11.8.1节“使用AspectJ依赖于使用Spring注入域对象”和第11.8.2节“AspectJ的其他Spring方面”讨论了该库的内容以及如何使用它。第11.8.3节“使用Spring IoC配置AspectJ方面”讨论了如何依赖注入使用AspectJ编译器编织的AspectJ方面。最后，第11.8.4节“在Spring框架中使用AspectJ进行加载时编织”介绍了使用AspectJ为Spring应用程序加载时编织。

### 11.8.1 使用AspectJ依赖注入域对象与Spring

Spring容器实例化和配置在应用程序上下文中定义的bean。 在给定包含要应用的配置的bean定义的名称的情况下，还可以要求bean工厂配置预先存在的对象。 spring-aspects.jar包含一个注释驱动的方面，利用此功能允许依赖注入任何对象。 该支持旨在用于在任何容器控制之外创建的对象。 域对象通常属于此类别，因为它们通常使用new运算符以编程方式创建，或者由于数据库查询而由ORM工具创建。

@Configurable注释标记一个类符合Spring驱动配置的条件。 在最简单的情况下，它可以用作标记注释：

```
package com.xyz.myapp.domain;

import org.springframework.beans.factory.annotation.Configurable;

@Configurable
public class Account {
    // ...
}
```

当以这种方式用作标记接口时，Spring将使用与完全限定类型名称相同名称的bean定义（通常为prototype-scoped）来配置带注释类型的新实例（在本例中为Account）（com.xyz）.myapp.domain.Account）。 由于bean的默认名称是其类型的完全限定名称，因此声明原型定义的简便方法是省略id属性：

```
<bean class="com.xyz.myapp.domain.Account" scope="prototype">
    <property name="fundsTransferService" ref="fundsTransferService"/>
</bean>
```

如果要显式指定要使用的原型bean定义的名称，可以直接在注释中执行此操作：

```
package com.xyz.myapp.domain;

import org.springframework.beans.factory.annotation.Configurable;

@Configurable("account")
public class Account {
    // ...
}
```

Spring现在将查找名为“account”的bean定义，并将其用作配置新Account实例的定义。

您还可以使用自动装配来避免必须指定专用的bean定义。要让Spring应用自动装配使用@Configurable批注的autowire属性：指定@Configurable（autowire = Autowire.BY\_TYPE）或@Configurable（autowire = Autowire.BY\_NAME分别按类型或名称进行自动装配。作为替代， Spring 2.5最好在字段或方法级别使用@Autowired或@Inject为@Configurable bean指定显式的，注释驱动的依赖注入（更多详细信息，请参见第7.9节“基于注释的容器配置”）。

最后，您可以使用dependencyCheck属性为新创建和配置的对象中的对象引用启用Spring依赖项检查（例如：@Configurable（autowire = Autowire.BY\_NAME，dependencyCheck = true））。如果此属性设置为true，则Spring将在配置后验证是否已设置所有属性（不是基元或集合）。

单独使用注释当然不会做任何事情。 Spring-aspects.jar中的AnnotationBeanConfigurerAspect作用于注释的存在。本质上，方面说“在从使用@Configurable注释的类型的新对象的初始化返回之后，根据注释的属性使用Spring配置新创建的对象”。在此上下文中，初始化是指新实例化的对象（例如，用新运算符实例化的对象）以及正在进行反序列化的可序列化对象（例如，通过readResolve（））。

上段中的一个关键短语是“实质上”。 对于大多数情况，'从新对象初始化返回后'的确切语义将很好......在此上下文中，'初始化'意味着在构造对象后将注入依赖项 - 这意味着 依赖项将无法在类的构造函数体中使用。 如果您希望在构造函数体执行之前注入依赖项，从而可以在构造函数体中使用，那么您需要在@Configurable声明中定义它，如下所示：

```
@Configurable(preConstruction=true)
```

您可以在AspectJ编程指南的本附录中找到有关AspectJ中各种切入点类型的语言语义的更多信息。

为此，必须使用AspectJ编织器编写带注释的类型 - 您可以使用构建时Ant或Maven任务来执行此操作（请参阅AspectJ开发环境指南）或加载时编织（请参阅第11.8节）。 4，“在Spring框架中使用AspectJ进行加载时编织”）。 AnnotationBeanConfigurerAspect本身需要通过Spring进行配置（以获取对用于配置新对象的bean工厂的引用）。 如果您使用的是基于Java的配置，只需将@EnableSpringConfigured添加到任何@Configuration类。

```
@Configuration
@EnableSpringConfigured
public class AppConfig {

}
```

如果您更喜欢基于XML的配置，则Spring上下文命名空间定义了一个方便的上下文：spring-configured元素：

```
<context:spring-configured/>
```

在配置方面之前创建的@Configurable对象的实例将导致向调试日志发出消息，并且不会发生对象的配置。 一个示例可能是Spring配置中的bean，它在Spring初始化时创建域对象。 在这种情况下，您可以使用“depends-on”bean属性手动指定bean依赖于配置方面。

```
<bean id="myService"
        class="com.xzy.myapp.service.MyService"
        depends-on="org.springframework.beans.factory.aspectj.AnnotationBeanConfigurerAspect">

    <!-- ... -->

</bean>
```

不要通过bean配置器方面激活@Configurable处理，除非你真的想在运行时依赖它的语义。 特别是，确保不对使用容器注册为常规Spring bean的bean类使用@Configurable：否则将进行双初始化，一次通过容器，一次通过方面。

#### 单元测试@Configurable对象

@Configurable支持的目标之一是启用域对象的独立单元测试，而不会遇到与硬编码查找相关的困难。 如果AspectJ没有编译@Configurable类型，那么注释在单元测试期间没有任何影响，您可以简单地在被测对象中设置模拟或存根属性引用并继续正常进行。 如果AspectJ编译了@Configurable类型，那么您仍然可以正常地在容器外部进行单元测试，但每次构造一个@Configurable对象时，您将看到一条警告消息，表明它尚未由Spring配置。

#### 使用多个应用程序上下文

用于实现@Configurable支持的AnnotationBeanConfigurerAspect是AspectJ单例方面。单例方面的范围与静态成员的范围相同，也就是说每个类加载器有一个方面实例来定义类型。这意味着如果在同一个类加载器层次结构中定义多个应用程序上下文，则需要考虑在哪里定义@EnableSpringConfigured bean以及将spring-aspects.jar放在类路径上的位置。

考虑一个典型的Spring Web应用程序配置，其中包含共享父应用程序上下文，用于定义公共业务服务和支持它们所需的一切，以及每个servlet包含一个子应用程序上下文，其中包含特定于该servlet所有这些上下文将共存于同一个类加载器层次结构中，因此AnnotationBeanConfigurerAspect只能保存对其中一个的引用。在这种情况下，我们建议在共享（父）应用程序上下文中定义@EnableSpringConfigured bean：这将定义您可能要注入域对象的服务。结果是您无法使用@Configurable机制（可能不是您想要做的事情！）来配置域对象，并引用在子（特定于servlet）的上下文中定义的bean。

在同一容器中部署多个Web应用程序时，请确保每个Web应用程序使用自己的类加载器加载spring-aspects.jar中的类型（例如，将spring-aspects.jar放在“WEB-INF / lib”中） 。如果spring-aspects.jar仅添加到容器范围的类路径中（因此由共享父类加载器加载），则所有Web应用程序将共享相同的方面实例，这可能不是您想要的。

### 11.8.2  AspectJ的其他Spring方面

除了@Configurable方面，spring-aspects.jar还包含一个AspectJ方面，可用于为使用@Transactional注释注释的类型和方法驱动Spring的事务管理。这主要适用于希望在Spring容器之外使用Spring Framework的事务支持的用户。

解释@Transactional注释的方面是AnnotationTransactionAspect。使用此方面时，必须注释实现类（和/或该类中的方法），而不是该类实现的接口（如果有）。 AspectJ遵循Java的规则，即接口上的注释不会被继承。

类上的@Transactional注释指定了在类中执行任何公共操作的默认事务语义。

类中方法的@Transactional注释会覆盖类注释（如果存在）给出的默认事务语义。可以注释任何可见性的方法，包括私有方法。直接注释非公共方法是获得执行此类方法的事务划分的唯一方法。

从Spring Framework 4.2开始，spring-aspects提供了类似的方面，为标准的javax.transaction.Transactional注释提供了完全相同的功能。 检查JtaAnnotationTransactionAspect以获取更多详细信息。

对于想要使用Spring配置和事务管理支持但不想（或不能）使用注释的AspectJ程序员，spring-aspects.jar还包含可以扩展以提供自己的切入点定义的抽象方面。 有关更多信息，请参阅AbstractBeanConfigurerAspect和AbstractTransactionAspect方面的源代码。 作为示例，以下摘录显示了如何使用与完全限定类名匹配的原型bean定义编写方面来配置域模型中定义的所有对象实例：

```
public aspect DomainObjectConfiguration extends AbstractBeanConfigurerAspect {

    public DomainObjectConfiguration() {
        setBeanWiringInfoResolver(new ClassNameBeanWiringInfoResolver());
    }

    // the creation of a new bean (any object in the domain model)
    protected pointcut beanCreation(Object beanInstance) :
        initialization(new(..)) &&
        SystemArchitecture.inDomainModel() &&
        this(beanInstance);

}
```

### 11.8.3 使用Spring IoC配置AspectJ aspects

在Spring应用程序中使用AspectJ方面时，很自然希望能够使用Spring配置这些方面。 AspectJ运行时本身负责方面创建，并且通过Spring配置AspectJ创建方面的方法取决于方面使用的AspectJ实例化模型（per-xxx子句）。

AspectJ的大多数方面都是单例方面。 这些方面的配置非常简单：只需创建一个正常引用方面类型的bean定义，并包含bean属性'factory-method =“aspectOf”'。 这可以确保Spring通过向AspectJ请求它来获取方面实例，而不是尝试创建实例本身。 例如：

```
<bean id="profiler" class="com.xyz.profiler.Profiler"
        factory-method="aspectOf">

    <property name="profilingStrategy" ref="jamonProfilingStrategy"/>
</bean>
```

非单例方面更难配置：但是可以通过创建原型bean定义并使用spring-aspects.jar中的@Configurable支持来配置方面实例（一旦它们具有由AspectJ运行时创建的bean）。

如果您想要使用AspectJ编写一些@AspectJ方面（例如，使用域模型类型的加载时编织）和其他要与Spring AOP一起使用的@AspectJ方面，并且这些方面都是使用Spring配置的 ，那么你需要告诉Spring AOP @AspectJ autoproxying支持配置中定义的@AspectJ方面的确切子集应该用于自动代理。 您可以通过在&lt;aop：aspectj-autoproxy /&gt;声明中使用一个或多个&lt;include /&gt;元素来完成此操作。 每个&lt;include /&gt;元素指定一个名称模式，只有名称与至少一个模式匹配的bean才会用于Spring AOP autoproxy配置：

```
<aop:aspectj-autoproxy>
    <aop:include name="thisBean"/>
    <aop:include name="thatBean"/>
</aop:aspectj-autoproxy>
```

不要被&lt;aop：aspectj-autoproxy /&gt;元素的名称误导：使用它将导致创建Spring AOP代理。 这里只使用@AspectJ样式的方面声明，但不涉及AspectJ运行时。

### 11.8.4 在Spring框架中使用AspectJ进行加载时编织

#### 第一个例子

#### 方面

#### 'META-INF / aop.xml文件'

#### 必需的库（JARS）

#### 弹簧配置

#### 特定于环境的配置




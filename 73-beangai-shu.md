## 7.3 Bean概述

Spring IoC容器管理一个或多个bean。 这些bean是使用您提供给容器的配置元数据创建的，例如，以XML &lt;bean/&gt;定义的形式。

在容器本身内，这些bean定义表示为BeanDefinition对象，其中包含（以及其他信息）以下元数据：

* 包限定的类名：通常是正在定义的bean的实际实现类。
* Bean行为配置元素，说明bean在容器中的行为方式（范围，生命周期回调等）。
* 引用bean执行其工作所需的其他bean; 这些引用也称为协作者或依赖项。
* 要在新创建的对象中设置的其他配置设置，例如，在管理连接池的Bean中使用的连接数或池的大小限制。

此元数据转换为组成每个bean定义的一组属性。

| Property | Explained in…​ |
| :--- | :--- |
| class | [Section 7.3.2, “Instantiating beans”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-class) |
| name | [Section 7.3.1, “Naming beans”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-beanname) |
| scope | [Section 7.5, “Bean scopes”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-scopes) |
| constructor arguments | [Section 7.4.1, “Dependency Injection”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-collaborators) |
| properties | [Section 7.4.1, “Dependency Injection”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-collaborators) |
| autowiring mode | [Section 7.4.5, “Autowiring collaborators”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-autowire) |
| lazy-initialization mode | [Section 7.4.4, “Lazy-initialized beans”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-lazy-init) |
| initialization method | [the section called “Initialization callbacks”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-lifecycle-initializingbean) |
| destruction method | [the section called “Destruction callbacks”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-lifecycle-disposablebean) |

除了包含有关如何创建特定bean的信息的bean定义之外，ApplicationContext实现还允许用户注册在容器外部创建的现有对象。 这是通过getBeanFactory\(\)方法访问ApplicationContext的BeanFactory来完成的，该方法返回BeanFactory实现DefaultListableBeanFactory。 DefaultListableBeanFactory通过方法registerSingleton（..）和registerBeanDefinition（..）支持此注册。 但是，典型应用程序仅适用于通过元数据bean定义定义的bean。

_需要尽早注册Bean元数据和手动提供的单例实例，以便容器在自动装配和其他内省步骤期间正确推理它们。 虽然在某种程度上支持覆盖现有元数据和现有单例实例，但是在运行时注册新bean（与对工厂的实时访问同时）并未得到官方支持，并且可能导致bean容器中的并发访问异常和/或不一致状态。_

### 7.3.1命名bean

每个bean都有一个或多个标识符。这些标识符在托管bean的容器中必须是唯一的。bean通常只有一个标识符，但如果它需要多个标识符，则额外的标识符可以被视为别名。

在基于XML的配置元数据中，使用id和/或name属性指定bean标识符。该id属性允许您指定一个id。通常，这些名称是字母数字（'myBean'，'fooService'等），但也可能包含特殊字符。如果要向bean引入其他别名，还可以在name 属性中指定它们，用逗号（,），分号（;）或空格分隔。作为历史记录，在Spring 3.1之前的版本中，该id属性被定义为一种xsd:ID类型，它约束了可能的字符。从3.1开始，它被定义为一种xsd:string类型。请注意，id容器仍然强制执行bean 唯一性，但不再是XML解析器。

您不需要为bean提供名称或ID。如果没有显式提供名称或标识，则容器会为该bean生成唯一的名称。但是，如果要通过名称引用该bean，则必须通过使用ref元素或 Service Locator样式查找来提供名称。不提供名称的动机与使用内部bean和自动装配协作者有关。

_Bean命名约定_

_惯例是在命名bean时使用标准Java约定作为实例字段名称。也就是说，bean名称以小写字母开头，从那时起就是驼峰式的。这种名称的例子将是（不带引号）'accountManager'， 'accountService'，'userDao'，'loginController'，等等。_

_命名bean始终使您的配置更易于阅读和理解，如果您使用的是Spring AOP，那么在将建议应用于与名称相关的一组bean时，它会有很大帮助。_

### 7.3.2实例化bean

bean定义本质上是用于创建一个或多个对象的配方。容器在被询问时查看命名bean的配方，并使用由该bean定义封装的配置元数据来创建（或获取）实际对象。

如果使用基于XML的配置元数据，则指定要在元素的class属性中实例化的对象的类型（或类）&lt;bean/&gt;。此 class属性在内部是 实例Class上的属性BeanDefinition，通常是必需的。（有关异常，请参阅 “使用实例工厂方法实例化”一节和第7.7节“Bean定义继承”。）您可以通过Class以下两种方式之一使用该属性：

#### 使用构造函数实例化

当您通过构造方法创建bean时，所有普通类都可以使用并与Spring兼容。也就是说，正在开发的类不需要实现任何特定接口或以特定方式编码。简单地指定bean类就足够了。但是，根据您为该特定bean使用的IoC类型，您可能需要一个默认（空）构造函数。

Spring IoC容器几乎可以管理您希望它管理的任何类;它不仅限于管理真正的JavaBeans。大多数Spring用户更喜欢实际的JavaBeans，只有一个默认（无参数）构造函数，并且在容器中的属性之后建模了适当的setter和getter。您还可以在容器中拥有更多异国情调的非bean样式类。例如，如果您需要使用绝对不符合JavaBean规范的旧连接池，那么Spring也可以对其进行管理。

使用基于XML的配置元数据，您可以按如下方式指定bean类：

```
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

有关为构造函数提供参数的机制（如果需要）以及在构造对象后设置对象实例属性的详细信息，请参阅注入依赖项。

#### 使用静态工厂方法实例化

定义使用静态工厂方法创建的bean时，可以使用class属性指定包含静态工厂方法的类和名为factory-method的属性，以指定工厂方法本身的名称。 您应该能够调用此方法（使用后面描述的可选参数）并返回一个活动对象，随后将其视为通过构造函数创建的对象。 这种bean定义的一个用途是在遗留代码中调用静态工厂。

以下bean定义指定通过调用factory-method创建bean。 该定义未指定返回对象的类型（类），仅指定包含工厂方法的类。 在此示例中，createInstance（）方法必须是静态方法。

```
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

```
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

有关在从工厂返回对象后向工厂方法提供（可选）参数并设置对象实例属性的机制的详细信息，请参阅详细信息中的依赖关系和配置。

#### 使用实例工厂方法实例化

与通过静态工厂方法实例化类似，使用实例工厂方法进行实例化会从容器调用现有bean的非静态方法来创建新bean。 要使用此机制，请将class属性保留为空，并在factory-bean属性中，在当前（或父/祖先）容器中指定bean的名称，该容器包含要调用以创建对象的实例方法。 使用factory-method属性设置工厂方法本身的名称。

```
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

```
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以包含多个工厂方法，如下所示：

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
    }
}
```

这种方法表明可以通过依赖注入（DI）来管理和配置工厂bean本身。 请参阅详细信息中的依赖关系和配置。

在Spring文档中，工厂bean指的是在Spring容器中配置的bean，它将通过实例或静态工厂方法创建对象。 相比之下，FactoryBean（注意大小写）是指Spring特定的FactoryBean。



## 




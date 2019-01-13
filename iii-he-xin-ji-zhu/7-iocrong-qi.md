# 7 Ioc容器

## 7.1 Spring IoC容器和bean简介

本章介绍了Spring Framework实现的控制反转（IoC）原理。 IoC也称为依赖注入（DI）。 这是一个过程，通过这个过程，对象定义它们的依赖关系，即它们使用的其他对象，只能通过构造函数参数，工厂方法的参数，或者在构造或从工厂方法返回后在对象实例上设置的属性。 然后容器在创建bean时注入这些依赖项。 这个过程基本上是反向的，因此名称Inversion of Control（IoC），bean本身通过使用类的直接构造或诸如Service Locator模式之类的机制来控制其依赖关系的实例化或位置。

org.springframework.beans和org.springframework.context包是Spring Framework的IoC容器的基础。 BeanFactory接口提供了一种能够管理任何类型对象的高级配置机制。 ApplicationContext是BeanFactory的子接口。 它增加了与Spring的AOP功能的更容易的集成; 消息资源处理（用于国际化），事件发布; 和特定于应用程序层的上下文，例如WebApplicationContext，用于Web应用程序。

简而言之，BeanFactory提供配置框架和基本功能，ApplicationContext添加了更多特定于企业的功能。 ApplicationContext是BeanFactory的完整超集，在本章中专门用于Spring的IoC容器的描述。 有关使用BeanFactory而不是ApplicationContext的更多信息，请参见第7.16节“BeanFactory”。

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。 bean是一个由Spring IoC容器实例化，组装和管理的对象。 否则，bean只是应用程序中众多对象之一。 Bean及其之间的依赖关系反映在容器使用的配置元数据中。

## 

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

#### 使用静态工厂方法实例化

#### 使用实例工厂方法实例化

## 7.4依赖性

典型的企业应用程序不包含单个对象（或Spring用法中的bean）。 即使是最简单的应用程序也有一些对象可以协同工作，以呈现最终用户所看到的连贯应用程序。 下一节将介绍如何定义多个独立的bean定义，以及对象协作实现目标的完全实现的应用程序。

### 7.4.1 依赖注入

依赖注入（DI）是一个过程，通过这个过程，对象定义它们的依赖关系，即它们使用的其他对象，只能通过构造函数参数，工厂方法的参数或在构造或返回对象实例后在对象实例上设置的属性。从工厂方法。然后容器在创建bean时注入这些依赖项。这个过程基本上是反向的，因此名称Inversion of Control（IoC），bean本身通过使用类的直接构造或服务定位器模式来控制其依赖项的实例化或位置。

使用DI原理的代码更清晰，当对象提供其依赖项时，解耦更有效。该对象不查找其依赖项，也不知道依赖项的位置或类。因此，您的类变得更容易测试，特别是当依赖关系在接口或抽象基类上时，这允许在单元测试中使用存根或模拟实现。

DI存在两个主要变体，基于构造函数的依赖注入和基于Setter的依赖注入。

#### 基于构造函数的依赖注入

基于构造函数的DI由容器调用具有多个参数的构造函数来完成，每个参数表示一个依赖项。 调用具有特定参数的static工厂方法来构造bean几乎是等效的，本讨论同样处理构造函数和static工厂方法的参数。 以下示例显示了一个只能通过构造函数注入进行依赖注入的类。 请注意，此类没有什么特别之处，它是一个POJO，它不依赖于容器特定的接口，基类或注释。

```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

#### 基于Setter的依赖注入

使用参数的类型进行构造函数参数解析匹配。 如果bean定义的构造函数参数中不存在潜在的歧义，那么在bean定义中定义构造函数参数的顺序是在实例化bean时将这些参数提供给适当的构造函数的顺序。 考虑以下class：

```
package x.y;

public class Foo {

    public Foo(Bar bar, Baz baz) {
        // ...
    }
}
```

假设Bar和Baz类与继承无关，则不存在潜在的歧义。 因此，以下配置工作正常，您无需在&lt;constructor-arg/&gt;元素中显式指定构造函数参数索引和/或类型。

```
<beans>
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
    </bean>

    <bean id="bar" class="x.y.Bar"/>

    <bean id="baz" class="x.y.Baz"/>
</beans>
```

当引用另一个bean时，类型是已知的，并且可以进行匹配（与前面的示例一样）。 当使用简单类型时，例如&lt;value&gt; true &lt;/ value&gt;，Spring无法确定值的类型，因此无法在没有帮助的情况下按类型进行匹配。 考虑以下class

#### 

#### 依赖性解决过程

#### 依赖注入的例子

### 7.4.2 依赖关系和配置详细

如上一节所述，您可以将bean属性和构造函数参数定义为对其他托管bean（协作者）的引用，或者作为内联定义的值。 Spring的基于XML的配置元数据为此目的支持其&lt;property /&gt;和&lt;constructor-arg /&gt;元素中的子元素类型。

#### 直值（基本类型，字符串等）

&lt;property /&gt;元素的value属性将属性或构造函数参数指定为人类可读的字符串表示形式。 Spring的转换服务用于将这些值从String转换为属性或参数的实际类型。

#### 引用其他bean（协作者）

#### Inner beans

&lt;property /&gt;或&lt;constructor-arg /&gt;元素中的&lt;bean /&gt;元素定义了一个所谓的内部bean。

```
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean定义不需要定义的id或名称; 如果指定，则容器不使用这样的值作为标识符。 容器还会在创建时忽略范围标志：内部bean始终是匿名的，并且始终使用外部bean创建它们。 无法将内部bean注入协作bean，而不是将其注入封闭bean或独立访问它们。

作为极端情况，可以从自定义范围接收销毁回调，例如， 对于包含在单例bean中的请求范围的内部bean：内部bean实例的创建将绑定到其包含的bean，但是销毁回调允许它参与请求范围的生命周期。 这不是常见的情况; 内部bean通常只是共享其包含bean的范围。

#### 集合

#### 空值和空字符串值

Spring将属性等的空参数视为空字符串。 以下基于XML的配置元数据片段将email属性设置为空String值（“”）。

```
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

上面的示例等效于以下Java代码：

```
exampleBean.setEmail("");
```

&lt;null /&gt;元素处理空值。 例如：

```
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

以上配置等同于以下Java代码：

```
exampleBean.setEmail(null);
```

#### 带有p命名空间的XML快捷方式

#### 带有c-namespace的XML快捷方式

#### 复合属性名称

### 7.4.3 使用依赖

如果bean是另一个bean的依赖项，通常意味着将一个bean设置为另一个bean的属性。 通常，您可以使用基于XML的配置元数据中的&lt;ref /&gt;元素来完成此操作。 但是，有时bean之间的依赖关系不那么直接; 例如，需要触发类中的静态初始化程序，例如数据库驱动程序注册。 在初始化使用此元素的bean之前，depends-on属性可以**显式强制初始化一个或多个bea**n。 以下示例使用depends-on属性表示对单个bean的依赖关系：

```
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表示对多个bean的依赖关系，请提供bean名称列表作为depends-on属性的值，使用逗号，空格和分号作为有效分隔符：

```
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

---

_bean定义中的depends-on属性既可以指定初始化时间依赖关系，也可以指定仅限单例bean的相应销毁时间依赖关系。 在给定的bean本身被销毁之前，首先销毁定义与给定bean的依赖关系的从属bean。 因此，依赖也可以控制关机顺序。_

---

### 7.4.4 懒惰初始化的bean

默认情况下，ApplicationContext实现会急切地创建和配置所有单例bean，作为初始化过程的一部分。 通常，这种预先实例化是可取的，因为配置或周围环境中的错误是立即发现的，而不是几小时甚至几天后。 如果不希望出现这种情况，可以通过将bean定义标记为延迟初始化来阻止单例bean的预实例化。 延迟初始化的bean告诉IoC容器在第一次请求时创建bean实例，而不是在启动时。

在XML中，此行为由&lt;bean /&gt;元素上的lazy-init属性控制; 例如：

```
<bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.foo.AnotherBean"/>
```

当ApplicationContext使用前面的配置时，在ApplicationContext启动时，不会急切地预先实例化名为lazy的bean，而是急切地预先实例化not.lazy bean。

但是，当延迟初始化的bean是不是延迟初始化的单例bean的依赖项时，ApplicationContext会在启动时创建延迟初始化的bean，因为它必须满足单例的依赖关系。 惰性初始化的bean被注入到其他地方的单例bean中，而不是懒惰初始化的。

您还可以使用&lt;beans /&gt;元素上的default-lazy-init属性在容器级别控制延迟初始化; 例如：

```
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

### 7.4.5 自动装配协作者

Spring容器可以自动连接协作bean之间的关系。 您可以通过检查ApplicationContext的内容，允许Spring自动为您的bean解析协作者（其他bean）。 自动装配具有以下优点：

* 自动装配可以显着减少指定属性或构造函数参数的需要。 （在本章其他地方讨论的其他机制，如bean模板，在这方面也很有价值。）
* 自动装配可以随着对象的发展更新配置。 例如，如果需要向类添加依赖项，则可以自动满足该依赖项，而无需修改配置。 因此，自动装配在开发期间尤其有用，而不会在代码库变得更稳定时否定切换到显式布线的选项。

使用基于XML的配置元数据\[2\]时，可以使用&lt;bean /&gt;元素的autowire属性为bean定义指定autowire模式。 自动装配功能有四种模式。 您指定每个bean的自动装配，因此可以选择要自动装配的那些。

**Autowiring modes**

| 模式 | 说明 |
| :--- | :--- |
| no | （默认）无自动装配。 必须通过ref元素定义Bean引用。 不建议对较大的部署更改默认设置，因为明确指定协作者可以提供更好的控制和清晰度。 在某种程度上，它记录了系统的结构。 |
| byName | 按属性名称自动装配。 Spring查找与需要自动装配的属性同名的bean。 例如，如果bean定义按名称设置为autowire，并且它包含master属性（即，它具有setMaster（..）方法），则Spring会查找名为master的bean定义，并使用它来设置 属性。 |
| byType | 如果容器中只存在一个属性类型的bean，则允许自动装配属性。 如果存在多个，则抛出致命异常，这表示您不能对该bean使用byType自动装配。 如果没有匹配的bean，则没有任何反应; 该物业未设定。 |
| 构造器 | 类似于byType，但适用于构造函数参数。 如果容器中没有构造函数参数类型的一个bean，则会引发致命错误。 |

使用byType或构造函数自动装配模式，您可以连接数组和类型集合。 在这种情况下，提供容器内与预期类型匹配的所有autowire候选者以满足依赖性。 如果预期的键类型为String，则可以自动装配强类型映射。 自动装配的Maps值将包含与预期类型匹配的所有Bean实例，而Maps键将包含相应的bean名称。

您可以将autowire行为与依赖关系检查结合起来，这将在自动装配完成后执行。

#### 自动装配的局限和缺点

自动装配在项目中一致使用时效果最佳。 如果一般不使用自动装配，那么开发人员使用它来连接一个或两个bean定义可能会让人感到困惑。

考虑自动装配的局限和缺点：

* property和constructor-arg设置中的显式依赖项始终覆盖自动装配。 您无法自动装配所谓的简单属性，例如基元，字符串和类（以及此类简单属性的数组）。 这种限制是按设计的。
* 自动装配不如显式布线精确。 虽然如上表所示，Spring会小心避免在可能产生意外结果的歧义的情况下进行猜测，但不再明确记录Spring管理对象之间的关系。
* 可能无法为可能从Spring容器生成文档的工具提供接线信息。
* 容器中的多个bean定义可能与要自动装配的setter方法或构造函数参数指定的类型匹配。 对于数组，集合或地图，这不一定是个问题。 但是，对于期望单个值的依赖关系，这种模糊性不是任意解决的。 如果没有可用的唯一bean定义，则抛出异常。

在后一种情况下，您有几种选择：

* 放弃自动装配以支持显式布线。
* 通过将autowire-candidate属性设置为false，避免对bean定义进行自动装配，如下一节所述。
* 通过将其&lt;bean /&gt;元素的primary属性设置为true，将单个bean定义指定为主要候选者。
* 使用基于注释的配置实现更细粒度的控件，如第7.9节“基于注释的容器配置”中所述。

#### 从自动装配中排除一个bean

在每个bean的基础上，您可以从自动装配中排除bean。 在Spring的XML格式中，将&lt;bean /&gt;元素的autowire-candidate属性设置为false; 容器使特定的bean定义对自动装配基础结构不可用（包括注释样式配置，如@Autowired）。

_autowire-candidate属性旨在仅影响基于类型的自动装配。 它不会影响名称的显式引用，即使指定的bean未标记为autowire候选，也会解析它。 因此，如果名称匹配，按名称自动装配将注入bean。_

您还可以根据针对bean名称的模式匹配来限制autowire候选者。 顶级&lt;beans /&gt;元素在其default-autowire-candidates属性中接受一个或多个模式。 例如，要将autowire候选状态限制为名称以Repository结尾的任何bean，请提供值\* Repository。 要提供多个模式，请在逗号分隔的列表中定义它们。 bean定义autowire-candidate属性的显式值true或false始终优先，对于此类bean，模式匹配规则不适用。

这些技术对于您永远不希望通过自动装配注入其他bean的bean非常有用。 这并不意味着排除的bean本身不能使用自动装配进行配置。 相反，bean本身不是自动装配其他bean的候选者。

### 7.4.6 方法注入

在大多数应用程序场景中，容器中的大多数bean都是单例。 当单例bean需要与另一个单例bean协作，或者非单例bean需要与另一个非单例bean协作时，通常通过将一个bean定义为另一个bean的属性来处理依赖关系。 当bean生命周期不同时会出现问题。 假设单例bean A需要使用非单例（原型）bean B，可能是在A上的每个方法调用上。容器只创建一次单例bean A，因此只有一次机会来设置属性。 每次需要时，容器都不能为bean A提供bean B的新实例。

解决方案是放弃一些控制反转。 您可以通过实现ApplicationContextAware接口使bean A了解容器，并通过对容器进行getBean（“B”）调用，每次bean A需要时都要求（通常是新的）bean B实例。 以下是此方法的示例：

```
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

前面的内容是不可取的，因为业务代码知道并耦合到Spring Framework。 Method Injection是Spring IoC容器的一个高级功能，它允许以干净的方式处理这个用例。

You can read more about the motivation for Method Injection in[this blog entry](https://spring.io/blog/2004/08/06/method-injection/).

#### lookup方法注入

Lookup方法注入是容器覆盖容器托管bean上的方法的能力，以返回容器中另一个命名bean的查找结果。 查找通常涉及原型bean，如上一节中描述的场景。 Spring Framework通过使用CGLIB库中的字节码生成来实现此方法注入，以动态生成覆盖该方法的子类。

* 要使这个动态子类工作，Spring bean容器将子类化的类不能是final，并且要重写的方法也不能是final。
* 对具有抽象方法的类进行单元测试需要您自己对类进行子类化，并提供抽象方法的存根实现。
* 组件扫描也需要具体方法，这需要具体的类别来获取。
* 另一个关键限制是查找方法不适用于工厂方法，特别是配置类中的@Bean方法，因为容器在这种情况下不负责创建实例，因此无法创建运行时生成的子类on the fly。

查看前面代码片段中的CommandManager类，您会看到Spring容器将动态覆盖createCommand（）方法的实现。 您的CommandManager类将不具有任何Spring依赖项，如重新编写的示例中所示：

```
package fiona.apple;

// no more Spring imports!

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

在包含要注入的方法的客户机类（本例中为CommandManager）中，要注入的方法需要以下形式的签名：

```
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法是抽象的，则动态生成的子类实现该方法。 否则，动态生成的子类将覆盖原始类中定义的具体方法。 例如：

```
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

标识为commandManager的bean在需要myCommand bean的新实例时调用自己的方法createCommand（）。 您必须小心将myCommand bean部署为原型，如果这实际上是需要的话。 如果它是一个单例，则每次都返回myCommand bean的相同实例。

或者，在基于注释的组件模型中，您可以通过@Lookup批注声明查找方法：

```
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

或者，更具惯用性，您可以依赖于针对查找方法的声明返回类型解析目标bean：

```
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```

请注意，您通常会使用具体的存根实现来声明这种带注释的查找方法，以使它们与Spring的组件扫描规则兼容，其中默认情况下抽象类被忽略。 此限制不适用于显式注册或显式导入的bean类。

_访问不同范围的目标bean的另一种方法是ObjectFactory / Provider注入点。 查看名为“Scoped beans as dependencies”的部分。_

_感兴趣的读者也可以找到ServiceLocatorFactoryBean（在org.springframework.beans.factory.config包中）。_

#### 任意方法更换

与查找方法注入相比，一种不太有用的方法注入形式是能够使用另一个方法实现替换托管bean中的任意方法。 用户可以安全地跳过本节的其余部分，直到实际需要该功能。

使用基于XML的配置元数据，您可以使用被替换的方法元素将已有的方法实现替换为已部署的bean。 考虑以下类，使用方法computeValue，我们要覆盖它：

```
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```

实现org.springframework.beans.factory.support.MethodReplacer接口的类提供了新的方法定义。

```
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```

部署原始类并指定方法覆盖的bean定义如下所示：

```
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

您可以在&lt;replacement-method /&gt;元素中使用一个或多个包含的&lt;arg-type /&gt;元素来指示被覆盖的方法的方法签名。 仅当方法重载且类中存在多个变体时，才需要参数的签名。 为方便起见，参数的类型字符串可以是完全限定类型名称的子字符串。 例如，以下所有内容都匹配java.lang.String：

```
java.lang.String
String
Str
```

因为参数的数量通常足以区分每个可能的选择，所以通过允许您只键入与参数类型匹配的最短字符串，此快捷方式可以节省大量的输入。

## 7.5 Bean范围

创建bean定义时，将创建一个配方，用于创建由该bean定义定义的类的实际实例。 bean定义是一个配方的想法很重要，因为它意味着，与一个类一样，您可以从一个配方创建许多对象实例。

您不仅可以控制要插入到从特定bean定义创建的对象中的各种依赖项和配置值，还可以控制从特定bean定义创建的对象的范围。这种方法功能强大且灵活，您可以选择通过配置创建的对象的范围，而不必在Java类级别烘焙对象的范围。可以将Bean定义为部署在多个范围之一中：开箱即用，Spring Framework支持七个范围，其中五个范围仅在您使用Web感知的ApplicationContext时才可用。

开箱即用支持以下范围。您还可以创建自定义范围。

| Scope | Description |
| :--- | :--- |
| [singleton](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-scopes-singleton) | （默认）将每个Spring IoC容器的单个bean定义范围限定为单个对象实例。 |
| [prototype](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-scopes-prototype) | 将单个bean定义范围限定为任意数量的对象实例。 |
| [request](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-scopes-request) | 将单个bean定义范围限定为单个HTTP请求的生命周期; 也就是说，每个HTTP请求都有自己的bean实例，它是在单个bean定义的后面创建的。 仅在Web感知Spring ApplicationContext的上下文中有效。 |
| [session](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-scopes-session) | 将单个bean定义范围限定为HTTP会话的生命周期。 仅在Web感知Spring ApplicationContext的上下文中有效。 |
| [globalSession](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-scopes-global-session) | 将单个bean定义范围限定为全局HTTP会话的生命周期。 通常仅在Portlet上下文中使用时有效。 仅在Web感知Spring ApplicationContext的上下文中有效。 |
| [application](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-scopes-application) | 将单个bean定义范围限定为ServletContext的生命周期。 仅在Web感知Spring ApplicationContext的上下文中有效。 |
| [websocket](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#websocket-stomp-websocket-scope) | 将单个bean定义范围限定为WebSocket的生命周期。 仅在Web感知Spring ApplicationContext的上下文中有效。 |

_从Spring 3.0开始，线程范围可用，但默认情况下未注册。 有关更多信息，请参阅SimpleThreadScope的文档。 有关如何注册此范围或任何其他自定义范围的说明，请参阅“使用自定义范围”一节。_

### 7.5.1 单例范围

只管理单个bean的一个共享实例，并且对具有与该bean定义匹配的id或id的bean的所有请求都会导致Spring容器返回一个特定的bean实例。

换句话说，当您定义一个bean定义并且它的范围是一个单例时，Spring IoC容器只创建该bean定义定义的对象的一个实例。 此单个实例存储在此类单例bean的缓存中，并且该命名Bean的所有后续请求和引用都将返回缓存对象。

Spring的单例bean概念不同于Gang of Four（GoF）模式书中定义的Singleton模式。 GoF Singleton对对象的范围进行硬编码，使得每个ClassLoader创建一个且只有一个特定类的实例。 Spring单例的范围最好按容器和每个bean描述。 这意味着如果在单个Spring容器中为特定类定义一个bean，那么Spring容器将创建该bean定义所定义的类的唯一一个实例。 单例范围是Spring中的默认范围。 要将bean定义为XML中的单例，您可以编写，例如：

```
<bean id="accountService" class="com.foo.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.foo.DefaultAccountService" scope="singleton"/>
```

### 7.5.2 原型范围

bean的非单例原型范围部署导致每次发出对该特定bean的请求时都会创建一个新的bean实例。 也就是说，bean被注入另一个bean，或者通过对容器的getBean（）方法调用来请求它。 通常，对所有有状态bean使用原型范围，对无状态bean使用单例范围。

下图说明了Spring原型范围。 数据访问对象（DAO）通常不配置为原型，因为典型的DAO不保持任何会话状态; 这个作者更容易重用单例图的核心。

以下示例将bean定义为XML中的原型：

```
<bean id="accountService" class="com.foo.DefaultAccountService" scope="prototype"/>
```

与其他作用域相比，Spring不管理原型bean的完整生命周期：容器实例化，配置和组装原型对象，并将其交给客户端，而不再记录该原型实例。因此，尽管无论范围如何都在所有对象上调用初始化生命周期回调方法，但在原型的情况下，不会调用已配置的销毁生命周期回调。客户端代码必须清理原型范围的对象并释放原型bean所持有的昂贵资源。要让Spring容器释放原型范围内的bean所拥有的资源，请尝试使用自定义bean后处理器，它包含对需要清理的bean的引用。

在某些方面，Spring容器关于原型范围bean的角色是Java new运算符的替代品。超过该点的所有生命周期管理必须由客户端处理。 （有关Spring容器中bean的生命周期的详细信息，请参见第7.6.1节“生命周期回调”。）

### 7.5.3 具有原型bean依赖关系的单例bean

当您使用具有依赖于原型bean的单例作用域bean时，请注意在实例化时解析依赖项。 因此，如果依赖项将原型范围的bean注入到单例范围的bean中，则会实例化一个新的原型bean，然后将依赖注入到单例bean中。 原型实例是唯一提供给单例范围bean的实例。

但是，假设您希望单例范围的bean在运行时重复获取原型范围的bean的新实例。 您不能将原型范围的bean依赖注入到您的单例bean中，因为当Spring容器实例化单例bean并解析并注入其依赖项时，该注入只发生一次。 如果您需要在运行时多次使用原型bean的新实例，请参见第7.4.6节“方法注入”。

### 7.5.4 请求，会话，全局会话，应用程序和WebSocket范围

Request, session, global session, application, and WebSocket scopes

请求，会话，globalSession，应用程序和websocket范围仅在您使用Web感知的Spring ApplicationContext实现（例如XmlWebApplicationContext）时才可用。 如果将这些作用域与常规的Spring IoC容器（如ClassPathXmlApplicationContext）一起使用，则会抛出IllegalStateException，抱怨未知的bean作用域。

#### 初始Web配置

要在请求，会话，globalSession，应用程序和websocket级别（Web范围的bean）支持bean的范围，在定义bean之前需要一些小的初始配置。 （standard scopes，singleton和prototype不需要此初始设置。）

如何完成此初始设置取决于您的特定Servlet环境。

如果您在Spring Web MVC中访问作用域bean，实际上是在Spring DispatcherServlet或DispatcherPortlet处理的请求中，则无需进行特殊设置：DispatcherServlet和DispatcherPortlet已公开所有相关状态。

如果您使用Servlet 2.5 Web容器，并且在Spring的DispatcherServlet之外处理请求（例如，使用JSF或Struts时），则需要注册org.springframework.web.context.request.RequestContextListener ServletRequestListener。对于Servlet 3.0+，可以通过WebApplicationInitializer界面以编程方式完成。或者，或者对于旧容器，将以下声明添加到Web应用程序的web.xml文件中：

```
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

或者，如果您的侦听器设置存在问题，请考虑使用Spring的RequestContextFilter。 过滤器映射取决于周围的Web应用程序配置，因此您必须根据需要进行更改。

```
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

DispatcherServlet，RequestContextListener和RequestContextFilter都完全相同，即将HTTP请求对象绑定到为该请求提供服务的Thread。 这使得请求和会话范围的bean可以在调用链的下游进一步使用。

#### 请求范围

考虑bean定义的以下XML配置：

```
<bean id="loginAction" class="com.foo.LoginAction" scope="request"/>
```

Spring容器通过对每个HTTP请求使用loginAction bean定义来创建LoginAction bean的新实例。 也就是说，loginAction bean的作用域是HTTP请求级别。 您可以根据需要更改创建的实例的内部状态，因为从同一个loginAction bean定义创建的其他实例将不会在状态中看到这些更改; 它们特别针对个人要求。 当请求完成处理时，将放弃作用于请求的bean。

使用注释驱动的组件或Java Config时，可以使用@RequestScope注释将组件分配给请求范围。

```
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

#### session范围

考虑bean定义的以下XML配置：

```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
```

Spring容器通过在单个HTTP会话的生存期内使用userPreferences bean定义来创建UserPreferences bean的新实例。 换句话说，userPreferences bean在HTTP会话级别有效地作用域。 与请求范围的bean一样，您可以根据需要更改创建的实例的内部状态，因为知道同样使用从同一userPreferences bean定义创建的实例的其他HTTP Session实例在状态中看不到这些更改 ，因为它们特定于单个HTTP会话。 最终丢弃HTTP会话时，也会丢弃作用于该特定HTTP会话的bean。

使用注释驱动的组件或Java Config时，可以使用@SessionScope注释将组件分配给会话范围。

```
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

#### global session范围

考虑bean定义的以下XML配置：

```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="globalSession"/>
```

globalSession作用域类似于标准HTTP会话作用域（如上所述），仅适用于基于portlet的Web应用程序的上下文。 portlet规范定义了构成单个portlet Web应用程序的所有portlet之间共享的全局会话的概念。 在globalSession范围定义的Bean的范围（或绑定）到全局portlet会话的生存期。

如果编写基于Servlet的标准Web应用程序并将一个或多个bean定义为具有globalSession作用域，则使用标准HTTP会话作用域，并且不会引发错误。

#### application范围

考虑bean定义的以下XML配置：

```
<bean id="appPreferences" class="com.foo.AppPreferences" scope="application"/>
```

Spring容器通过对整个Web应用程序使用appPreferences bean定义一次来创建AppPreferences bean的新实例。 也就是说，appPreferences bean的作用域是ServletContext级别，存储为常规的ServletContext属性。 这有点类似于Spring单例bean，但在两个重要方面有所不同：它是每个ServletContext的单例，而不是每个Spring的'ApplicationContext'（在任何给定的Web应用程序中可能有几个），它实际上是暴露的，因此 作为ServletContext属性可见。

使用注释驱动的组件或Java Config时，可以使用@ApplicationScope注释将组件分配给应用程序范围。

```
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

#### 作为依赖关系的bean（Scoped beans as dependencies）

Spring IoC容器不仅管理对象（bean）的实例化，还管理协作者（或依赖关系）的连接。 如果要将（例如）HTTP请求作用域bean注入到具有较长寿命范围的另一个bean中，您可以选择注入AOP代理来代替作用域bean。 也就是说，您需要注入一个代理对象，该对象公开与范围对象相同的公共接口，但也可以从相关范围（例如HTTP请求）检索真实目标对象，并将方法调用委托给真实对象。

您还可以在作为单例的范围内的bean之间使用&lt;aop：scoped-proxy /&gt;，然后引用通过可序列化的中间代理，从而能够在反序列化时重新获取目标单例bean。

_当针对范围原型的bean声明&lt;aop：scoped-proxy /&gt;时，共享代理上的每个方法调用都将导致创建一个新的目标实例，然后该调用将被转发到该目标实例。_

_此外，范围代理不是以生命周期安全的方式从较短范围访问bean的唯一方法。您也可以简单地将您的注入点（即构造函数/ setter参数或自动装配字段）声明为ObjectFactory &lt;MyTargetBean&gt;，允许getObject（）调用在每次需要时按需检索当前实例 - 而无需保留实例或单独存储它。_

_作为扩展变体，您可以声明ObjectProvider &lt;MyTargetBean&gt;，它提供了几个额外的访问变体，包括getIfAvailable和getIfUnique。_

_JSR-330的变体称为Provider，与Provider &lt;MyTargetBean&gt;声明一起使用，并且对每次检索尝试都使用相应的get（）调用。有关JSR-330整体的更多详细信息，请参见此处。_

---

以下示例中的配置只有一行，但了解“为什么”以及它背后的“如何”非常重要。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/>
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.foo.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

要创建这样的代理，可以将子&lt;aop:scoped-proxy /&gt;元素插入到作用域bean定义中（请参阅“选择要创建的代理类型”一节和第41章基于XML模式的配置）。 为什么在请求，会话，globalSession和自定义范围级别定义bean的定义需要&lt;aop:scoped-proxy/&gt;元素？ 让我们检查下面的单例bean定义，并将其与您需要为上述范围定义的内容进行对比（请注意，下面的userPreferences bean定义不完整）。

```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

在前面的示例中，单例bean userManager注入了对HTTP会话范围的bean userPreferences的引用。这里的重点是userManager bean是一个单例：它将在每个容器中实例化一次，并且它的依赖项（在这种情况下只有一个，userPreferences bean）也只注入一次。这意味着userManager bean只能在完全相同的userPreferences对象上操作，即最初注入的对象。

当将一个寿命较短的scoped bean注入一个寿命较长的scoped bean时，这不是你想要的行为，例如将一个HTTP Session-scoped合作bean作为依赖注入singleton bean。相反，您需要一个userManager对象，并且在HTTP会话的生命周期中，您需要一个特定于所述HTTP会话的userPreferences对象。因此，容器创建一个对象，该对象公开与UserPreferences类（理想情况下是UserPreferences实例的对象）完全相同的公共接口，该对象可以从作用域机制（HTTP请求，会话等）中获取真实的UserPreferences对象。容器将此代理对象注入userManager bean，该bean不知道此UserPreferences引用是代理。在此示例中，当UserManager实例在依赖注入的UserPreferences对象上调用方法时，它实际上是在代理上调用方法。然后，代理从（在这种情况下）HTTP会话中获取真实的UserPreferences对象，并将方法调用委托给检索到的真实UserPreferences对象。

因此，在将request-，session-和globalSession-scoped bean注入协作对象时，您需要以下正确和完整的配置：

```
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

#### 选择要创建的代理类型

默认情况下，当Spring容器为使用&lt;aop：scoped-proxy /&gt;元素标记的bean创建代理时，将创建基于CGLIB的类代理。

CGLIB代理只拦截公共方法调用！ 不要在这样的代理上调用非公开方法; 它们不会被委托给实际的作用域目标对象。

或者，您可以通过为&lt;aop：scoped-proxy /&gt;元素的proxy-target-class属性的值指定false来配置Spring容器，以便为此类作用域bean创建基于JDK接口的标准代理。 使用基于JDK接口的代理意味着您不需要在应用程序类路径中使用其他库来实现此类代理。 但是，它还意味着作用域bean的类必须至少实现一个接口，并且注入了作用域bean的所有协作者必须通过其中一个接口引用bean。

```
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.foo.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

有关选择基于类或基于接口的代理的更多详细信息，请参见第11.6节“代理机制”。

### 7.5.5 自定义范围

bean scoping是可扩展的，您可以定义自己的作用域，甚至可以重新定义现有作用域，尽管后者被认为是不好的做法，您无法覆盖内置的单例和原型作用域。

#### 创建customer范围

要将自定义作用域集成到Spring容器中，需要实现org.springframework.beans.factory.config.Scope接口，本节将对此进行介绍。 有关如何实现自己的作用域的想法，请参阅Spring Framework本身和Scope javadocs提供的Scope实现，它解释了您需要更详细地实现的方法。

Scope接口有四种方法可以从作用域中获取对象，从作用域中删除它们，并允许它们被销毁。

以下方法从基础范围返回对象。 例如，会话范围实现返回会话范围的bean（如果它不存在，则该方法在将其绑定到会话以供将来参考之后返回该bean的新实例）。

```
Object get(String name, ObjectFactory objectFactory)
```

以下方法从基础范围中删除对象。 例如，会话范围实现从基础会话中删除会话范围的bean。 应返回该对象，但如果找不到具有指定名称的对象，则可以返回null。

```
Object remove(String name)
```

以下方法注册范围应在销毁时或在范围中指定的对象被销毁时应执行的回调。 有关销毁回调的更多信息，请参阅javadocs或Spring作用域实现。

```
String getConversationId()
```

#### 使用自定义范围

在编写并测试一个或多个自定义Scope实现之后，您需要让Spring容器知道您的新范围。 以下方法是使用Spring容器注册新Scope的核心方法：

```
void registerScope(String scopeName, Scope scope);
```

此方法在ConfigurableBeanFactory接口上声明，该接口在通过BeanFactory属性随Spring提供的大多数具体ApplicationContext实现上可用。

registerScope（..）方法的第一个参数是与范围关联的唯一名称; Spring容器本身的这些名称的例子是singleton和prototype。 registerScope（..）方法的第二个参数是您希望注册和使用的自定义Scope实现的实际实例。

假设您编写自定义Scope实现，然后按如下所示进行注册。

下面的示例使用Spring附带的SimpleThreadScope，但默认情况下未注册。 您自己的自定义Scope实现的说明是相同的。

```
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

然后，您创建符合自定义作用域的作用域规则的bean定义：

```
<bean id="..." class="..." scope="thread">
```

使用自定义Scope实现，您不仅限于范围的编程注册。 您还可以使用CustomScopeConfigurer类以声明方式执行Scope注册：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="bar" class="x.y.Bar" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="foo" class="x.y.Foo">
        <property name="bar" ref="bar"/>
    </bean>

</beans>
```

当您在FactoryBean实现中放置&lt;aop:scoped-proxy /&gt;时，它是作用域的工厂bean本身，而不是从getObject（）返回的对象。

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




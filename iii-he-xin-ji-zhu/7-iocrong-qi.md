# 7 Ioc容器

## 7.1 Spring IoC容器和bean简介

本章介绍了Spring Framework实现的控制反转（IoC）原理。 IoC也称为依赖注入（DI）。 这是一个过程，通过这个过程，对象定义它们的依赖关系，即它们使用的其他对象，只能通过构造函数参数，工厂方法的参数，或者在构造或从工厂方法返回后在对象实例上设置的属性。 然后容器在创建bean时注入这些依赖项。 这个过程基本上是反向的，因此名称Inversion of Control（IoC），bean本身通过使用类的直接构造或诸如Service Locator模式之类的机制来控制其依赖关系的实例化或位置。

org.springframework.beans和org.springframework.context包是Spring Framework的IoC容器的基础。 BeanFactory接口提供了一种能够管理任何类型对象的高级配置机制。 ApplicationContext是BeanFactory的子接口。 它增加了与Spring的AOP功能的更容易的集成; 消息资源处理（用于国际化），事件发布; 和特定于应用程序层的上下文，例如WebApplicationContext，用于Web应用程序。

简而言之，BeanFactory提供配置框架和基本功能，ApplicationContext添加了更多特定于企业的功能。 ApplicationContext是BeanFactory的完整超集，在本章中专门用于Spring的IoC容器的描述。 有关使用BeanFactory而不是ApplicationContext的更多信息，请参见第7.16节“BeanFactory”。

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。 bean是一个由Spring IoC容器实例化，组装和管理的对象。 否则，bean只是应用程序中众多对象之一。 Bean及其之间的依赖关系反映在容器使用的配置元数据中。

## 7.2 容器概述

接口org.springframework.context.ApplicationContext表示Spring IoC容器，负责实例化，配置和组装上述bean。 容器通过读取配置元数据获取有关要实例化，配置和组装的对象的指令。 配置元数据以XML，Java注释或Java代码表示。 它允许您表达组成应用程序的对象以及这些对象之间丰富的相互依赖性。

ApplicationContext接口的几个实现是与Spring一起提供的。 在独立应用程序中，通常会创建ClassPathXmlApplicationContext或FileSystemXmlApplicationContext的实例。 虽然XML是定义配置元数据的传统格式，但您可以通过提供少量XML配置来声明性地支持这些其他元数据格式，从而指示容器使用Java注释或代码作为元数据格式。

在大多数应用程序方案中，不需要显式用户代码来实例化Spring IoC容器的一个或多个实例。 例如，在Web应用程序场景中，应用程序的web.xml文件中的简单八行（左右）样板Web描述符XML通常就足够了（请参见第7.15.4节“Web应用程序的便捷ApplicationContext实例化”）。 如果您使用的是Spring工具套件Eclipse驱动的开发环境，只需点击几下鼠标或按键即可轻松创建此样板文件配置。

下图是Spring工作原理的高级视图。 您的应用程序类与配置元数据相结合，以便在创建和初始化ApplicationContext之后，您拥有一个完全配置且可执行的系统或应用程序。

![](/assets/thespringioccontainer.png)

### 7.2.1配置元数据

如上图所示，Spring IoC容器使用一种配置元数据形式; 此配置元数据表示您作为应用程序开发人员如何告诉Spring容器在应用程序中实例化，配置和组装对象。

传统上，配置元数据以简单直观的XML格式提供，本章大部分内容用于传达Spring IoC容器的关键概念和功能。

_基于XML的元数据不是唯一允许的配置元数据形式。 Spring IoC容器本身完全与实际编写此配置元数据的格式分离。 目前，许多开发人员为其Spring应用程序选择基于Java的配置。_

有关在Spring容器中使用其他形式的元数据的信息，请参阅：

* 基于注释的配置\(7.9节\)：Spring 2.5引入了对基于注释的配置元数据的支持。
* 基于Java的配置\(7.12节\)：从Spring 3.0开始，Spring JavaConfig项目提供的许多功能成为核心Spring Framework的一部分。 因此，您可以使用Java而不是XML文件在应用程序类外部定义bean。 要使用这些新功能，请参阅@Configuration，@ Label，@ Import和@DependsOn注释。

Spring配置由容器必须管理的至少一个且通常不止一个bean定义组成。 基于XML的配置元数据显示这些bean在顶级&lt;beans /&gt;元素中配置为&lt;bean/&gt;元素。 Java配置通常在@Configuration类中使用@Bean注释方法。

这些bean定义对应于构成应用程序的实际对象。 通常，您定义服务层对象，数据访问对象（DAO），表示对象（如Struts Action实例），基础结构对象（如Hibernate SessionFactories，JMS队列等）。 通常，不会在容器中配置细粒度域对象，因为DAO和业务逻辑通常负责创建和加载域对象。 但是，您可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。 请参阅使用AspectJ使用Spring依赖注入域对象。

以下示例显示了基于XML的配置元数据的基本结构：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

id属性是一个字符串，用于标识单个bean定义。 class属性定义bean的类型并使用完全限定的classname。 id属性的值指的是协作对象。 本例中未显示用于引用协作对象的XML; 有关更多信息，请参阅依赖项。

### 7.2.2实例化容器

实例化Spring IoC容器非常简单。 提供给ApplicationContext构造函数的位置路径实际上是资源字符串，允许容器从各种外部资源（如本地文件系统，Java CLASSPATH等）加载配置元数据。

```
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

在了解了Spring的IoC容器之后，您可能想要了解有关Spring的资源抽象的更多信息，如第8章“资源”中所述，它提供了一种从URI语法中定义的位置读取InputStream的便捷机制。 特别是，资源路径用于构建应用程序上下文，如第8.7节“应用程序上下文和资源路径”中所述。

以下示例显示了服务层对象（services.xml）配置文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

以下示例显示了数据访问对象daos.xml文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

在前面的示例中，服务层由PetStoreServiceImpl类和两个JpaAccountDao和JpaItemDao类型的数据访问对象组成（基于JPA对象/关系映射标准）。属性name元素引用JavaBean属性的名称，ref元素引用另一个bean定义的名称。id和ref元素之间的这种联系表达了协作对象之间的依赖关系。有关配置对象的依赖项的详细信息，请参阅依赖项。

#### 编写基于XML的配置元数据

让bean定义跨越多个XML文件会很有用。 通常，每个单独的XML配置文件都代表架构中的逻辑层或模块。

您可以使用应用程序上下文构造函数从所有这些XML片段加载bean定义。 此构造函数采用多个Resource位置，如上一节中所示。 或者，使用一个或多个&lt;import /&gt;元素来从另一个或多个文件加载bean定义。 例如：

```
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

在前面的示例中，外部bean定义从三个文件加载：services.xml，messageSource.xml和themeSource.xml。 所有位置路径都与执行导入的定义文件相关，因此services.xml必须与执行导入的文件位于相同的目录或类路径位置，而messageSource.xml和themeSource.xml必须位于该位置下的资源位置 导入文件。 正如您所看到的，忽略了一个前导斜杠，但考虑到这些路径是相对的，最好不要使用斜杠。 根据Spring Schema，要导入的文件的内容（包括顶级&lt;beans/&gt;元素）必须是有效的XML bean定义。

可以（但不建议）使用相对“../”路径引用父目录中的文件。 这样做会对当前应用程序之外的文件创建依赖关系。 特别是，不建议将此引用用于“classpath：”URL（例如，“classpath：../ services.xml”），其中运行时解析过程选择“最近的”类路径根，然后查看其父目录。 类路径配置更改可能导致选择不同的，不正确的目录。

您始终可以使用完全限定的资源位置而不是相对路径：例如，“file：C:/config/services.xml”或“classpath:/config/services.xml”。 但是，请注意您将应用程序的配置与特定的绝对位置耦合。 通常最好为这样的绝对位置保持间接，例如，通过在运行时针对JVM系统属性解析的“$ {...}”占位符。

import指令是beans命名空间本身提供的功能。 除了普通bean定义之外的其他配置功能在Spring提供的一系列XML命名空间中可用，例如： “context”和“util”命名空间。

### 7.2.3 使用容器

ApplicationContext是高级工厂的接口，能够维护不同bean及其依赖项的注册表。 使用方法T getBean（String name，Class &lt;T&gt; requiredType），您可以检索bean的实例。

ApplicationContext使您可以读取bean定义并按如下方式访问它们：

```
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

使用Groovy配置，bootstrapping看起来非常相似，只是一个不同的上下文实现类，它是Groovy感知的（但也理解XML bean定义）：

```
ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");
```

最灵活的变体是GenericApplicationContext与读者代表的组合，例如， 使用XML文件的XmlBeanDefinitionReader：

```
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

或者使用GroovyBeanDefinitionReader for Groovy文件：

```
GenericApplicationContext context = new GenericApplicationContext();
new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
context.refresh();
```

这些读者委托可以在同一个ApplicationContext上混合和匹配，如果需要，可以从不同的配置源读取bean定义。

然后，您可以使用getBean来检索Bean的实例。 ApplicationContext接口有一些其他方法可以检索bean，但理想情况下，您的应用程序代码绝不应该使用它们。 实际上，您的应用程序代码根本不应该调用getBean\(\)方法，因此根本不依赖于Spring API。 例如，Spring与Web框架的集成为各种Web框架组件（如控制器和JSF托管bean）提供依赖注入，允许您通过元数据（例如自动装配注释）声明对特定bean的依赖性。

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

#### 直值（基本类型，字符串等）

#### 引用其他bean（协作者）

#### Inner beans

#### 集合

#### 空值和空字符串值

#### 带有p命名空间的XML快捷方式

#### 带有c-namespace的XML快捷方式

#### 复合属性名称

### 7.4.3 使用依赖

### 7.4.4 懒惰初始化的bean

### 7.4.5 自动装配协作者

#### 自动装配的局限和缺点

#### 从自动装配中排除一个bean

### 7.4.6 方法注入

#### 查找方法注入

#### 任意方法更换

7.5 Bean范围

7.5.1 single type

7.5.2 原型范围

7.5.3 具有原型bean依赖关系的单例bean

7.5.4 请求，会话，全局会话，应用程序和WebSocket范围

初始Web配置

请求范围

会话范围

全球会议范围

适用范围

作为依赖关系的范围豆

7.5.5 自定义范围

创建自定义范围

使用自定义范围

7.6 自定义bean的本质

7.6.1 生命周期回调

初始化回调

销毁回调

默认初始化和销毁​​方法

结合生命周期机制

启动和关闭回调

在非Web应用程序中正常关闭Spring IoC容器

7.6.2  ApplicationContextAware和BeanNameAware

7.6.3 其他Aware接口

7.7  Bean定义继承

7.8 集装箱扩建点

7.8.1 使用BeanPostProcessor定制bean

示例：Hello World，BeanPostProcessor样式

示例：RequiredAnnotationBeanPostProcessor

7.8.2 使用BeanFactoryPostProcessor定制配置元数据

示例：类名替换PropertyPlaceholderConfigurer

示例：PropertyOverrideConfigurer

7.8.3 使用FactoryBean自定义实例化逻辑

7.9 基于注释的容器配置

7.9.1  @Required

7.9.2  @Autowired

7.9.3 使用@Primary微调基于注释的自动装配

7.9.4 使用限定符微调基于注释的自动装配

7.9.5 使用泛型作为自动装配限定符

7.9.6  CustomAutowireConfigurer上

7.9.7  @Resource

7.9.8  @PostConstruct和@PreDestroy

7.10 类路径扫描和托管组件

7.10.1  @Component和进一步的构造型注释

7.10.2 元注释

7.10.3 自动检测类并注册bean定义

7.10.4 使用过滤器自定义扫描

7.10.5 在组件中定义bean元数据

7.10.6 命名自动检测的组件

7.10.7 为自动检测的组件提供范围

7.10.8 提供带注释的限定符元数据

7.11 使用JSR 330标准注释

7.11.1 使用@Inject和@Named进行依赖注入

7.11.2  @Named和@ManagedBean：@Component注释的标准等价物

7.11.3  JSR-330标准注释的局限性

7.12 基于Java的容器配置

7.12.1 基本概念：@Bean和@Configuration

7.12.2 使用AnnotationConfigApplicationContext实例化Spring容器

结构简单

使用register（Class &lt;？&gt; ...）以编程方式构建容器

使用扫描启用组件扫描（String ...）

使用AnnotationConfigWebApplicationContext支持Web应用程序

7.12.3 使用@Bean注释

声明一个bean

Bean依赖项

接收生命周期回调

指定bean范围

自定义bean命名

Bean别名

bean的描述

7.12.4 使用@Configuration注释

注入bean间依赖关系

查找方法注入

有关基于Java的配置如何在内部工作的更多信息

7.12.5 编写基于Java的配置

使用@Import注释

有条件地包括@Configuration类或@Bean方法

结合Java和XML配置

7.13 环境抽象

7.13.1  Bean定义配置文件

@Profile

XML bean定义配置文件

激活profile

默认profile

7.13.2  PropertySource抽象

7.13.3  @PropertySource

7.13.4 占位符决议在陈述中

7.14 注册LoadTimeWeaver

7.15  ApplicationContext的其他功能

7.15.1 使用MessageSource进行国际化

7.15.2 标准和自定义活动

基于注释的事件侦听器

异步监听器

订购听众

通用事件

7.15.3 方便地访问低级资源

7.15.4 方便的Web应用程序的ApplicationContext实例化

7.15.5 将Spring ApplicationContext部署为Java EE RAR文件

7.16  BeanFactory

7.16.1  BeanFactory或ApplicationContext？

7.16.2 耦合代码与不良单例


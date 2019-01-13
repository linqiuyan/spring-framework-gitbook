## 7.2 容器概述

接口org.springframework.context.ApplicationContext表示Spring IoC容器，负责实例化，配置和组装上述bean。 容器通过读取配置元数据获取有关要实例化，配置和组装的对象的指令。 配置元数据以XML，Java注释或Java代码表示。 它允许您表达组成应用程序的对象以及这些对象之间丰富的相互依赖性。

ApplicationContext接口的几个实现是与Spring一起提供的。 在独立应用程序中，通常会创建ClassPathXmlApplicationContext或FileSystemXmlApplicationContext的实例。 虽然XML是定义配置元数据的传统格式，但您可以通过提供少量XML配置来声明性地支持这些其他元数据格式，从而指示容器使用Java注释或代码作为元数据格式。

在大多数应用程序方案中，不需要显式用户代码来实例化Spring IoC容器的一个或多个实例。 例如，在Web应用程序场景中，应用程序的web.xml文件中的简单八行（左右）样板Web描述符XML通常就足够了（请参见第7.15.4节“Web应用程序的便捷ApplicationContext实例化”）。 如果您使用的是Spring工具套件Eclipse驱动的开发环境，只需点击几下鼠标或按键即可轻松创建此样板文件配置。

下图是Spring工作原理的高级视图。 您的应用程序类与配置元数据相结合，以便在创建和初始化ApplicationContext之后，您拥有一个完全配置且可执行的系统或应用程序。

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


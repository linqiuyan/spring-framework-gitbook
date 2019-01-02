# Part I. Spring Framework概述

Spring框架是一个轻量级一站式企业应用解决方案。Spring是模块化的，允许你只引入你需要的部分，不需要将所有的模块全部引入。你可以仅使用IOC容器，在任何web框架项目中。同样，你也可以只引入 [Hibernate集成代码](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#orm-hibernate) 或者 [JDBC抽象层](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#jdbc-introduction)。Spring框架支持声明式的事务管理、远程数据访问RMI、web服务等来持久化数据。它提供了一个功能完整的[MVC框架](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#mvc-introduction)，允许你透明地将[AOP](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#aop-introduction)整合到软件中。

Spring的设计是非侵入性的，这意味着你的逻辑代码通常不会依赖你的框架本身。在集成层（如数据访问层），一些依赖于数据访问技术需要Spring库的存在，但它也可以很容易地去隔离这些依赖。

这个文档可以作为Spring Framework功能的指导手册。如果你有任何请求、评论或问题在这个文档中，请发邮件给我们。任何框架相关的问题，可以在StackOverflow上咨询（[https://spring.io/questions](https://spring.io/questions)）。

## 1 开始使用

这个参考指南提供了关于Spring框架的详细信息，全面的功能特性介绍以及一些背景的基础概念\(如“依赖注入”\)。

如果你是刚刚开始Spring，你可以通过创建一个[Spring Boot](https://projects.spring.io/spring-boot/)应用程序来快速使用Spring框架。Spring Boot提供了一种快速创建Spring应用的途径。它是基于Spring框架，支持约定优于配置，目的是让你能尽快启动并运行。

你可以使用 [start.spring.io](https://start.spring.io/) 生成一个基本项目或者使用["Getting Started"](https://spring.io/guides) 类似[Getting Started Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/) 来创建。这些指导手册更容易理解，大部分项目都是基于Spring Boot来创建的。如果你想去解决一个特定的问题，也可以使用这些指导手册去参阅。

## 2 Spring框架简介

Spring Framework是一个Java平台，为开发Java应用程序提供全面的基础架构支持。 Spring处理基础架构，因此您可以专注于您的应用程序。

Spring允许您从“普通旧Java对象”（POJO）构建应用程序，并以非侵入方式将企业服务应用于POJOs。 此功能适用于Java SE编程模型以及完整和部分Java EE。

作为应用程序开发人员，您可以从Spring平台中受益的示例如下：

* 允许Java方法在数据库事务中执行，而不需要使用处理事务的APIs。
* 使本地Java方法成为HTTP端点，而无需处理Servlet API。
* 使本地Java方法成为消息处理程序，而无需处理JMS API。
* 使本地Java方法成为管理操作，而无需处理JMX API。

### 2.1 依赖注入与控制反转

Java应用程序，从嵌入式应用程序到n层服务器端企业应用程序，通常都有很多协作对象组成，这些应用程序中的对象彼此依赖。

尽管Java平台提供了丰富的应用程序开发功能，但它缺乏将基本构建块组织成一个连贯整体的方法，将该任务留给架构师和开发人员。 虽然您可以使用如工厂，抽象工厂，构建器，装饰器和服务Locatorto之类的设计模式，来组成构成应用程序的各种类和对象实例。设计模式仅仅只描述了它的作用、使用方式以及能解决的问题，而在应用程序中自己实际应用这些设计模式才是最佳实践。

Spring框架控制反转（IoC）组件，提供将不同组件组合成一个可以使用的应用程序的形式化方法来解决这一问题。 Spring Framework将形式化的设计模式编码为可以集成到自己的应用程序中。 许多组织和机构以这种方式使用Spring Framework来设计健壮，可维护的应用程序。

```
背景
“问题是，控制哪个方面的反转？” Martin Fowler于2004年在他的网站上提出了关于控制反转（IoC）的问题.
Fowler建议重新命名原则，使其更加不言自明，并想出了依赖注入。
```

### 2.2 框架模块

Spring Framework由大约20个模块组成的功能组成。 这些模块分为核心容器，数据访问/集成，Web，AOP（面向切面编程）， Instrumentation，消息传递和测试，如下图所示。

图2.1——————————————————

接下来部分将列举每个功能模块的工程名及其相关主题。工程名关联的_artifact IDs_在[依赖管理工具](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#dependency-management)中。

#### 2.2.1 核心容器

核心容器由`spring-core`,`spring-beans`,`spring-context`,`spring-context-support`, `spring-expression`\(Spring Expression Language\) 几个模块组成。

spring-core与spring-beans两个模块提供了framework最基础的部分，包含IoC与依赖注入功能。`BeanFactory`是工厂模式的复杂实现。 它消除了对程序化单例的需求，并允许您从实际程序逻辑中分离依赖项的配置和规范。

Context（spring-context）模块构建在Core和Beans模块提供的坚实基础之上：它是一种以类似于JNDI注册表的框架样式方式访问对象的方法。 Context模块从Beans模块继承其功能，并添加对国际化（例如，使用资源包），事件传播，资源加载以及通过例如Servlet容器透明创建上下文的支持。 Context模块还支持Java EE功能，例如EJB，JMX和基本远程处理。 ApplicationContext接口是Context模块的焦点。 spring-context-support支持将常见的第三方库集成到Spring应用程序上下文中，用于缓存（EhCache，Guava，JCache），邮件（JavaMail），调度（CommonJ，Quartz）和模板引擎（FreeMarker，JasperReports，Velocity）。

spring-expression模块提供了强大的表达式语言，用于在运行时查询和操作对象图。 它是JSP 2.1规范中指定的统一表达式语言（统一EL）的扩展。 该语言支持设置和获取属性值，属性赋值，方法调用，访问数组，集合和索引器的内容，逻辑和算术运算符，命名变量以及从Spring的IoC容器中按名称检索对象。 它还支持列表投影和选择以及常用列表聚合。

#### 2.2.2 AOP

spring-aop模块提供符合AOP Alliance标准的面向方面的编程实现。例如在方法拦截器和切入点，以干净地解耦实现应该分离的功能的代码。 使用源级元数据功能，您还可以以类似于.NET属性的方式将行为信息合并到代码中。

提供与AspectJ的集成的独立spring-aspects模块。

spring-instrument模块提供了在某些应用程序服务器中使用的类检测支持和类加载器实现。 spring-instrument-tomcat模块包含Spring的Tomcat检测代理。

#### 2.2.3 Messaging

Spring Framework 4包含一个Spring-messaging模块，其中包含来自Spring Integration项目的关键抽象，如Message，MessageChannel，MessageHandler等，可作为基于消息传递的应用程序的基础。 该模块还包括一组用于将消息映射到方法的注释，类似于基于Spring MVC注释的编程模型。

#### 2.2.4 数据访问/集成

数据访问/集成层由JDBC，ORM，OXM，JMS和事务模块组成。

* spring-jdbc模块提供了一个JDBC抽象层，无需进行繁琐的JDBC编码和解析数据库供应商特定的错误代码。
* spring-tx模块支持对实现特殊接口的类和所有POJO（普通旧Java对象）的类进行编程和声明式事务管理。
* spring-orm模块为流行的对象关系映射API提供集成层，包括JPA，JDO和Hibernate。使用spring-orm模块，您可以将所有这些O / R映射框架与Spring提供的所有其他功能结合使用，例如前面提到的简单声明式事务管理功能。
* spring-oxm模块提供了一个抽象层，支持对象/ XML映射实现，如JAXB，Castor，XMLBeans，JiBX和XStream。
* spring-jms模块（Java Messaging Service）包含用于生成和使用消息的功能。从Spring Framework 4.1开始，它提供了与spring-messaging模块的集成。

#### 2.2.5 Web

Web层由spring-web，spring-webmvc，spring-websocket和spring-webmvc-portlet模块组成。

* spring-web模块提供基本的面向Web的集成功能，例如多部分文件上载功能以及使用Servlet侦听器和面向Web的应用程序上下文初始化IoC容器。 它还包含一个HTTP客户端以及Spring的远程支持的Web相关部分。
* spring-webmvc模块（也称为Web-Servlet模块）包含用于Web应用程序的Spring的模型 - 视图 - 控制器（MVC）和REST Web服务实现。 Spring的MVC框架提供了域模型代码和Web表单之间的清晰分离，并与Spring Framework的所有其他功能集成在一起。
* spring-webmvc-portlet模块（也称为Web-Portlet模块）提供了在Portlet环境中使用的MVC实现，并镜像了基于Servlet的spring-webmvc模块的功能。

#### 2.2.6 Test

spring-test 模块支持使用JUnit或TestNG对Spring组件进行单元测试和集成测试。 它提供Spring ApplicationContexts的一致加载和这些上下文的缓存。 它还提供了可用于独立测试代码的模拟对象。

### 2.3 使用场景

前面描述的构建块使Spring成为许多场景中的合理选择，从在资源受限设备上运行的嵌入式应用程序到使用Spring的事务管理功能和Web框架集成的成熟企业应用程序。

图2.2 典型的完整Spring Web应用程序————————————————

Spring的声明式事务管理功能使Web应用程序完全是事务性的，就像使用EJB容器管理的事务一样。 您可以使用简单的POJO实现所有自定义业务逻辑，并由Spring的IoC容器管理。 其他服务包括支持发送独立于Web层的电子邮件和验证，这使您可以选择执行验证规则的位置。 Spring的ORM支持与JPA，Hibernate和JDO集成在一起; 例如，在使用Hibernate时，您可以继续使用现有的映射文件和标准的Hibernate SessionFactory配置。 表单控制器将Web层与域模型无缝集成，无需ActionForms或其他将HTTP参数转换为域模型值的类。

图2.3 Spring中间层使用第三方Web框架————————————————

有时情况不允许您完全切换到不同的框架。 Spring Framework不会强迫您使用其中的所有内容; 它不是一个全有或全无的解决方案。 使用Struts，Tapestry，JSF或其他UI框架构建的现有前端可以与基于Spring的中间层集成，从而允许您使用Spring事务功能。 您只需使用ApplicationContext连接业务逻辑，并使用WebApplicationContext集成Web层。

图2.4 远程调用场景————————————————

当您需要通过Web服务访问现有代码时，可以使用Spring的Hessian，Burlap，Rmi或JaxRpcProxyFactory类。 启用对现有应用程序的远程访问并不困难。

图2.5 EJB  - 包装现有的POJO ————————————

Spring Framework还为Enterprise JavaBeans提供了一个访问和抽象层，使您能够重用现有的POJO并将它们包装在无状态会话Bean中，以便在可能需要声明性安全性的可伸缩，故障安全Web应用程序中使用。

#### 2.3.1 依赖管理与命名约定

依赖管理和依赖注入是不同的事情。 要将Spring的这些优秀功能集成到您的应用程序中（如依赖注入），您需要组装所需的所有库（jar文件）并在运行时将它们放到类路径中，并且可能在编译时。 这些依赖项不是注入的虚拟组件，而是文件系统中的物理资源（通常）。 依赖关系管理过程涉及定位这些资源，存储它们并将它们添加到类路径中。 依赖关系可以是直接的（例如我的应用程序在运行时依赖于Spring），也可以是间接的（例如，我的应用程序依赖于commons-dbcp，它取决于commons-pool）。 间接依赖关系也称为“传递”，它是最难识别和管理的依赖关系。

如果你打算使用Spring，你需要获得一个包含你需要的Spring部分的jar库的副本。 为了使这更容易，Spring被打包为一组模块，尽可能地分离依赖项，因此，例如，如果您不想编写Web应用程序，则不需要spring-web模块。 要在本指南中引用Spring库模块，我们使用简写命名约定spring- \*或spring  -  \* .jar，其中\*表示模块的简称（例如spring-core，spring-webmvc，spring-jms等。）。 您使用的实际jar文件名通常是与版本号连接的模块名称（例如spring-core-4.3.21.RELEASE.jar）。

Spring Framework的每个版本都会将工件发布到以下位置：

* Maven Central，它是Maven查询的默认存储库，不需要任何特殊配置即可使用。 Spring所依赖的许多公共库也可以从Maven Central获得，而Spring社区的很大一部分都使用Maven进行依赖管理，因此这对他们来说很方便。 这里的罐子名称是spring  -  \*  -  &lt;version&gt; .jar，Maven groupId是org.springframework。
* 在专门为Spring托管的公共Maven存储库中。 除了最终的GA版本之外，该存储库还托管了开发快照和里程碑。 jar文件名与Maven Central的格式相同，因此这是一个有用的地方，可以将Spring的开发版本与Maven Central中部署的其他库一起使用。 此存储库还包含一个捆绑包分发zip文件，其中包含捆绑在一起的所有Spring jar，以便于下载。

因此，您需要决定的第一件事是如何管理您的依赖项：我们通常建议使用Maven，Gradle或Ivy等自动化系统，但您也可以通过自己下载所有jar来手动完成。

您将在下面找到Spring工件列表。 有关每个模块的更完整描述，请参见第2.2节“框架模块”。

| GroupId | ArtifactId | Description |
| :--- | :--- | :--- |
| org.springframework | spring-aop | Proxy-based AOP support |
| org.springframework | spring-aspects | AspectJ based aspects |
| org.springframework | spring-beans | Beans support, including Groovy |
| org.springframework | spring-context | Application context runtime, including scheduling and remoting abstractions |
| org.springframework | spring-context-support | Support classes for integrating common third-party libraries into a Spring application context |
| org.springframework | spring-core | Core utilities, used by many other Spring modules |
| org.springframework | spring-expression | Spring Expression Language \(SpEL\) |
| org.springframework | spring-instrument | Instrumentation agent for JVM bootstrapping |
| org.springframework | spring-instrument-tomcat | Instrumentation agent for Tomcat |
| org.springframework | spring-jdbc | JDBC support package, including DataSource setup and JDBC access support |
| org.springframework | spring-jms | JMS support package, including helper classes to send/receive JMS messages |
| org.springframework | spring-messaging | Support for messaging architectures and protocols |
| org.springframework | spring-orm | Object/Relational Mapping, including JPA and Hibernate support |
| org.springframework | spring-oxm | Object/XML Mapping |
| org.springframework | spring-test | Support for unit testing and integration testing Spring components |
| org.springframework | spring-tx | Transaction infrastructure, including DAO support and JCA integration |
| org.springframework | spring-web | Foundational web support, including web client and web-based remoting |
| org.springframework | spring-webmvc | HTTP-based Model-View-Controller and REST endpoints for Servlet stacks |
| org.springframework | spring-webmvc-portlet | MVC implementation to be used in a Portlet environment |
| org.springframework | spring-websocket | WebSocket and SockJS infrastructure, including STOMP messaging support |

##### Spring依赖和依赖于Spring

尽管Spring为大量企业和其他外部工具提供集成和支持，但它有意将其强制依赖性保持在最低限度：您不必定位和下载（甚至自动）大量jar库，以便 将Spring用于简单的用例。 对于基本依赖项注入，只有一个强制外部依赖项，即用于日志记录（有关日志记录选项的更详细说明，请参见下文）。

接下来，我们概述了配置依赖于Spring的应用程序所需的基本步骤，首先使用Maven，然后使用Gradle，最后使用Ivy。 在所有情况下，如果有任何不清楚的地方，请参阅依赖关系管理系统的文档，或者查看一些示例代码 -  Spring本身使用Gradle来构建依赖关系，我们的示例主要使用Gradle或Maven。

##### Maven依赖管理

如果您使用Maven进行依赖项管理，则甚至不需要显式提供日志记录依赖项。 例如，要创建应用程序上下文并使用依赖项注入来配置应用程序，您的Maven依赖项将如下所示：

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.3.21.RELEASE</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

请注意，如果您不需要针对Spring API进行编译，则可以将范围声明为运行时，这通常是基本依赖项注入用例的情况。

上面的示例适用于Maven Central存储库。 要使用Spring Maven存储库（例如，用于里程碑或开发人员快照），您需要在Maven配置中指定存储库位置。 对于完整版本：

```
<repositories>
    <repository>
        <id>io.spring.repo.maven.release</id>
        <url>http://repo.spring.io/release/</url>
        <snapshots><enabled>false</enabled></snapshots>
    </repository>
</repositories>
```

##### Maven“物料清单”依赖

使用Maven时，可能会意外混合不同版本的Spring JAR。 例如，您可能会发现第三方库或另一个Spring项目将旧版本的传递依赖性拉入其中。 如果您忘记自己明确声明直接依赖，则可能会出现各种意外问题。

为了克服这些问题，Maven支持“物料清单”（BOM）依赖性的概念。 您可以在dependencyManagement部分中导入spring-framework-bom，以确保所有spring依赖项（直接和传递）都在同一版本中。

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-framework-bom</artifactId>
            <version>4.3.21.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

使用BOM的另一个好处是，在依赖Spring Framework工件时，您不再需要指定&lt;version&gt;属性：

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </dependency>
<dependencies>
```

#### 2.3.2 Logging

日志记录是Spring的一个非常重要的依赖项，因为：

1. 它是唯一强制性的外部依赖项
2. 每个人都喜欢从他们使用的工具中看到一些输出，
3. Spring集成了许多其他工具，所有这些工具也都是选择日志记录依赖项。应用程序开发人员的目标之一通常是在整个应用程序的中心位置配置统一日志记录，包括所有外部组件。由于有很多日志框架选择，因此这比以往更难。

Spring中的强制日志记录依赖是Jakarta Commons Logging API（JCL）。我们针对JCL进行编译，并且我们还使JCL Log对象对于扩展Spring Framework的类是可见的。对于用户来说，所有版本的Spring都使用相同的日志库是很重要的：迁移很容易，因为即使扩展Spring的应用程序也可以保留向后兼容性。我们这样做的方法是让Spring中的一个模块明确依赖于commons-logging（JCL的规范实现），然后让所有其他模块在编译时依赖于它。例如，如果您正在使用Maven，并想知道在哪里获得了对commons-logging的依赖，那么它来自Spring，特别是来自名为spring-core的中央模块。

关于commons-logging的好处是你不需要任何其他东西来使你的应用程序工作。它有一个运行时发现算法，可以在类路径中的众所周知的位置查找其他日志记录框架，并使用它认为合适的一个（或者如果需要，可以告诉它哪一个）。如果没有其他可用的东西，你只需从JDK（java.util.logging或简称JUL）获得相当漂亮的日志。在大多数情况下，您应该会发现Spring应用程序可以正常运行并快速登录到控制台，这很重要。

##### 使用Log4j 1.2 或 2.x

```
Log4j 1.2同时也是EOL。 此外，Log4j 2.3是最新的Java 6兼容版本，较新的Log4j 2.x版本需要Java 7+。
```

许多人使用Log4j作为日志框架进行配置和管理。 它是高效且完善的，事实上它是我们在构建Spring时在运行时使用的。 Spring还提供了一些用于配置和初始化Log4j的实用程序，因此它在某些模块中对Log4j具有可选的编译时依赖性。

要使Log4j 1.2使用默认的JCL依赖项（commons-logging），您需要做的就是将Log4j放在类路径上，并为其提供配置文件（类路径根目录中的log4j.properties或log4j.xml）。 因此，对于Maven用户，这是您的依赖声明：

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>4.3.21.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```

这是一个用于登录控制台的log4j.properties示例：

```
log4j.rootCategory=INFO, stdout

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %t %c{2}:%L - %m%n

log4j.category.org.springframework.beans.factory=DEBUG
```

要将Log4j 2.x与JCL一起使用，您需要做的就是将Log4j放在类路径上，并为其提供配置文件（log4j2.xml，log4j2.properties或其他支持的配置格式）。 对于Maven用户，所需的最小依赖项是：

```
<dependencies>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.6.2</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-jcl</artifactId>
        <version>2.6.2</version>
    </dependency>
</dependencies>
```

如果您还希望启用SLF4J以委派给Log4j，例如 对于默认使用SLF4J的其他库，还需要以下依赖项：

```
<dependencies>
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.6.2</version>
  </dependency>
</dependencies>
```

以下是用于记录到控制台的log4j2.xml示例：

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Logger name="org.springframework.beans.factory" level="DEBUG"/>
    <Root level="error">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```

##### 避免使用Commons Logging

遗憾的是，标准的commons-logging API中的运行时发现算法虽然对最终用户来说很方便，但却存在问题。 如果您想避免使用JCL的标准查找，基本上有两种方法可以将其关闭：

1. 从spring-core模块中排除依赖关系（因为它是唯一明确依赖于commons-logging的模块）
2. 依赖于一个特殊的commons-logging依赖项，用一个空jar替换库（更多细节可以在SLF4J FAQ中找到）

要排除commons-logging，请将以下内容添加到dependencyManagement部分：

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>4.3.21.RELEASE</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

现在这个应用程序已经崩溃，因为类路径上没有JCL API的实现，所以为了修复它，必须提供一个新的。 在下一节中，我们将向您展示如何使用SLF4J提供JCL的替代实现。

##### 将SLF4J与Log4j或Logback一起使用

Java的简单日志外观（SLF4J）是Spring常用的其他库使用的流行API。它通常与Logback一起使用，后者是SLF4J API的本机实现。

SLF4J提供了对许多常见日志框架（包括Log4j）的绑定，它也反过来了：其他日志框架与其自身之间的桥梁。因此，要在Spring中使用SLF4J，您需要使用SLF4J-JCL桥替换commons-logging依赖项。完成后，从Spring中记录调用将转换为对SLF4J API的日志记录调用。因此，如果应用程序中的其他库使用该API，那么您只需一个地方来配置和管理日志记录。

一个常见的选择可能是将Spring桥接到SLF4J，然后提供从SLF4J到Log4j的显式绑定。您需要提供多个依赖项（并排除现有的commons-logging）：JCL桥，与Log4j绑定的SLF4j以及Log4j提供程序本身。在Maven，你会这样做：

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>4.3.21.RELEASE</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
        <version>1.7.21</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.21</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```

SLF4J用户中使用较少步骤并生成较少依赖关系的更常见选择是直接绑定到Logback。 这消除了额外的绑定步骤，因为Logback直接实现了SLF4J，所以你只需要依赖两个库，即jcl-over-slf4j和logback）：

```
<dependencies>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
        <version>1.7.21</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.1.7</version>
    </dependency>
</dependencies>
```

##### 使用 JUL \(java.util.logging\)

默认情况下，Commons Logging将委托给java.util.logging，前提是在类路径中没有检测到Log4j。 所以没有特殊的依赖设置：只需在没有外部依赖的情况下使用Spring将日志输出到java.util.logging，无论是在独立应用程序中（在JDK级别使用自定义或默认JUL设置）还是使用应用程序服务器 日志系统（及其系统范围的JUL设置）。

##### WebSphere的Commons Logging

Spring应用程序可以在一个容器上运行，该容器本身提供JCL的实现，例如， IBM的WebSphere Application Server（WAS）。这不会导致问题本身，但会导致需要理解的两种不同情况：

在“父优先”ClassLoader委托模型（WAS上的默认模式）中，应用程序将始终获取服务器提供的Commons Logging版本，委托给WAS日志记录子系统（实际上是基于JUL）。 JCL的应用程序提供的变体，无论是标准的Commons Logging还是JCL-over-SLF4J桥，都将被有效地忽略，以及任何本地包含的日志提供程序。

使用“父级最后”委托模型（常规Servlet容器中的默认值，但WAS上的显式配置选项），将选择应用程序提供的Commons Logging变体，使您能够设置本地包含的日志提供程序，例如， Log4j或Logback，在您的应用程序中。如果没有本地日志提供程序，则默认情况下，常规Commons Logging将委派给JUL，有效地记录到WebSphere的日志记录子系统，就像在“父第一”方案中一样。

总而言之，我们建议在“父最后”模型中部署Spring应用程序，因为它自然允许本地提供程序以及服务器的日志子系统。


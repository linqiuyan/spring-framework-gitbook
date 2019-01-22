# 15 集成测试

## 15.1 概览

能够执行某些集成测试非常重要，无需部署到应用程序服务器或连接到其他企业基础结构。 这将使您能够测试以下内容：

* Spring IoC容器上下文的正确连接。
* 使用JDBC或ORM工具进行数据访问。这将包括诸如SQL语句的正确性，Hibernate查询，JPA实体映射等。

Spring Framework为spring-test模块中的集成测试提供了一流的支持。实际JAR文件的名称可能包含发行版本，也可能是长的org.springframework.test形式，具体取决于您从何处获取（有关说明，请参阅依赖关系管理部分）。该库包含org.springframework.test包，其中包含用于使用Spring容器进行集成测试的有价值的类。此测试不依赖于应用程序服务器或其他部署环境。这些测试比单元测试运行得慢，但比依赖部署到应用服务器的等效Selenium测试或远程测试快得多。

在Spring 2.5及更高版本中，单元和集成测试支持以注释驱动的Spring TestContext Framework的形式提供。 TestContext框架与使用中的实际测试框架无关，因此允许在各种环境中进行测试，包括JUnit，TestNG等。

## 15.2 集成测试的目标

Spring的集成测试支持有以下主要目标：

* 在测试执行之间管理Spring IoC容器缓存。
* 提供测试夹具实例的依赖注入。
* 提供适合集成测试的事务管理。
* 提供特定于Spring的基类，帮助开发人员编写集成测试。
* 接下来的几节将介绍每个目标，并提供实现和配置详细信息的链接。

### 15.2.1 上下文管理和缓存

Spring TestContext Framework提供Spring ApplicationContexts和WebApplicationContexts的一致加载以及这些上下文的缓存。支持缓存加载的上下文非常重要，因为启动时间可能会成为一个问题 - 不是因为Spring本身的开销，而是因为Spring容器实例化的对象需要时间来实例化。例如，具有50到100个Hibernate映射文件的项目可能需要10到20秒来加载映射文件，并且在每个测试夹具中运行每个测试之前产生该成本会导致整体测试运行速度变慢，从而降低开发人员的工作效率。

测试类通常声明XML或Groovy配置元数据的资源位置数组（通常在类路径中）或用于配置应用程序的带注释类的数组。这些位置或类与web.xml或生产部署的其他配置文件中指定的位置或类相同或类似。

默认情况下，一旦加载，配置的ApplicationContext将重复用于每个测试。因此，每个测试套件仅产生一次设置成本，并且后续测试执行要快得多。在此上下文中，术语测试套件意味着所有测试都在同一JVM中运行 - 例如，所有测试都是针对给定项目或模块从Ant，Maven或Gradle构建的。在不太可能的情况下，测试会破坏应用程序上下文并需要重新加载 - 例如，通过修改bean定义或应用程序对象的状态 - 可以将TestContext框架配置为在执行下一个之前重新加载配置并重建应用程序上下文测试。

请参见第15.5.4节“上下文管理”和使用TestContext框架的“上下文缓存”一节。

### 15.2.2 依赖注入测试夹具

当TestContext框架加载应用程序上下文时，它可以选择通过依赖注入配置测试类的实例。这为使用应用程序上下文中的预配置bean设置测试夹具提供了一种便捷的机制。这里的一个重要好处是您可以在各种测试场景中重用应用程序上下文（例如，用于配置Spring管理的对象图，事务代理，DataSource等），从而避免为单个测试用例复制复杂的测试夹具设置。

作为示例，考虑我们有一个类HibernateTitleRepository的场景，该类实现Title域实体的数据访问逻辑。我们想编写测试以下方面的集成测试：

* Spring配置：基本上，与HibernateTitleRepository bean的配置相关的一切是否正确和存在？
* Hibernate映射文件配置：是否正确映射了所有内容，并且是否具有正确的延迟加载设置？
* HibernateTitleRepository的逻辑：此类的已配置实例是否按预期执行？

请参阅使用TestContext框架对测试夹具进行依赖注入。

### 15.2.3 交易管理

访问真实数据库的测试中的一个常见问题是它们对持久性存储的状态的影响。即使您使用的是开发数据库，​​对状态的更改也可能会影响将来的测试。此外，许多操作（例如插入或修改持久数据）无法在事务外执行（或验证）。

TestContext框架解决了这个问题。默认情况下，框架将为每个测试创建并回滚事务。您只需编写可以假定存在事务的代码。如果在测试中调用事务代理对象，则根据其配置的事务语义，它们将正常运行。此外，如果测试方法在为测试管理的事务中运行时删除所选表的内容，则事务将默认回滚，并且数据库将返回到执行测试之前的状态。通过在测试的应用程序上下文中定义的PlatformTransactionManager bean为测试提供事务支持。

如果您希望事务提交 - 异常，但在您希望特定测试填充或修改数据库时偶尔有用 - 可以指示TestContext框架使事务提交而不是通过@Commit注释回滚。

使用TestContext框架查看事务管理。

### 15.2.4 支持集成测试类

Spring TestContext Framework提供了几个抽象支持类，可以简化集成测试的编写。 这些基本测试类提供了定义良好的钩子到测试框架中，以及方便的实例变量和方法，使您可以访问：

ApplicationContext，用于执行显式bean查找或测试整个上下文的状态。

JdbcTemplate，用于执行SQL语句以查询数据库。 此类查询可用于在执行与数据库相关的应用程序代码之前和之后确认数据库状态，Spring确保此类查询在与应用程序代码相同的事务范围内运行。 与ORM工具结合使用时，请务必避免误报。

此外，您可能希望使用特定于项目的实例变量和方法创建自己的自定义应用程序范围的超类。

请参阅TestContext框架的支持类。

## 15.3  JDBC测试支持

org.springframework.test.jdbc包中包含JdbcTestUtils，它是JDBC相关实用程序函数的集合，旨在简化标准数据库测试方案。具体来说，JdbcTestUtils提供以下静态实用程序方法。

* countRowsInTable（..）：计算给定表中的行数
* countRowsInTableWhere（..）：使用提供的WHERE子句计算给定表中的行数
* deleteFromTables（..）：删除指定表中的所有行
* deleteFromTableWhere（..）：使用提供的WHERE子句从给定表中删除行
* dropTables（..）：删除指定的表

请注意，AbstractTransactionalJUnit4SpringContextTests和AbstractTransactionalTestNGSpringContextTests提供了方便的方法，它们委托JdbcTestUtils中的上述方法。

spring-jdbc模块支持配置和启动嵌入式数据库，该数据库可用于与数据库交互的集成测试。有关详细信息，请参见第19.8节“嵌入式数据库支持”和第19.8.5节“使用嵌入式数据库测试数据访问逻辑”。



15.5  Spring TestContext框架

15.5.1 关键的抽象

的TestContext

TestContextManager来

TestExecutionListener的

上下文加载器

15.5.2 引导TestContext框架

15.5.3  TestExecutionListener配置

注册自定义TestExecutionListeners

自动发现默认的TestExecutionListeners

订购TestExecutionListeners

合并TestExecutionListeners

15.5.4 上下文管理

使用XML资源配置上下文

使用Groovy脚本配置上下文

带注释类的上下文配置

混合XML，Groovy脚本和带注释的类

上下文配置与上下文初始化器

上下文配置继承

环境配置与环境配置文件

具有测试属性源的上下文配置

加载WebApplicationContext

上下文缓存

上下文层次结构

15.5.5 测试夹具的依赖注入

15.5.6 测试请求和会话范围的bean

15.5.7 交易管理

测试管理的事务

启用和禁用事务

事务回滚和提交行为

程序化交易管理

在事务之外执行代码

配置事务管理器

演示所有与事务相关的注释

15.5.8 执行SQL脚本

以编程方式执行SQL脚本

使用@Sql以声明方式执行SQL脚本

15.5.9 TestContext Framework支持类

Spring JUnit 4 Runner

Spring JUnit 4规则

JUnit 4支持类

JUnit 5支持

TestNG支持类

15.6  Spring MVC测试框架

15.6.1 服务器端测试

静态进口

设置选择

执行请求

定义期望

过滤注册

容器外和端到端集成测试之间的差异

进一步的服务器端测试示例

15.6.2  HtmlUnit集成

为何选择HtmlUnit？

MockMvc和HtmlUnit

MockMvc和WebDriver

MockMvc和Geb

15.6.3 客户端REST测试

静态进口

客户端REST测试的更多示例

15.7  PetClinic示例


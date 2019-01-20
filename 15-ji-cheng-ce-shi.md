15.集成测试

15.1。概观

15.2。集成测试的目标

15.2.1。上下文管理和缓存

15.2.2。依赖注入测试夹具

15.2.3。交易管理

15.2.4。支持集成测试类

15.3。 JDBC测试支持

15.4。注释

15.4.1。弹簧测试注释

@BootstrapWith

@ContextConfiguration

@WebAppConfiguration

@ContextHierarchy

@ActiveProfiles

@TestPropertySource

@DirtiesContext

@TestExecutionListeners

@承诺

@Rollback

@BeforeTransaction

@AfterTransaction

@sql

@SqlConfig

@SqlGroup

15.4.2。标准注释支持

15.4.3。 Spring JUnit 4测试注释

同时@IfProfileValue

@ProfileValueSourceConfiguration

@Timed

@重复

15.4.4。测试的元注释支持

15.5。 Spring TestContext框架

15.5.1。关键的抽象

的TestContext

TestContextManager来

TestExecutionListener的

上下文加载器

15.5.2。引导TestContext框架

15.5.3。 TestExecutionListener配置

注册自定义TestExecutionListeners

自动发现默认的TestExecutionListeners

订购TestExecutionListeners

合并TestExecutionListeners

15.5.4。上下文管理

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

15.5.5。测试夹具的依赖注入

15.5.6。测试请求和会话范围的bean

15.5.7。交易管理

测试管理的事务

启用和禁用事务

事务回滚和提交行为

程序化交易管理

在事务之外执行代码

配置事务管理器

演示所有与事务相关的注释

15.5.8。执行SQL脚本

以编程方式执行SQL脚本

使用@Sql以声明方式执行SQL脚本

15.5.9。 TestContext Framework支持类

Spring JUnit 4 Runner

Spring JUnit 4规则

JUnit 4支持类

JUnit 5支持

TestNG支持类

15.6。 Spring MVC测试框架

15.6.1。服务器端测试

静态进口

设置选择

执行请求

定义期望

过滤注册

容器外和端到端集成测试之间的差异

进一步的服务器端测试示例

15.6.2。 HtmlUnit集成

为何选择HtmlUnit？

MockMvc和HtmlUnit

MockMvc和WebDriver

MockMvc和Geb

15.6.3。客户端REST测试

静态进口

客户端REST测试的更多示例

15.7。 PetClinic示例


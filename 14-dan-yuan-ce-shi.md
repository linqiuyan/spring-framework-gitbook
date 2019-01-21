# 14 单元测试

与传统的Java EE开发相比，依赖注入应该使您的代码对容器的依赖性降低。构成应用程序的POJO应该可以在JUnit或TestNG测试中测试，对象只需使用new运算符实例化，不需要Spring或任何其他容器。您可以使用模拟对象（与其他有价值的测试技术结合使用）来单独测试代码。如果您遵循Spring的体系结构建议，那么代码库的干净分层和组件化将有助于更轻松地进行单元测试。例如，您可以通过存根或模拟DAO或Repository接口来测试服务层对象，而无需在运行单元测试时访问持久数据。

真正的单元测试通常运行得非常快，因为没有要设置的运行时基础结构。强调真正的单元测试作为开发方法的一部分将提高您的工作效率。您可能不需要测试章节的这一部分来帮助您为基于IoC的应用程序编写有效的单元测试。但是，对于某些单元测试场景，Spring Framework提供了以下模拟对象和测试支持类。

## 14.1 Mock Objects

### 14.1.1  Environment

org.springframework.mock.env包中包含Environment和PropertySource抽象的模拟实现（请参见第7.13.1节“Bean定义概要”和第7.13.2节“PropertySource抽象”）。 MockEnvironment和MockPropertySource对于开发依赖于特定于环境的属性的代码的容器外测试非常有用。

### 14.1.2 JNDI

org.springframework.mock.jndi包中包含JNDI SPI的实现，您可以使用它为测试套件或独立应用程序设置简单的JNDI环境。 例如，如果JDBC DataSources在测试代码中绑定到与Java EE容器中相同的JNDI名称，则可以在测试方案中重用应用程序代码和配置而无需修改。

### 14.1.3  Servlet API

org.springframework.mock.web包中包含一组全面的Servlet API模拟对象，可用于测试Web上下文，控制器和过滤器。 这些模拟对象的目标是使用Spring的Web MVC框架，并且通常比动态模拟对象（如EasyMock）或其他Servlet API模拟对象（如MockObjects）更方便使用。 从Spring Framework 4.0开始，org.springframework.mock.web包中的模拟集基于Servlet 3.0 API。

有关Spring MVC和REST控制器的完整集成测试以及Spring MVC的WebApplicationContext配置，请参阅Spring MVC测试框架。

### 14.1.4 Portlet API

org.springframework.mock.web.portlet包中包含一组Portlet API模拟对象，旨在使用Spring的Portlet MVC框架。

## 14.2 单元测试支持类

### 14.2.1 一般测试工具

org.springframework.test.util包中包含几个用于单元和集成测试的通用实用程序。

ReflectionTestUtils是基于反射的实用程序方法的集合。 开发人员在测试需要更改常量值，设置非公共字段，调用非公共setter方法或在测试涉及使用的应用程序代码时调用非公共配置或生命周期回调方法的场景中使用这些方法 例如以下情况。

* ORM框架，如JPA和Hibernate，它们容忍私有或受保护的字段访问，而不是域实体中属性的公共setter方法。
* Spring支持@Autowired，@ Inject和@Resource等注释，它为私有或受保护字段，setter方法和配置方法提供依赖注入。
* 使用@PostConstruct和@PreDestroy等注释进行生命周期回调方法。

AopTestUtils是AOP相关实用程序方法的集合。 这些方法可用于获取对隐藏在一个或多个Spring代理后面的底层目标对象的引用。 例如，如果您使用像EasyMock或Mockito这样的库将bean配置为动态模拟，并且模拟包装在Spring代理中，则可能需要直接访问底层模拟，以便为其配置期望并执行验证。 对于Spring的核心AOP实用程序，请参阅AopUtils和AopProxyUtils。

### 14.2.2  Spring MVC

org.springframework.test.web包中包含ModelAndViewAssert，您可以将其与JUnit，TestNG或任何其他测试框架结合使用，以处理Spring MVC ModelAndView对象的单元测试。

\[提示\]

要将Spring MVC控制器单元测试为POJO，请使用ModelAndViewAssert与Spring的Servlet API模拟中的MockHttpServletRequest，MockHttpSession等结合使用。 有关Spring MVC和REST控制器的完整集成测试以及Spring MVC的WebApplicationContext配置，请使用Spring MVC测试框架。


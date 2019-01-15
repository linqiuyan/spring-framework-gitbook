# 7 Ioc容器

## 7.1 Spring IoC容器和bean简介

本章介绍了Spring Framework实现的控制反转（IoC）原理。 IoC也称为依赖注入（DI）。 这是一个过程，通过这个过程，对象定义它们的依赖关系，即它们使用的其他对象，只能通过构造函数参数，工厂方法的参数，或者在构造或从工厂方法返回后在对象实例上设置的属性。 然后容器在创建bean时注入这些依赖项。 这个过程基本上是反向的，因此名称Inversion of Control（IoC），bean本身通过使用类的直接构造或诸如Service Locator模式之类的机制来控制其依赖关系的实例化或位置。

org.springframework.beans和org.springframework.context包是Spring Framework的IoC容器的基础。 BeanFactory接口提供了一种能够管理任何类型对象的高级配置机制。 ApplicationContext是BeanFactory的子接口。 它增加了与Spring的AOP功能的更容易的集成; 消息资源处理（用于国际化），事件发布; 和特定于应用程序层的上下文，例如WebApplicationContext，用于Web应用程序。

简而言之，BeanFactory提供配置框架和基本功能，ApplicationContext添加了更多特定于企业的功能。 ApplicationContext是BeanFactory的完整超集，在本章中专门用于Spring的IoC容器的描述。 有关使用BeanFactory而不是ApplicationContext的更多信息，请参见第7.16节“BeanFactory”。

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。 bean是一个由Spring IoC容器实例化，组装和管理的对象。 否则，bean只是应用程序中众多对象之一。 Bean及其之间的依赖关系反映在容器使用的配置元数据中。




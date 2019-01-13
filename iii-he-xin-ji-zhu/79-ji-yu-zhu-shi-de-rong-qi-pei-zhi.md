## 7.9 基于注释的容器配置

---

注释是否比配置Spring的XML更好？

基于注释的配置的引入引发了这种方法是否比XML更“好”的问题。简短的答案取决于它。答案很长，每种方法都有其优点和缺点，通常由开发人员决定哪种策略更适合他们。由于它们的定义方式，注释在其声明中提供了大量上下文，从而导致更短更简洁的配置。但是，XML擅长在不触及源代码或重新编译它们的情况下连接组件。一些开发人员更喜欢将布线靠近源，而另一些开发人员则认为注释类不再是POJO，而且配置变得分散且难以控制。

无论选择如何，Spring都可以兼顾两种风格，甚至可以将它们混合在一起。值得指出的是，通过其JavaConfig选项，Spring允许以非侵入方式使用注释，而无需触及目标组件源代码，并且在工具方面，Spring Tool Suite支持所有配置样式。

---

---

基于注释的配置提供了XML设置的替代方案，该配置依赖于字节码元数据来连接组件而不是角括号声明。开发人员不是使用XML来描述bean连接，而是通过在相关的类，方法或字段声明上使用注释将配置移动到组件类本身。正如在“示例：RequiredAnnotationBeanPostProcessor”一节中所提到的，将BeanPostProcessor与注释结合使用是扩展Spring IoC容器的常用方法。例如，Spring 2.0引入了使用@Required注释强制执行所需属性的可能性。 Spring 2.5使得有可能采用相同的通用方法来驱动Spring的依赖注入。从本质上讲，@ Autowired注释提供了与第7.4.5节“自动装配协作者”中所述相同的功能，但具有更细粒度的控制和更广泛的适用性。 Spring 2.5还增加了对JSR-250注释的支持，例如@ PostConstruct和@PreDestroy。 Spring 3.0增加了对javax.inject包中包含的JSR-330（Java的依赖注入）注释的支持，例如@Inject和@Named。有关这些注释的详细信息，请参阅相关章节。

注释注入在XML注入之前执行，因此后一种配置将覆盖通过两种方法连接的属性的前者。

与往常一样，您可以将它们注册为单独的bean定义，但也可以通过在基于XML的Spring配置中包含以下标记来隐式注册它们（请注意包含上下文命名空间）：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

（隐式注册的后处理器包括AutowiredAnnotationBeanPostProcessor，CommonAnnotationBeanPostProcessor，PersistenceAnnotationBeanPostProcessor，以及前面提到的RequiredAnnotationBeanPostProcessor。）

&lt;context:annotation-config /&gt;仅查找在定义它的同一应用程序上下文中的bean上的注释。 这意味着，如果将&lt;context:annotation-config/&gt;放在DispatcherServlet的WebApplicationContext中，它只检查控制器中的@Autowired bean，而不检查您的服务。 有关更多信息，请参见第22.2节“DispatcherServlet”。

### 7.9.1 @Required

@Required注释适用于bean属性setter方法，如下例所示：



### 7.9.2 @Autowired










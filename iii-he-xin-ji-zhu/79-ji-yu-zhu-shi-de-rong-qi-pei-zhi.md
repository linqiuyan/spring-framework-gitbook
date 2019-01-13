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

### 7.9.1 @Required

@Required注释适用于bean属性setter方法，如下例所示：

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

此注释仅指示必须在配置时通过bean定义中的显式属性值或通过自动装配填充受影响的bean属性。 如果尚未填充受影响的bean属性，容器将引发异常; 这允许急切和明确的失败，以后避免NullPointerExceptions等。 仍然建议您将断言放入bean类本身，例如，放入init方法。 即使您在容器外使用类，这样做也会强制执行那些必需的引用和值。

### 7.9.2 @Autowired

在下面的示例中，可以使用JSR 330的@Inject注释代替Spring的@Autowired注释。 有关详细信息，请参见此处。

您可以将@Autowired批注应用于构造函数：

```
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

从Spring Framework 4.3开始，如果目标bean只定义了一个开头的构造函数，则不再需要在这样的构造函数上使用@Autowired注释。 但是，如果有几个构造函数可用，则必须注释至少一个构造函数，以教导容器使用哪一个。

正如所料，您还可以将@Autowired注释应用于“传统”setter方法：

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

您还可以将注释应用于具有任意名称和/或多个参数的方法：

```
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

您也可以将@Autowired应用于字段，甚至可以将它与构造函数混合使用：

```
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

确保您的目标组件（例如MovieCatalog，CustomerPreferenceDao）始终按照您用于@Autowired注释注入点的类型进行声明。 否则，由于在运行时未找到类型匹配，注入可能会失败。

对于通过类路径扫描找到的XML定义的bean或组件类，容器通常预先知道具体类型。 但是，对于@Bean工厂方法，您需要确保声明的返回类型具有足够的表达能力。 对于实现多个接口的组件或可能由其实现类型引用的组件，请考虑在工厂方法上声明最具体的返回类型（至少与引用bean的注入点所要求的具体相同）。

通过将注释添加到期望该类型数组的字段或方法，还可以从ApplicationContext提供特定类型的所有bean：

```
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```

这同样适用于类型集合：

```
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

如果希望按特定顺序对数组或列表中的项进行排序，则目标bean可以实现org.springframework.core.Ordered接口或使用@Order或标准@Priority批注。 否则，它们的顺序将遵循容器中相应目标bean定义的注册顺序。

@Order注释可以在目标类级别声明，也可以在@Bean方法上声明，可能是每个bean定义非常独立（如果多个定义具有相同的bean类）。 @Order值可能影响注入点的优先级，但请注意它们不会影响单例启动顺序，这是由依赖关系和@DependsOn声明确定的正交关注点。

请注意，标准的javax.annotation.Priority注释在@Bean级别不可用，因为它无法在方法上声明。 它的语义可以通过@Order值与每个类型的单个bean上的@Primary一起建模。

只要预期的键类型是String，即使是类型化的地图也可以自动装配。 Map值将包含所需类型的所有bean，并且键将包含相应的bean名称：

```
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

默认情况下，只要零候选bean可用，自动装配就会失败; 默认行为是将带注释的方法，构造函数和字段视为指示所需的依赖项。 可以更改此行为，如下所示。

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

每个类只能标记一个带注释的构造函数，但可以注释多个非必需的构造函数。 在这种情况下，每个都被认为是候选者之一，Spring使用最贪婪的构造函数，其依赖性可以得到满足，即具有最多参数的构造函数。

建议使用@Autowired的必需属性而不是@Required注释。 required属性表示该属性不是自动装配所必需的，如果无法自动装配该属性，则会忽略该属性。 另一方面，@ Required更强大，因为它强制执行由容器支持的任何方式设置的属性。 如果未注入任何值，则会引发相应的异常。

或者，您可以通过Java 8的java.util.Optional表达特定依赖项的非必需特性：

```
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

您还可以将@Autowired用于众所周知的可解析依赖项的接口：BeanFactory，ApplicationContext，Environment，ResourceLoader，ApplicationEventPublisher和MessageSource。 这些接口及其扩展接口（如ConfigurableApplicationContext或ResourcePatternResolver）将自动解析，无需特殊设置。

```
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

@Autowired，@Inject，@Resource和@Value注释由Spring BeanPostProcessor实现处理，这反过来意味着您不能在自己的BeanPostProcessor或BeanFactoryPostProcessor类型（如果有）中应用这些注释。 必须通过XML或使用Spring @Bean方法显式地“连接”这些类型。

### 7.9.3 使用@Primary微调基于注释的自动装配

### 7.9.4 使用限定符微调基于注释的自动装配

### 7.9.5 使用泛型作为自动装配限定符

### 7.9.6  CustomAutowireConfigurer上

### 7.9.7  @Resource

### 7.9.8  @PostConstruct和@PreDestroy




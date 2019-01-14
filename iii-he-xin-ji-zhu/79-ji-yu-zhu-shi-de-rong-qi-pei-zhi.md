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

由于按类型自动装配可能会导致多个候选人，因此通常需要对选择过程有更多控制权。 实现这一目标的一种方法是使用Spring的@Primary注释。 @Primary表示当多个bean可以自动装配到单值依赖项时，应该优先选择特定的bean。 如果候选者中只存在一个“主”bean，则它将是自动装配的值。

假设我们有以下配置将firstMovieCatalog定义为主要MovieCatalog。

```
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

通过这样的配置，以下MovieRecommender将与firstMovieCatalog一起自动装配。

```
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

相应的bean定义如下所示。

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

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

### 7.9.4 使用限定符微调基于注释的自动装配（Fine-tuning annotation-based autowiring with qualifiers）

@Primary是一种有效的方法，可以在确定一个主要候选者时使用具有多个实例的类型进行自动装配。 当需要更多地控制选择过程时，可以使用Spring的@Qualifier注释。 您可以将限定符值与特定参数相关联，缩小类型匹配集，以便为每个参数选择特定的bean。 在最简单的情况下，这可以是一个简单的描述性值：

```
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```

@Qualifier注释也可以在各个构造函数参数或方法参数上指定：

```
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main")MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

相应的bean定义如下所示。 具有限定符值“main”的bean与使用相同值限定的构造函数参数连接。

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

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/>

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

对于回退匹配，bean名称被视为默认限定符值。 因此，您可以使用id“main”而不是嵌套的限定符元素来定义bean，从而得到相同的匹配结果。 但是，虽然您可以使用此约定来按名称引用特定bean，但@Autowired基本上是关于具有可选语义限定符的类型驱动注入。 这意味着即使使用bean名称回退，限定符值在类型匹配集中也总是具有缩小的语义; 它们在语义上不表示对唯一bean id的引用。 好的限定符值是“main”或“EMEA”或“persistent”，表示独立于bean id的特定组件的特征，在匿名bean定义的情况下可以自动生成，如上例中的那个。

限定符也适用于类型化集合，如上所述，例如，Set &lt;MovieCatalog&gt;。 在这种情况下，根据声明的限定符的所有匹配bean都作为集合注入。 这意味着限定符不必是唯一的; 它们只是简单地构成过滤标准。 例如，您可以使用相同的限定符值“action”定义多个MovieCatalog bean，所有这些bean都将注入到使用@Qualifier（“action”）注释的Set &lt;MovieCatalog&gt;中。

---

_在类型匹配候选项中，根据目标bean名称选择限定符值，甚至不需要在注入点处使用@Qualifier注释。 如果没有其他分辨率指示符（例如限定符或主要标记），对于非唯一依赖性情况，Spring将使注入点名称（即字段名称或参数名称）与目标bean名称匹配，并选择相同的 -  如果有的话，就是候选人。_

也就是说，如果您打算按名称表达注释驱动的注入，请不要主要使用@Autowired，即使能够在类型匹配候选者中通过bean名称进行选择。 相反，使用JSR-250 @Resource注释，该注释在语义上定义为通过其唯一名称标识特定目标组件，声明的类型与匹配过程无关。 @Autowired具有相当不同的语义：在按类型选择候选bean之后，将仅在那些类型选择的候选者中考虑指定的字符串限定符值，例如， 将“account”限定符与标记有相同限定符标签的bean匹配。

对于自身定义为集合/映射或数组类型的bean，@ Resource是一个很好的解决方案，通过唯一名称引用特定的集合或数组bean。 也就是说，从4.3开始，只要元素类型信息保存在@Bean返回类型签名或集合继承层次结构中，集合/映射和数组类型也可以通过Spring的@Autowired类型匹配算法进行匹配。 在这种情况下，限定符值可用于在相同类型的集合中进行选择，如上一段所述。

从4.3开始，@ Autowired还考虑了自我引用注入，即引用回到当前注入的bean。 请注意，自我注射是一种后备; 对其他组件的常规依赖性始终具有优先权。 从这个意义上讲，自我引用并不参与常规的候选人选择，因此特别是不是主要的; 相反，它们总是最低优先级。 在实践中，仅使用自引用作为最后的手段，例如 通过bean的事务代理调用同一实例上的其他方法：在这种情况下，考虑将受影响的方法分解为单独的委托bean。 或者，使用@Resource，它可以通过其唯一名称获取代理回到当前bean。

@Autowired适用于字段，构造函数和多参数方法，允许在参数级别缩小限定符注释。 相比之下，@ Resource仅支持具有单个参数的字段和bean属性setter方法。 因此，如果您的注入目标是构造函数或多参数方法，请坚持使用限定符。

---

您可以创建自己的自定义限定符注释。 只需定义注释并在定义中提供@Qualifier注释：

```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```

然后，您可以在自动装配的字段和参数上提供自定义限定符：

```
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;

    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...
}
```

接下来，提供候选bean定义的信息。 您可以将&lt;qualifier /&gt;标记添加为&lt;bean /&gt;标记的子元素，然后指定与自定义限定符注释匹配的类型和值。 该类型与注释的完全限定类名匹配。 或者，为方便起见，如果不存在冲突名称的风险，您可以使用短类名。 以下示例演示了这两种方法。

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

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="example.Genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

在第7.10节“类路径扫描和托管组件”中，您将看到基于注释的替代方法，用于在XML中提供限定符元数据。 具体来说，请参见第7.10.8节“使用注释提供限定符元数据”。

在某些情况下，使用没有值的注释可能就足够了。 当注释用于更通用的目的并且可以跨多种不同类型的依赖项应用时，这可能很有用。 例如，您可以提供在没有Internet连接时将搜索的脱机目录。 首先定义简单注释：

```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {

}
```

然后将注释添加到要自动装配的字段或属性中：

```
public class MovieRecommender {

    @Autowired
    @Offline
    private MovieCatalog offlineCatalog;

    // ...
}
```

现在bean定义只需要一个限定符类型：

```
<bean class="example.SimpleMovieCatalog">
    <qualifier type="Offline"/>
    <!-- inject any dependencies required by this bean -->
</bean>
```

您还可以定义除简单值属性之外或代替简单值属性接受命名属性的自定义限定符注释。 如果随后在要自动装配的字段或参数上指定了多个属性值，则bean定义必须匹配所有此类属性值才能被视为自动装配候选。 例如，请考虑以下注释定义：

```
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();
}
```

在这种情况下，Format是一个枚举：

```
public enum Format {
    VHS, DVD, BLURAY
}
```

要自动装配的字段使用自定义限定符进行注释，并包含两个属性的值：genre和format。

```
public class MovieRecommender {

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Action")
    private MovieCatalog actionVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Comedy")
    private MovieCatalog comedyVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.DVD, genre="Action")
    private MovieCatalog actionDvdCatalog;

    @Autowired
    @MovieQualifier(format=Format.BLURAY, genre="Comedy")
    private MovieCatalog comedyBluRayCatalog;

    // ...
}
```

最后，bean定义应包含匹配的限定符值。 此示例还演示可以使用bean元属性而不是&lt;qualifier /&gt;子元素。 如果可用，&lt;qualifier /&gt;及其属性优先，但如果不存在此类限定符，则自动装配机制将回退到&lt;meta /&gt;标记内提供的值，如以下示例中的最后两个bean定义。

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

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Action"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Comedy"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="DVD"/>
        <meta key="genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="BLURAY"/>
        <meta key="genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

</beans>
```

### 7.9.5 使用泛型作为自动装配限定符

除了@Qualifier注释之外，还可以使用Java泛型类型作为隐式的限定形式。 例如，假设您具有以下配置：

```
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
```

假设上面的bean实现了一个通用接口，即Store &lt;String&gt;和Store &lt;Integer&gt;，你可以@Autowire Store接口，泛型将被用作限定符：

```
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
```

通用限定符也适用于自动装配列表，地图和数组：

```
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```

### 7.9.6  CustomAutowireConfigurer上

CustomAutowireConfigurer是一个BeanFactoryPostProcessor，它允许您注册自己的自定义限定符注释类型，即使它们没有使用Spring的@Qualifier注释进行注释。

```
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```

AutowireCandidateResolver通过以下方式确定autowire候选者：

* 每个bean定义的autowire-candidate值
* &lt;beans /&gt;元素上可用的任何default-autowire-candidates模式
* @Qualifier注释的存在以及使用CustomAutowireConfigurer注册的任何自定义注释

当多个bean有资格作为autowire候选者时，“primary”的确定如下：如果候选者中只有一个bean定义的主要属性设置为true，则将选择它。

### 7.9.7  @Resource

Spring还支持在字段或bean属性setter方法上使用JSR-250 @Resource注释进行注入。 这是Java EE 5和6中的常见模式，例如在JSF 1.2托管bean或JAX-WS 2.0端点中。 Spring也支持Spring管理对象的这种模式。

@Resource采用name属性，默认情况下Spring将该值解释为要注入的bean名称。 换句话说，它遵循按名称语义，如本例所示：

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder")
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

如果未明确指定名称，则默认名称是从字段名称或setter方法派生的。 如果是字段，则采用字段名称; 在setter方法的情况下，它采用bean属性名称。 所以下面的例子将把名为“movieFinder”的bean注入其setter方法：

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

_随注释提供的名称由ApplicationContext解析为bean名称，CommonAnnotationBeanPostProcessor可以识别该名称。 如果您显式配置Spring的SimpleJndiBeanFactory，则可以通过JNDI解析名称。 但是，建议您依赖于默认行为，只需使用Spring的JNDI查找功能来保持间接级别。_

在没有指定显式名称且类似于@Autowired的@Resource用法的独家情况下，@ Resource找到主要类型匹配而不是特定的命名bean，并解析众所周知的可解析依赖项：BeanFactory，ApplicationContext，ResourceLoader，ApplicationEventPublisher， 和MessageSource接口。

因此，在以下示例中，customerPreferenceDao字段首先查找名为customerPreferenceDao的bean，然后返回CustomerPreferenceDao类型的主类型匹配。 基于已知的可解析依赖类型ApplicationContext注入“context”字段。

```
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

### 7.9.8  @PostConstruct和@PreDestroy

CommonAnnotationBeanPostProcessor不仅识别@Resource注释，还识别JSR-250生命周期注释。 在Spring 2.5中引入，对这些注释的支持提供了初始化回调和销毁回调中描述的另一种替代方法。 如果CommonAnnotationBeanPostProcessor在Spring ApplicationContext中注册，则在生命周期的同一点调用带有这些注释之一的方法，作为相应的Spring生命周期接口方法或显式声明的回调方法。 在下面的示例中，缓存将在初始化时预先填充，并在销毁时清除。

```
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```




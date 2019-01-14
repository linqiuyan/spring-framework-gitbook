## 7.10 类路径扫描和托管组件（Classpath scanning and managed components）

本章中的大多数示例都使用XML来指定在Spring容器中生成每个BeanDefinition的配置元数据。 上一节（第7.9节“基于注释的容器配置”）演示了如何通过源级注释提供大量配置元数据。 但是，即使在这些示例中，“基本”bean定义也在XML文件中明确定义，而注释仅驱动依赖项注入。 本节介绍通过扫描类路径隐式检测候选组件的选项。 候选组件是与过滤条件匹配的类，并且具有向容器注册的相应bean定义。 这消除了使用XML来执行bean注册的需要; 相反，您可以使用注释（例如@Component），AspectJ类型表达式或您自己的自定义筛选条件来选择哪些类将使用容器注册bean定义。

从Spring 3.0开始，Spring JavaConfig项目提供的许多功能都是核心Spring Framework的一部分。 这允许您使用Java而不是使用传统的XML文件来定义bean。 有关如何使用这些新功能的示例，请查看@Configuration，@ Both，@ Import和@DependsOn注释。

### 7.10.1 @Component和进一步的构造型注释（@Component and further stereotype annotations）

@Repository注释是任何满足存储库角色或构造型（也称为数据访问对象或DAO）的类的标记。该标记的用途包括自动翻译异常，如第20.2.2节“异常翻译”中所述。

Spring提供了进一步的构造型注释：@Component，@Service和@Controller。 @Component是任何Spring管理组件的通用构造型。 @Repository，@Service和@Controller是@Component的特化，用于更具体的用例，例如，分别在持久性，服务和表示层中。因此，您可以使用@Component注释组件类，但是通过使用@Repository，@Service或@Controller注释它们，您的类更适合通过工具处理或与方面关联。例如，这些刻板印象注释成为切入点的理想目标。在未来的Spring Framework版本中，@Repository，@Service和@Controller也可能带有额外的语义。因此，如果您选择在服务层使用@Component或@Service，@Service显然是更好的选择。同样，如上所述，已经支持@Repository作为持久层中自动异常转换的标记。

### 7.10.2 元注释

Spring提供的许多注释都可以在您自己的代码中用作元注释。 元注释只是一个可以应用于另一个注释的注释。 例如，上面提到的@Service注释是使用@Component进行元注释的：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component // Spring will see this and treat @Service in the same way as @Component
public @interface Service {

    // ....
}
```

元注释也可以组合以创建组合注释。 例如，Spring MVC的@RestController注释由@Controller和@ResponseBody组成。

此外，组合注释可以选择性地从元注释重新声明属性以允许用户定制。 当您只想公开元注释属性的子集时，这可能特别有用。 例如，Spring的@SessionScope注释将范围名称硬编码为会话，但仍允许自定义proxyMode。

```
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {

    /**
     * Alias for {@link Scope#proxyMode}.
     * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
     */
    @AliasFor(annotation = Scope.class)
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;

}
```

然后可以使用@SessionScope而不声明proxyMode，如下所示：

```
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```

或者使用proxyMode的重写值，如下所示：

```
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
    // ...
}
```

有关更多详细信息，请参阅 第38章 Spring Annotation Programming Model。

### 7.10.3 自动检测类并注册bean定义

Spring可以自动检测构造型类，并使用ApplicationContext注册相应的BeanDefinition。 例如，以下两个类符合此类自动检测的条件：

```
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

```
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```

要自动检测这些类并注册相应的bean，需要将@ComponentScan添加到@Configuration类，其中basePackages属性是这两个类的公共父包。 （或者，您可以指定包含每个类的父包的逗号/分号/空格分隔列表。）

```
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```

为简洁起见，上面可能使用了注释的value属性，即@ComponentScan（“org.example”）

以下是使用XML的替代方法

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```

使用&lt;context:component-scan&gt;隐式启用&lt;context:annotation-config&gt;的功能。 使用&lt;context:component-scan&gt;时，通常不需要包含&lt;context:annotation-config&gt;元素。

扫描类路径包需要在类路径中存在相应的目录条目。 使用Ant构建JAR时，请确保不要激活JAR任务的仅文件开关。 此外，在某些环境中，类路径目录可能不会基于安全策略公开，例如： JDK 1.7.0\_45及更高版本上的独立应用程序（需要在清单中设置“Trusted-Library”;请参阅[http://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources）。](http://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources）。)

此外，使用component-scan元素时，隐式包含AutowiredAnnotationBeanPostProcessor和CommonAnnotationBeanPostProcessor。 这意味着这两个组件是自动检测并连接在一起的 - 所有这些都没有在XML中提供任何bean配置元数据。

您可以通过包含值为false的annotation-config属性来禁用AutowiredAnnotationBeanPostProcessor和CommonAnnotationBeanPostProcessor的注册。

### 7.10.4 使用过滤器自定义扫描

默认情况下，使用@Component，@ Repository，@ Service，@ Controller注释的类或自身使用@Component注释的自定义注释是唯一检测到的候选组件。 但是，您可以通过应用自定义筛选器来修改和扩展此行为。 将它们添加为@ComponentScan注释的includeFilters或excludeFilters参数（或者作为component-scan元素的include-filter或exclude-filter子元素）。 每个filter元素都需要type和expression属性。 下表介绍了筛选选项。

**Filter Types**

| Filter Type |
| :--- |


|  | Example Expression | Description |
| :--- | :--- | :--- |
| annotation \(default\) | `org.example.SomeAnnotation` | 要在目标组件中的类型级别出现的注释。 |
| assignable | `org.example.SomeClass` | 目标组件可分配给（扩展/实现）的类（或接口）。 |
| aspectj | `org.example..*Service+` | 要由目标组件匹配的AspectJ类型表达式。 |
| regex | `org\.example\.Default.*` | 要由目标组件类名匹配的正则表达式。 |
| custom | `org.example.MyTypeFilter` | A custom implementation of the`org.springframework.core.type .TypeFilter`interface. 自定义实现。 |

以下示例显示忽略所有@Repository注释并使用“stub”存储库的配置。

```
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```

和使用XML的等价物

```
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```

您还可以通过在注释上设置useDefaultFilters = false或提供use-default-filters =“false”作为&lt;component-scan /&gt;元素的属性来禁用默认过滤器。 这实际上将禁止自动检测使用@ Component，@ Repository，@ Service，@ Controller或@Configuration注释的类。

### 7.10.5 在组件中定义bean元数据

Spring组件还可以向容器提供bean定义元数据。 您可以使用相同的@Bean批注来定义@Configuration带注释的类中的bean元数据。 这是一个简单的例子：

```
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```

此类是一个Spring组件，其doWork（）方法中包含特定于应用程序的代码。 但是，它还提供了一个bean定义，它具有引用publicInstance（）方法的工厂方法。 @Bean批注标识工厂方法和其他bean定义属性，例如通过@Qualifier批注的限定符值。 可以指定的其他方法级别注释是@ Siecope，@ Lazy和自定义限定符注释。

_除了组件初始化的作用外，@ Lazy注释也可以放在标有@Autowired或@Inject的注入点上。 在这种情况下，它会导致注入惰性解析代理。_

如前所述，支持自动装配的字段和方法，并支持自动装配@Bean方法：

```
@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters
    @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @RequestScope
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }
}
```

该示例将String方法参数country自动装配到另一个名为privateInstance的bean上的age属性的值。 Spring Expression Language元素通过符号＃{&lt;expression&gt;}定义属性的值。对于@Value注释，表达式解析器预先配置为在解析表达式文本时查找bean名称。

从Spring Framework 4.3开始，您还可以声明一个类型为InjectionPoint的工厂方法参数（或其更具体的子类DependencyDescriptor），以便访问触发创建当前bean的请求注入点。请注意，这仅适用于实例创建bean实例，而不适用于注入现有实例。因此，此功能对原型范围的bean最有意义。对于其他作用域，工厂方法只会看到触发在给定作用域中创建新bean实例的注入点：例如，触发创建惰性单例bean的依赖项。在这种情况下，使用提供的注入点元数据和语义关注。

```
@Component
public class FactoryMethodComponent {

    @Bean @Scope("prototype")
    public TestBean prototypeInstance(InjectionPoint injectionPoint) {
        return new TestBean("prototypeInstance for " + injectionPoint.getMember());
    }
}
```

常规Spring组件中的@Bean方法的处理方式与Spring @Configuration类中的对应方式不同。 不同之处在于，使用CGLIB不会增强@Component类来拦截方法和字段的调用。 CGLIB代理是调用@Configuration类中的@Bean方法中的方法或字段创建对协作对象的bean元数据引用的方法; 这些方法不是用普通的Java语义调用的，而是通过容器来提供通常的生命周期管理和Spring bean的代理，即使在通过对@Bean方法的编程调用引用其他bean时也是如此。 相反，在普通的@Component类中调用@Bean方法中的方法或字段具有标准的Java语义，没有应用特殊的CGLIB处理或其他约束。

---

您可以将@Bean方法声明为static，允许在不创建包含配置类作为实例的情况下调用它们。 这在定义后处理器bean时特别有意义，例如， 类型为BeanFactoryPostProcessor或BeanPostProcessor，因为这些bean将在容器生命周期的早期初始化，并应避免在此时触发配置的其他部分。

请注意，对静态@Bean方法的调用永远不会被容器拦截，甚至在@Configuration类中也不会被拦截（参见上文）。 这是由于技术限制：CGLIB子类化只能覆盖非静态方法。 因此，直接调用另一个@Bean方法将具有标准的Java语义，从而导致直接从工厂方法本身返回一个独立的实例。

@Bean方法的Java语言可见性不会立即影响Spring容器中的结果bean定义。 您可以自由地声明您在非@Configuration类中适用的工厂方法，也可以在任何地方声明静态方法。 但是，@ Consfiguration类中的常规@Bean方法需要是可覆盖的，即它们不能声明为private或final。

@Bean方法也将在给定组件或配置类的基类上以及在由组件或配置类实现的接口中声明的Java 8默认方法上发现。 这使得组成复杂配置安排具有很大的灵活性，从Spring 4.2开始，甚至可以通过Java 8默认方法实现多重继承。

最后，请注意，单个类可以为同一个bean保存多个@Bean方法，作为根据运行时可用依赖项使用的多个工厂方法的排列。 这与在其他配置方案中选择“最贪婪”构造函数或工厂方法的算法相同：将在构造时选择具有最多可满足依赖项的变体，类似于容器在多个@Autowired构造函数之间进行选择的方式。

---

### 7.10.6 命名自动检测的组件

当组件作为扫描过程的一部分自动检测时，其bean名称由该扫描程序已知的BeanNameGenerator策略生成。 默认情况下，任何包含名称值的Spring构造型注释（@ Component，@ Repository，@ Service和@Controller）都将为相应的bean定义提供该名称。

如果此类注释不包含任何名称值或任何其他检测到的组件（例如自定义过滤器发现的那些组件），则默认的bean名称生成器将返回未大写的非限定类名称。 例如，如果检测到以下组件类，则名称为myMovieLister和movieFinderImpl：

```
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```

```
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

如果您不想依赖默认的bean命名策略，则可以提供自定义bean命名策略。 首先，实现BeanNameGenerator接口，并确保包含默认的无参数构造函数。 然后，在配置扫描程序时提供完全限定的类名：

```
@Configuration
@ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
public class AppConfig {
    ...
}
```

```
<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>
```

作为一般规则，考虑在其他组件可能对其进行显式引用时使用注释指定名称。 另一方面，只要容器负责接线，自动生成的名称就足够了。

### 7.10.7 为自动检测的组件提供范围

与一般的Spring管理组件一样，自动检测组件的默认和最常见的范围是单例。 但是，有时您需要一个可以通过@Scope注释指定的不同范围。 只需在注释中提供范围的名称：

```
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

@Scope注释仅在具体bean类（用于带注释的组件）或工厂方法（用于@Bean方法）上进行内省。 与XML bean定义相比，没有bean定义继承的概念，类级别的继承层次结构与元数据目的无关。

有关特定于Web的范围（如Spring上下文中的“request”/“session”）的详细信息，请参见第7.5.4节“请求，会话，全局会话，应用程序和WebSocket范围”。 与这些范围的预构建注释一样，您也可以使用Spring的元注释方法编写自己的范围注释：例如 使用@Scope（“prototype”）进行元注释的自定义注释，可能还声明了自定义范围代理模式。

要为范围解析提供自定义策略而不是依赖基于注释的方法，请实现ScopeMetadataResolver接口，并确保包含默认的无参数构造函数。 然后，在配置扫描程序时提供完全限定的类名：

```
@Configuration
@ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class)
public class AppConfig {
    ...
}
```

```
<beans>
    <context:component-scan base-package="org.example" scope-resolver="org.example.MyScopeResolver"/>
</beans>
```

使用某些非单例范围时，可能需要为范围对象生成代理。 推理在“Scoped beans as dependencies”一节中描述。 为此，组件扫描元素上提供了scoped-proxy属性。 三个可能的值是：no，interfaces和targetClass。 例如，以下配置将生成标准JDK动态代理：

```
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
    ...
}
```

```
<beans>
    <context:component-scan base-package="org.example" scoped-proxy="interfaces"/>
</beans>
```

### 7.10.8 提供带注释的限定符元数据

@Qualifier注释将在第7.9.4节“使用限定符微调基于注释的自动装配”中讨论。 该部分中的示例演示了在解析自动线候选时使用@Qualifier注释和自定义限定符注释来提供细粒度控制。 因为这些示例基于XML bean定义，所以使用XML中bean元素的限定符或元子元素在候选bean定义上提供限定符元数据。 当依赖类路径扫描来自动检测组件时，可以在候选类上为类型级注释提供限定符元数据。 以下三个示例演示了此技术：

```
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```

与大多数基于注释的备选方案一样，请记住注释元数据绑定到类定义本身，而XML的使用允许多个相同类型的bean在其限定符元数据中提供变体，因为每个元数据都是按照  - 实例而不是每班。




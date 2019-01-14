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

扫描类路径包需要在类路径中存在相应的目录条目。 使用Ant构建JAR时，请确保不要激活JAR任务的仅文件开关。 此外，在某些环境中，类路径目录可能不会基于安全策略公开，例如： JDK 1.7.0\_45及更高版本上的独立应用程序（需要在清单中设置“Trusted-Library”;请参阅http://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources）。

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

### 7.10.6 命名自动检测的组件

### 7.10.7 为自动检测的组件提供范围

### 7.10.8 提供带注释的限定符元数据




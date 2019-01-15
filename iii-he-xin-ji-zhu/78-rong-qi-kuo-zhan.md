## 7.8  容器扩展点

通常，应用程序开发人员不需要子类化ApplicationContext实现类。 相反，可以通过插入特殊集成接口的实现来扩展Spring IoC容器。 接下来的几节将介绍这些集成接口。

### 7.8.1 使用BeanPostProcessor定制bean

BeanPostProcessor接口定义了可以实现的回调方法，以提供您自己的（或覆盖容器的默认）实例化逻辑，依赖关系解析逻辑等。 如果要在Spring容器完成实例化，配置和初始化bean之后实现某些自定义逻辑，则可以插入一个或多个BeanPostProcessor实现。

您可以配置多个BeanPostProcessor实例，并且可以通过设置order属性来控制这些BeanPostProcessors的执行顺序。 仅当BeanPostProcessor实现Ordered接口时，才能设置此属性; 如果您编写自己的BeanPostProcessor，则应考虑实现Ordered接口。 有关更多详细信息，请参阅BeanPostProcessor和Ordered接口的javadoc。 另请参阅下面有关BeanPostProcessors编程注册的说明。

---

**BeanPostProcessors在bean（或对象）实例上运行; 也就是说，Spring IoC容器实例化一个bean实例，然后BeanPostProcessors完成它们的工作。**

**BeanPostProcessors的范围是每个容器。 这仅在您使用容器层次结构时才相关。 如果在一个容器中定义BeanPostProcessor，它将只对该容器中的bean进行后处理。 换句话说，在一个容器中定义的bean不会被另一个容器中定义的BeanPostProcessor进行后处理，即使两个容器都是同一层次结构的一部分。**

**要更改实际的bean定义（即定义bean的蓝图），您需要使用BeanFactoryPostProcessor，如第7.8.2节“使用BeanFactoryPostProcessor定制配置元数据”中所述。**

---

org.springframework.beans.factory.config.BeanPostProcessor接口由两个回调方法组成。当这样的类被注册为带有容器的后处理器时，对于容器创建的每个bean实例，后处理器在容器初始化方法之前从容器中获取回调（例如InitializingBean的afterPropertiesSet（）和任何声明的init方法）以及任何bean初始化回调之后都会被调用。后处理器可以对bean实例执行任何操作，包括完全忽略回调。 bean后处理器通常检查回调接口或者可以用代理包装bean。一些Spring AOP基础结构类实现为bean后处理器，以便提供代理包装逻辑。

ApplicationContext自动检测在实现BeanPostProcessor接口的配置元数据中定义的任何bean。 ApplicationContext将这些bean注册为后处理器，以便稍后在创建bean时调用它们。 Bean后处理器可以像任何其他bean一样部署在容器中。

请注意，在配置类上使用@Bean工厂方法声明BeanPostProcessor时，工厂方法的返回类型应该是实现类本身，或者至少是org.springframework.beans.factory.config.BeanPostProcessor接口，清楚地表明该bean的后处理器性质。否则，ApplicationContext将无法在完全创建之前按类型自动检测它。由于BeanPostProcessor需要尽早实例化以便应用于上下文中其他bean的初始化，因此这种早期类型检测至关重要。

---

虽然BeanPostProcessor注册的推荐方法是通过ApplicationContext自动检测（如上所述），但也可以使用addBeanPostProcessor方法以编程方式对ConfigurableBeanFactory注册它们。 当需要在注册之前评估条件逻辑，或者甚至在层次结构中跨上下文复制bean后处理器时，这非常有用。 但请注意，以编程方式添加的BeanPostProcessors不遵循Ordered接口。 这是注册的顺序，它决定了执行的顺序。 另请注意，以编程方式注册的BeanPostProcessors始终在通过自动检测注册的BeanPostProcessors之前处理，而不管任何显式排序。

实现BeanPostProcessor接口的类是特殊的，容器会对它们进行不同的处理。直接引用的所有BeanPostProcessors和bean都会在启动时实例化，这是ApplicationContext的特殊启动阶段的一部分。接下来，所有BeanPostProcessors都以排序方式注册，并应用于容器中的所有其他bean。因为AOP自动代理是作为BeanPostProcessor本身实现的，所以BeanPostProcessors和它们直接引用的bean都不符合自动代理的条件，因此不会将方面编入其中。

对于任何此类bean，您应该看到一条信息性日志消息：“Bean foo不适合所有BeanPostProcessor接口处理（例如：不符合自动代理条件）”。

请注意，如果您使用自动装配或@Resource（可能会回退到自动装配）将Bean连接到BeanPostProcessor，则Spring可能会在搜索类型匹配依赖项候选项时访问意外的bean，从而使它们不符合自动代理或其他类型的条件豆后处理。例如，如果你有一个使用@Resource注释的依赖项，其中field / setter名称不直接对应于bean的声明名称而没有使用name属性，那么Spring将访问其他bean以按类型匹配它们。

---

以下示例显示如何在ApplicationContext中编写，注册和使用BeanPostProcessors。

#### 示例：Hello World，BeanPostProcessor样式

第一个例子说明了基本用法。 该示例显示了一个自定义BeanPostProcessor实现，该实现在容器创建时调用每个bean的toString（）方法，并将生成的字符串输出到系统控制台。

在下面找到自定义BeanPostProcessor实现类定义：

```
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/lang
        http://www.springframework.org/schema/lang/spring-lang.xsd">

    <lang:groovy id="messenger"
            script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
        <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
    </lang:groovy>

    <!--
    when the above bean (messenger) is instantiated, this custom
    BeanPostProcessor implementation will output the fact to the system console
    -->
    <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>
```

请注意如何简单地定义InstantiationTracingBeanPostProcessor。 它甚至没有名称，因为它是一个bean，它可以像任何其他bean一样被依赖注入。 （前面的配置还定义了一个由Groovy脚本支持的bean。Spring动态语言支持在第35章动态语言支持一章中有详细介绍。）

以下简单Java应用程序执行上述代码和配置：

```
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
        Messenger messenger = (Messenger) ctx.getBean("messenger");
        System.out.println(messenger);
    }

}
```

上述应用程序的输出类似于以下内容：

```
Bean 'messenger' created : org.springframework.scripting.groovy.GroovyMessenger@272961
org.springframework.scripting.groovy.GroovyMessenger@272961
```



#### 示例：RequiredAnnotationBeanPostProcessor

将回调接口或注释与自定义BeanPostProcessor实现结合使用是扩展Spring IoC容器的常用方法。 一个例子是Spring的RequiredAnnotationBeanPostProcessor  - 一个随Spring发行版一起提供的BeanPostProcessor实现，它确保用（任意）注释标记的bean上的JavaBean属性实际上（配置为）依赖注入值。

### 7.8.2 使用BeanFactoryPostProcessor定制配置元数据

我们将看到的下一个扩展点是org.springframework.beans.factory.config.BeanFactoryPostProcessor。 这个接口的语义类似于BeanPostProcessor的语义，主要区别在于：BeanFactoryPostProcessor对bean配置元数据进行操作; 也就是说，Spring IoC容器允许BeanFactoryPostProcessor读取配置元数据，并可能在容器实例化除BeanFactoryPostProcessors之外的任何bean之前更改它。

您可以配置多个BeanFactoryPostProcessors，并且可以通过设置order属性来控制这些BeanFactoryPostProcessors的执行顺序。 但是，如果BeanFactoryPostProcessor实现Ordered接口，则只能设置此属性。 如果编写自己的BeanFactoryPostProcessor，则应考虑实现Ordered接口。 有关更多详细信息，请参阅BeanFactoryPostProcessor和Ordered接口的javadoc。

---

**如果要更改实际的bean实例（即，从配置元数据创建的对象），则需要使用BeanPostProcessor（如上面第7.8.1节“使用BeanPostProcessor定制bean”中所述）。 虽然技术上可以在BeanFactoryPostProcessor中使用bean实例（例如，使用BeanFactory.getBean（）），但这样做会导致过早的bean实例化，从而违反标准的容器生命周期。 这可能会导致负面影响，例如绕过bean后处理。**

**此外，BeanFactoryPostProcessors的范围是每个容器。 这仅在您使用容器层次结构时才相关。 如果在一个容器中定义BeanFactoryPostProcessor，它将仅应用于该容器中的bean定义。 BeanFactoryPostProcessors不会在另一个容器中对一个容器中的Bean定义进行后处理，即使两个容器都是同一层次结构的一部分。**

---

bean工厂后处理器在ApplicationContext中声明时自动执行，以便将更改应用于定义容器的配置元数据。 Spring包含许多预定义的bean工厂后处理器，例如PropertyOverrideConfigurer和PropertyPlaceholderConfigurer。 例如，也可以使用自定义BeanFactoryPostProcessor来注册自定义属性编辑器。

ApplicationContext自动检测部署到其中的任何实现BeanFactoryPostProcessor接口的bean。 它在适当的时候使用这些bean作为bean工厂后处理器。 您可以像处理任何其他bean一样部署这些后处理器bean。

与BeanPostProcessors一样，您通常不希望为延迟初始化配置BeanFactoryPostProcessors。 如果没有其他bean引用Bean（Factory）PostProcessor，则该后处理器根本不会被实例化。 因此，将忽略将其标记为延迟初始化，即使在&lt;beans /&gt;元素的声明中将default-lazy-init属性设置为true，也会急切地实例化Bean（Factory）PostProcessor。



#### 示例：类名替换PropertyPlaceholderConfigurer

您可以使用PropertyPlaceholderConfigurer使用标准Java Properties格式在单独的文件中外部化bean定义中的属性值。 这样做可以使部署应用程序的人员自定义特定于环境的属性（如数据库URL和密码），而无需修改主XML定义文件或容器文件的复杂性或风险。

请考虑以下基于XML的配置元数据片段，其中定义了具有占位符值的DataSource。 该示例显示了从外部属性文件配置的属性。 在运行时，PropertyPlaceholderConfigurer应用于将替换DataSource的某些属性的元数据。 要替换的值指定为$ {property-name}形式的占位符，该形式遵循Ant / log4j / JSP EL样式。

```
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations" value="classpath:com/foo/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

实际值来自标准Java Properties格式的另一个文件：

```
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```

因此，字符串$ {jdbc.username}在运行时将替换为值'sa'，这同样适用于与属性文件中的键匹配的其他占位符值。 PropertyPlaceholderConfigurer检查bean定义的大多数属性和属性中的占位符。 此外，可以自定义占位符前缀和后缀。

使用Spring 2.5中引入的上下文命名空间，可以使用专用配置元素配置属性占位符。 可以在location属性中提供一个或多个位置作为逗号分隔列表。

```
<context:property-placeholder location="classpath:com/foo/jdbc.properties"/>
```

PropertyPlaceholderConfigurer不仅在您指定的属性文件中查找属性。 默认情况下，如果它无法在指定的属性文件中找到属性，它还会检查Java System属性。 您可以通过使用以下三个受支持的整数值之一设置configurer的systemPropertiesMode属性来自定义此行为：

* never（0）：从不检查系统属性
* fallback（1）：如果在指定的属性文件中无法解析，则检查系统属性。 这是默认值。
* override（2）：在尝试指定的属性文件之前，首先检查系统属性。 这允许系统属性覆盖任何其他属性源。

有关更多信息，请参阅PropertyPlaceholderConfigurer javadocs。

您可以使用PropertyPlaceholderConfigurer替换类名，这在您必须在运行时选择特定实现类时有时很有用。 例如：

```
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
        <value>classpath:com/foo/strategy.properties</value>
    </property>
    <property name="properties">
        <value>custom.strategy.class=com.foo.DefaultStrategy</value>
    </property>
</bean>

<bean id="serviceStrategy" class="${custom.strategy.class}"/>
```

如果在运行时无法将类解析为有效类，则在即将创建bean时，bean的解析将失败，这是在非lazy-init bean的ApplicationContext的preInstantiateSingletons（）阶段期间。

#### 示例：PropertyOverrideConfigurer

PropertyOverrideConfigurer是另一个bean工厂后处理器，类似于PropertyPlaceholderConfigurer，但与后者不同，原始定义可以具有默认值，或者根本不具有bean属性的值。 如果重写的Properties文件没有某个bean属性的条目，则使用默认的上下文定义。

请注意，bean定义不知道被覆盖，因此从XML定义文件中可以立即看出正在使用覆盖配置器。 如果多个PropertyOverrideConfigurer实例为同一个bean属性定义了不同的值，则由于覆盖机制，最后一个实例会获胜。

属性文件配置行采用以下格式：

```
beanName.property=value
```

例如：

```
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql:mydb
```

此示例文件可以与包含名为dataSource的bean的容器定义一起使用，该bean具有驱动程序和url属性。

复制属性名称也受支持，只要路径的每个组件（重写的最终属性除外）都已经非空（可能由构造函数初始化）。 在这个例子中......

```
foo.fred.bob.sammy=123
```

foo bean的fred属性的bob属性的sammy属性设置为标量值123。

指定的覆盖值始终是文字值; 它们不会被翻译成bean引用。 当XML bean定义中的原始值指定bean引用时，此约定也适用。

使用Spring 2.5中引入的上下文命名空间，可以使用专用配置元素配置属性覆盖：

```
<context:property-override location="classpath:override.properties"/>
```

### 7.8.3 使用FactoryBean自定义实例化逻辑

为本身为工厂的对象实现org.springframework.beans.factory.FactoryBean接口。

FactoryBean接口是Spring IoC容器实例化逻辑的可插拔点。如果你有一个复杂的初始化代码，用Java表示，而不是（可能）冗长的XML，你可以创建自己的FactoryBean，在该类中编写复杂的初始化，然后将自定义FactoryBean插入容器。

FactoryBean接口提供三种方法：

* Object getObject（）：返回此工厂创建的对象的实例。实例可以共享，具体取决于此工厂是返回单例还是原型。
* boolean isSingleton（）：如果此FactoryBean返回单例，则返回true，否则返回false。
* 类getObjectType（）：返回getObject（）方法返回的对象类型，如果事先不知道类型，则返回null。

FactoryBean概念和接口用于Spring Framework中的许多地方; FactoryBean接口的50多个实现随Spring一起提供。

当您需要向容器询问实际的FactoryBean实例本身而不是它生成的bean时，在调用ApplicationContext的getBean（）方法时，使用＆符号（＆）作为bean的id前缀。因此，对于id为myBean的给定FactoryBean，在容器上调用getBean（“myBean”）将返回FactoryBean的产品;而，调用getBean（“＆myBean”）会返回FactoryBean实例本身。


## 12.5 使用ProxyFactoryBean创建AOP代理

如果您正在为业务对象使用Spring IoC容器（ApplicationContext或BeanFactory） - 您应该这样做！  - 您将需要使用Spring的AOP FactoryBeans之一。 （请记住，工厂bean引入了一个间接层，使其能够创建不同类型的对象。）

\[注意\]

Spring AOP支持还使用了工厂bean。

在Spring中创建AOP代理的基本方法是使用org.springframework.aop.framework.ProxyFactoryBean。 这样可以完全控制将要应用的切入点和建议及其排序。 但是，如果您不需要此类控件，则可以使用更简单的选项。

### 12.5.1 基本

与其他Spring FactoryBean实现一样，ProxyFactoryBean引入了一个间接层。 如果定义名为foo的ProxyFactoryBean，那么引用foo的对象不是ProxyFactoryBean实例本身，而是由ProxyFactoryBean的getObject（）方法实现创建的对象。 此方法将创建包装目标对象的AOP代理。

使用ProxyFactoryBean或其他IoC感知类来创建AOP代理的最重要的好处之一是，它意味着IoC还可以管理建议和切入点。 这是一个强大的功能，可以实现其他AOP框架难以实现的某些方法。 例如，一个建议本身可以引用应用程序对象（除了目标，它应该在任何AOP框架中可用），受益于依赖注入提供的所有可插入性。

### 12.5.2  JavaBean属性

与Spring提供的大多数FactoryBean实现一样，ProxyFactoryBean类本身就是一个JavaBean。其属性用于：

* 指定要代理的目标。
* 指定是否使用CGLIB（参见下文以及第12.5.3节“基于JDK和CGLIB的代理”）。

一些关键属性继承自org.springframework.aop.framework.ProxyConfig（Spring中所有AOP代理工厂的超类）。这些关键属性包括：

* proxyTargetClass：如果要代理目标类，则为true，而不是目标类的接口。如果此属性值设置为true，则将创建CGLIB代理（但另请参见第12.5.3节“基于JDK和CGLIB的代理”）。
* optimize：控制是否将积极优化应用于通过CGLIB创建的代理。除非完全理解相关AOP代理如何处理优化，否则不应轻易使用此设置。目前仅用于CGLIB代理;它对JDK动态代理没有影响。
* 冻结：如果冻结了代理配置，则不再允许更改配置。这既可以作为轻微优化，也可以用于在创建代理后不希望调用者能够操作代理（通过Advised接口）的情况。此属性的默认值为false，因此允许添加其他建议等更改。
* exposeProxy：确定当前代理是否应该在ThreadLocal中公开，以便目标可以访问它。如果目标需要获取代理并且exposeProxy属性设置为true，则目标可以使用AopContext.currentProxy（）方法。

ProxyFactoryBean特有的其他属性包括：

* proxyInterfaces：String接口名称数组。如果未提供，则将使用目标类的CGLIB代理（但另请参见第12.5.3节“基于JDK和CGLIB的代理”）。
* interceptorNames：要应用的Advisor，拦截器或其他建议名称的字符串数组。以先到先得的方式订购非常重要。也就是说，列表中的第一个拦截器将是第一个能够拦截调用的拦截器。

名称是当前工厂中的bean名称，包括来自祖先工厂的bean名称。你不能在这里提到bean引用，因为这样做会导致ProxyFactoryBean忽略通知的单例设置。

您可以使用星号（\*）附加拦截器名称。这将导致应用所有顾问bean，其名称以要应用星号之前的部分开头。有关使用此功能的示例，请参见第12.5.6节“使用'全局'顾问程序”。

* singleton：无论调用getObject（）方法的频率如何，工厂是否应该返回单个对象。几个FactoryBean实现提供了这样的方法。默认值是true。如果您想使用有状态建议 - 例如，对于有状态的mixins  - 使用原型建议以及单个值false。

### 12.5.3 基于JDK和CGLIB的代理

本节作为关于ProxyFactoryBean如何选择为特定目标对象（即要代理）创建基于JDK和CGLIB的代理之一的权威文档。

\[注意\]

ProxyFactoryBean在创建基于JDK或CGLIB的代理方面的行为在Spring的1.2.x和2.0版本之间发生了变化。现在，ProxyFactoryBean在自动检测接口方面表现出与TransactionProxyFactoryBean类相似的语义。

如果要代理的目标对象的类（以下简称为目标类）未实现任何接口，则将创建基于CGLIB的代理。这是最简单的方案，因为JDK代理是基于接口的，没有接口意味着甚至不可能进行JDK代理。只需插入目标bean，并通过interceptorNames属性指定拦截器列表。请注意，即使ProxyFactoryBean的proxyTargetClass属性已设置为false，也将创建基于CGLIB的代理。 （显然这没有任何意义，最好从bean定义中删除，因为它最多是冗余的，最糟糕的是混乱。）

如果目标类实现一个（或多个）接口，则创建的代理类型取决于ProxyFactoryBean的配置。

如果ProxyFactoryBean的proxyTargetClass属性已设置为true，则将创建基于CGLIB的代理。这是有道理的，并且符合最少惊喜的原则。即使ProxyFactoryBean的proxyInterfaces属性已设置为一个或多个完全限定的接口名称，proxyTargetClass属性设置为true这一事实也会导致基于CGLIB的代理生效。

如果ProxyFactoryBean的proxyInterfaces属性已设置为一个或多个完全限定的接口名称，则将创建基于JDK的代理。创建的代理将实现proxyInterfaces属性中指定的所有接口;如果目标类碰巧实现了比在proxyInterfaces属性中指定的接口多得多的接口，那么这一切都很好，但返回的代理不会实现这些额外的接口。

如果尚未设置ProxyFactoryBean的proxyInterfaces属性，但目标类确实实现了一个（或多个）接口，那么ProxyFactoryBean将自动检测目标类确实实现至少一个接口和JDK-的事实将创建基于代理的代理。实际代理的接口将是目标类实现的所有接口;实际上，这与仅提供目标类为proxyInterfaces属性实现的每个接口的列表相同。但是，它的工作量明显减少，并且不太容易出现错别字。



### 12.5.4 代理接口

让我们看一下ProxyFactoryBean的一个简单示例。 这个例子涉及：

* 将被代理的目标bean。 这是下面示例中的“personTarget”bean定义。
* 用于提供建议的顾问和拦截器。
* AOP代理bean定义，指定目标对象（personTarget bean）和要代理的接口，以及要应用的建议。

```
<bean id="personTarget" class="com.mycompany.PersonImpl">
    <property name="name" value="Tony"/>
    <property name="age" value="51"/>
</bean>

<bean id="myAdvisor" class="com.mycompany.MyAdvisor">
    <property name="someProperty" value="Custom string property value"/>
</bean>

<bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor">
</bean>

<bean id="person"
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="com.mycompany.Person"/>

    <property name="target" ref="personTarget"/>
    <property name="interceptorNames">
        <list>
            <value>myAdvisor</value>
            <value>debugInterceptor</value>
        </list>
    </property>
</bean>
```

请注意，interceptorNames属性采用String列表：当前工厂中拦截器或顾问程序的bean名称。 可以使用顾问，拦截器，返回之前，投掷建议对象。 顾问的排序很重要。

\[注意\]

您可能想知道为什么列表不包含bean引用。 这样做的原因是，如果ProxyFactoryBean的singleton属性设置为false，则它必须能够返回独立的代理实例。 如果任何顾问本身就是原型，则需要返回一个独立的实例，因此必须能够从工厂获得原型的实例; 持有参考是不够的。

上面的“person”bean定义可以用来代替Person实现，如下所示：

```
Person person = (Person) factory.getBean("person");
```

与普通Java对象一样，同一IoC上下文中的其他bean可以表达对它的强类型依赖关系：

```
<bean id="personUser" class="com.mycompany.PersonUser">
    <property name="person"><ref bean="person"/></property>
</bean>
```

此示例中的PersonUser类将公开Person类型的属性。 就其而言，可以透明地使用AOP代理来代替“真实”的人实现。 但是，它的类将是一个动态代理类。 可以将其投射到Advised界面（下面讨论）。

可以使用匿名内部bean隐藏目标和代理之间的区别，如下所示。 只有ProxyFactoryBean定义不同; 仅包含完整性的建议：

```
<bean id="myAdvisor" class="com.mycompany.MyAdvisor">
    <property name="someProperty" value="Custom string property value"/>
</bean>

<bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor"/>

<bean id="person" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="com.mycompany.Person"/>
    <!-- Use inner bean, not local reference to target -->
    <property name="target">
        <bean class="com.mycompany.PersonImpl">
            <property name="name" value="Tony"/>
            <property name="age" value="51"/>
        </bean>
    </property>
    <property name="interceptorNames">
        <list>
            <value>myAdvisor</value>
            <value>debugInterceptor</value>
        </list>
    </property>
</bean>
```

这样做的好处是，只有一个Person类型的对象：如果我们想要阻止应用程序上下文的用户获得对未建议对象的引用，或者需要避免使用Spring IoC自动装配的任何歧义，这将非常有用。 还有一个优点是ProxyFactoryBean定义是自包含的。 但是，有时能够从工厂获得未建议的目标实际上可能是一个优势：例如，在某些测试场景中。

### 12.5.5 代理classes

如果您需要代理一个类而不是一个或多个接口，该怎么办？

想象一下，在上面的示例中，没有Person接口：我们需要建议一个名为Person的类，它没有实现任何业务接口。在这种情况下，您可以将Spring配置为使用CGLIB代理，而不是动态代理。只需将ProxyFactoryBean上的proxyTargetClass属性设置为true即可。虽然最好是编程接口而不是类，但在使用遗留代码时，建议不实现接口的类的能力会很有用。 （一般来说，Spring不是规定性的。虽然它可以很容易地应用好的实践，但它避免强制使用特定的方法。）

如果您愿意，即使您有接口，也可以在任何情况下强制使用CGLIB。

CGLIB代理通过在运行时生成目标类的子类来工作。 Spring将这个生成的子类配置为委托对原始目标的方法调用：子类用于实现Decorator模式，在通知中编织。

CGLIB代理通常应对用户透明。但是，有一些问题需要考虑：

无法建议最终方法，因为它们无法被覆盖。

无需将CGLIB添加到类路径中。从Spring 3.2开始，CGLIB被重新打包并包含在spring-core JAR中。换句话说，基于CGLIB的AOP将像JDK动态代理一样“开箱即用”。

CGLIB代理和动态代理之间的性能差异很小。从Spring 1.0开始，动态代理略快一些。但是，这可能会在未来发生变化。在这种情况下，绩效不应该是决定性的考虑因素。

### 12.5.6 使用“global” advisors

通过在拦截器名称后附加星号，所有具有与星号前面部分匹配的bean名称的advisors程序将添加到advisors程序链中。 如果您需要添加一组标准的“global”advisors程序，这可以派上用场：

```
<bean id="proxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="service"/>
    <property name="interceptorNames">
        <list>
            <value>global*</value>
        </list>
    </property>
</bean>

<bean id="global_debug" class="org.springframework.aop.interceptor.DebugInterceptor"/>
<bean id="global_performance" class="org.springframework.aop.interceptor.PerformanceMonitorInterceptor"/>
```




## 12.9 使用“自动代理”工具

到目前为止，我们已经考虑使用ProxyFactoryBean或类似的工厂bean显式创建AOP代理。

Spring还允许我们使用“自动代理”bean定义，它可以自动代理选定的bean定义。 这是基于Spring“bean post processor”基础结构构建的，它可以在容器加载时修改任何bean定义。

在此模型中，您在XML bean定义文件中设置了一些特殊的bean定义来配置自动代理基础结构。 这允许您只声明符合自动代理的目标：您不需要使用ProxyFactoryBean。

有两种方法可以做到这一点：

* 使用引用当前上下文中特定bean的自动代理创建器。
* 自动代理创建的一个特例，值得单独考虑; 由源级元数据属性驱动的自动代理创建。

### 12.9.1  Autoproxy bean定义

org.springframework.aop.framework.autoproxy包提供以下标准自动代理创建器。

#### BeanNameAutoProxyCreator

BeanNameAutoProxyCreator类是一个BeanPostProcessor，它自动为名称与文字值或通配符匹配的bean创建AOP代理。

```
<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
    <property name="beanNames" value="jdk*,onlyJdk"/>
    <property name="interceptorNames">
        <list>
            <value>myInterceptor</value>
        </list>
    </property>
</bean>
```

与ProxyFactoryBean一样，有一个interceptorNames属性而不是拦截器列表，以允许原型顾问程序的正确行为。 命名为“拦截器”可以是顾问或任何建议类型。

与通常的自动代理一样，使用BeanNameAutoProxyCreator的要点是将相同的配置一致地应用于多个对象，并且配置量最小。 将声明性事务应用于多个对象是一种流行的选择。

名称匹配的Bean定义（例如上例中的“jdkMyBean”和“onlyJdk”）是具有目标类的普通旧bean定义。 BeanNameAutoProxyCreator将自动创建AOP代理。 相同的建议将应用于所有匹配的bean。 请注意，如果使用顾问程序（而不是上例中的拦截器），则切入点可能会以不同的方式应用于不同的bean。

#### DefaultAdvisorAutoProxyCreator

DefaultAdvisorAutoProxyCreator是一个更通用且功能更强大的自动代理创建器。这将在当前上下文中自动应用符合条件的顾问程序，而无需在auto-proxy advisor的bean定义中包含特定的bean名称。它提供与BeanNameAutoProxyCreator一样的一致配置和避免重复的优点。

使用此机制涉及：

* 指定DefaultAdvisorAutoProxyCreator bean定义。
* 在相同或相关的上下文中指定任意数量的顾问。请注意，这些必须是顾问，而不仅仅是拦截器或其他建议。这是必要的，因为必须有一个切入点来评估，以检查每个建议对候选bean定义的合格性。

DefaultAdvisorAutoProxyCreator将自动评估每个顾问程序中包含的切入点，以查看它应该应用于每个业务对象的建议（如果有的话）（例如示例中的“businessObject1”和“businessObject2”）。

这意味着可以自动将任意数量的顾问程序应用于每个业务对象。如果任何顾问程序中的切入点与业务对象中的任何方法都不匹配，则不会代理该对象。当为新业务对象添加bean定义时，如有必要，它们将自动被代理。

一般的自动代理具有使调用者或依赖者无法获得未建议的对象的优点。在此ApplicationContext上调用getBean（“businessObject1”）将返回AOP代理，而不是目标业务对象。 （前面所示的“内豆”成语也提供了这种好处。）

```
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="customAdvisor" class="com.mycompany.MyAdvisor"/>

<bean id="businessObject1" class="com.mycompany.BusinessObject1">
    <!-- Properties omitted -->
</bean>

<bean id="businessObject2" class="com.mycompany.BusinessObject2"/>
```

如果要将相同的建议一致地应用于许多业务对象，则DefaultAdvisorAutoProxyCreator非常有用。 基础架构定义到位后，您只需添加新业务对象，而无需包含特定的代理配置。 您还可以非常轻松地删除其他方面 - 例如，跟踪或性能监视方面 - 只需对配置进行最少的更改。

DefaultAdvisorAutoProxyCreator支持过滤（使用命名约定，以便仅评估某些顾问程序，允许在同一工厂中使用多个，不同配置的AdvisorAutoProxyCreators）和排序。 如果这是一个问题，顾问可以实现org.springframework.core.Ordered接口以确保正确的排序。 上例中使用的TransactionAttributeSourceAdvisor具有可配置的订单值; 默认设置是无序的。

#### AbstractAdvisorAutoProxyCreator

这是DefaultAdvisorAutoProxyCreator的超类。 您可以通过继承此类来创建自己的自动代理创建者，万一顾问程序定义对框架DefaultAdvisorAutoProxyCreator的行为提供的定制不足。

### 12.9.2 使用元数据驱动的自动代理

一种特别重要的自动代理类型由元数据驱动。这产生了与.NET ServicedComponents类似的编程模型。不是在XML描述符中定义元数据，而是在源级属性中保存事务管理和其他企业服务的配置。

在这种情况下，您将使用DefaultAdvisorAutoProxyCreator与理解元数据属性的Advisors结合使用。元数据细节保存在候选顾问程序的切入点部分，而不是自动代理创建类本身。

这确实是DefaultAdvisorAutoProxyCreator的特例，但值得自己考虑。 （元数据感知代码位于顾问程序中包含的切入点中，而不是AOP框架本身。）

JPetStore示例应用程序的/ attributes目录显示了属性驱动的自动代理的使用。在这种情况下，不需要使用TransactionProxyFactoryBean。由于使用了元数据感知切入点，仅仅在业务对象上定义事务属性就足够了。 bean定义在/WEB-INF/declarativeServices.xml中包含以下代码。请注意，这是通用的，可以在JPetStore外部使用：

```
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="transactionInterceptor"
        class="org.springframework.transaction.interceptor.TransactionInterceptor">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributeSource">
        <bean class="org.springframework.transaction.interceptor.AttributesTransactionAttributeSource">
            <property name="attributes" ref="attributes"/>
        </bean>
    </property>
</bean>

<bean id="attributes" class="org.springframework.metadata.commons.CommonsAttributes"/>
```

DefaultAdvisorAutoProxyCreator bean定义（名称不重要，因此甚至可以省略）将获取当前应用程序上下文中的所有符合条件的切入点。在这种情况下，TransactionAttributeSourceAdvisor类型的“transactionAdvisor”bean定义将应用于携带事务属性的类或方法。 TransactionAttributeSourceAdvisor依赖于TransactionInterceptor，通过构造函数依赖。该示例通过自动装配解决了这个问题。 AttributesTransactionAttributeSource依赖于org.springframework.metadata.Attributes接口的实现。在此片段中，“attributes”bean满足此要求，使用Jakarta Commons Attributes API获取属性信息。 （必须使用Commons Attributes编译任务编译应用程序代码。）

JPetStore示例应用程序的/ annotation目录包含由JDK 1.5+注释驱动的自动代理的类似示例。以下配置允许自动检测Spring的Transactional批注，从而导致包含该批注的bean的隐式代理：

```
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="transactionInterceptor"
        class="org.springframework.transaction.interceptor.TransactionInterceptor">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributeSource">
        <bean class="org.springframework.transaction.annotation.AnnotationTransactionAttributeSource"/>
    </property>
</bean>
```

此处定义的TransactionInterceptor依赖于PlatformTransactionManager定义，该定义未包含在此通用文件中（尽管可能是这样），因为它将特定于应用程序的事务要求（通常是JTA，如本示例中所示，或者是Hibernate，JDO或JDBC）：

```
<bean id="transactionManager"
        class="org.springframework.transaction.jta.JtaTransactionManager"/>
```

\[提示\]

如果只需要声明式事务管理，那么使用这些通用XML定义将导致Spring自动使用事务属性代理所有类或方法。 您不需要直接使用AOP，编程模型类似于.NET ServicedComponents。

这种机制是可扩展的。 可以基于自定义属性执行自动代理。 你需要：

* 定义自定义属性。
* 指定具有必要建议的Advisor，包括由类或方法上存在自定义属性触发的切入点。 您可以使用现有建议，只需实现一个拾取自定义属性的静态切入点。

这样的顾问可能对每个建议的类都是唯一的（例如，mixins）：它们只需要被定义为原型，而不是单独的bean定义。 例如，上面显示的Spring测试套件中的LockMixin介绍拦截器可以与通用的DefaultIntroductionAdvisor一起使用：

```
<bean id="lockMixin" class="test.mixin.LockMixin" scope="prototype"/>

<bean id="lockableAdvisor" class="org.springframework.aop.support.DefaultIntroductionAdvisor"
        scope="prototype">
    <constructor-arg ref="lockMixin"/>
</bean>
```

请注意，lockMixin和lockableAdvisor都被定义为原型。






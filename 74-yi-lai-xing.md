## 7.4依赖性

典型的企业应用程序不包含单个对象（或Spring用法中的bean）。 即使是最简单的应用程序也有一些对象可以协同工作，以呈现最终用户所看到的连贯应用程序。 下一节将介绍如何定义多个独立的bean定义，以及对象协作实现目标的完全实现的应用程序。

### 7.4.1 依赖注入

依赖注入（DI）是一个过程，通过这个过程，对象定义它们的依赖关系，即它们使用的其他对象，只能通过构造函数参数，工厂方法的参数或在构造或返回对象实例后在对象实例上设置的属性。从工厂方法。然后容器在创建bean时注入这些依赖项。这个过程基本上是反向的，因此名称Inversion of Control（IoC），bean本身通过使用类的直接构造或服务定位器模式来控制其依赖项的实例化或位置。

使用DI原理的代码更清晰，当对象提供其依赖项时，解耦更有效。该对象不查找其依赖项，也不知道依赖项的位置或类。因此，您的类变得更容易测试，特别是当依赖关系在接口或抽象基类上时，这允许在单元测试中使用存根或模拟实现。

DI存在两个主要变体，基于构造函数的依赖注入和基于Setter的依赖注入。

#### 基于构造函数的依赖注入

基于构造函数的DI由容器调用具有多个参数的构造函数来完成，每个参数表示一个依赖项。 调用具有特定参数的static工厂方法来构造bean几乎是等效的，本讨论同样处理构造函数和static工厂方法的参数。 以下示例显示了一个只能通过构造函数注入进行依赖注入的类。 请注意，此类没有什么特别之处，它是一个POJO，它不依赖于容器特定的接口，基类或注释。

```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

#### 基于Setter的依赖注入

使用参数的类型进行构造函数参数解析匹配。 如果bean定义的构造函数参数中不存在潜在的歧义，那么在bean定义中定义构造函数参数的顺序是在实例化bean时将这些参数提供给适当的构造函数的顺序。 考虑以下class：

```
package x.y;

public class Foo {

    public Foo(Bar bar, Baz baz) {
        // ...
    }
}
```

假设Bar和Baz类与继承无关，则不存在潜在的歧义。 因此，以下配置工作正常，您无需在&lt;constructor-arg/&gt;元素中显式指定构造函数参数索引和/或类型。

```
<beans>
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
    </bean>

    <bean id="bar" class="x.y.Bar"/>

    <bean id="baz" class="x.y.Baz"/>
</beans>
```

当引用另一个bean时，类型是已知的，并且可以进行匹配（与前面的示例一样）。 当使用简单类型时，例如&lt;value&gt; true &lt;/ value&gt;，Spring无法确定值的类型，因此无法在没有帮助的情况下按类型进行匹配。 考虑以下class

#### 

#### 依赖性解决过程

#### 依赖注入的例子

### 7.4.2 依赖关系和配置详细

如上一节所述，您可以将bean属性和构造函数参数定义为对其他托管bean（协作者）的引用，或者作为内联定义的值。 Spring的基于XML的配置元数据为此目的支持其&lt;property /&gt;和&lt;constructor-arg /&gt;元素中的子元素类型。

#### 直值（基本类型，字符串等）

&lt;property /&gt;元素的value属性将属性或构造函数参数指定为人类可读的字符串表示形式。 Spring的转换服务用于将这些值从String转换为属性或参数的实际类型。

#### 引用其他bean（协作者）

#### Inner beans

&lt;property /&gt;或&lt;constructor-arg /&gt;元素中的&lt;bean /&gt;元素定义了一个所谓的内部bean。

```
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean定义不需要定义的id或名称; 如果指定，则容器不使用这样的值作为标识符。 容器还会在创建时忽略范围标志：内部bean始终是匿名的，并且始终使用外部bean创建它们。 无法将内部bean注入协作bean，而不是将其注入封闭bean或独立访问它们。

作为极端情况，可以从自定义范围接收销毁回调，例如， 对于包含在单例bean中的请求范围的内部bean：内部bean实例的创建将绑定到其包含的bean，但是销毁回调允许它参与请求范围的生命周期。 这不是常见的情况; 内部bean通常只是共享其包含bean的范围。

#### 集合

#### 空值和空字符串值

Spring将属性等的空参数视为空字符串。 以下基于XML的配置元数据片段将email属性设置为空String值（“”）。

```
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

上面的示例等效于以下Java代码：

```
exampleBean.setEmail("");
```

&lt;null /&gt;元素处理空值。 例如：

```
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

以上配置等同于以下Java代码：

```
exampleBean.setEmail(null);
```

#### 带有p命名空间的XML快捷方式

#### 带有c-namespace的XML快捷方式

#### 复合属性名称

### 7.4.3 使用依赖

如果bean是另一个bean的依赖项，通常意味着将一个bean设置为另一个bean的属性。 通常，您可以使用基于XML的配置元数据中的&lt;ref /&gt;元素来完成此操作。 但是，有时bean之间的依赖关系不那么直接; 例如，需要触发类中的静态初始化程序，例如数据库驱动程序注册。 在初始化使用此元素的bean之前，depends-on属性可以**显式强制初始化一个或多个bea**n。 以下示例使用depends-on属性表示对单个bean的依赖关系：

```
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表示对多个bean的依赖关系，请提供bean名称列表作为depends-on属性的值，使用逗号，空格和分号作为有效分隔符：

```
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

---

_bean定义中的depends-on属性既可以指定初始化时间依赖关系，也可以指定仅限单例bean的相应销毁时间依赖关系。 在给定的bean本身被销毁之前，首先销毁定义与给定bean的依赖关系的从属bean。 因此，依赖也可以控制关机顺序。_

---

### 7.4.4 懒惰初始化的bean

默认情况下，ApplicationContext实现会急切地创建和配置所有单例bean，作为初始化过程的一部分。 通常，这种预先实例化是可取的，因为配置或周围环境中的错误是立即发现的，而不是几小时甚至几天后。 如果不希望出现这种情况，可以通过将bean定义标记为延迟初始化来阻止单例bean的预实例化。 延迟初始化的bean告诉IoC容器在第一次请求时创建bean实例，而不是在启动时。

在XML中，此行为由&lt;bean /&gt;元素上的lazy-init属性控制; 例如：

```
<bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.foo.AnotherBean"/>
```

当ApplicationContext使用前面的配置时，在ApplicationContext启动时，不会急切地预先实例化名为lazy的bean，而是急切地预先实例化not.lazy bean。

但是，当延迟初始化的bean是不是延迟初始化的单例bean的依赖项时，ApplicationContext会在启动时创建延迟初始化的bean，因为它必须满足单例的依赖关系。 惰性初始化的bean被注入到其他地方的单例bean中，而不是懒惰初始化的。

您还可以使用&lt;beans /&gt;元素上的default-lazy-init属性在容器级别控制延迟初始化; 例如：

```
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

### 7.4.5 自动装配协作者

Spring容器可以自动连接协作bean之间的关系。 您可以通过检查ApplicationContext的内容，允许Spring自动为您的bean解析协作者（其他bean）。 自动装配具有以下优点：

* 自动装配可以显着减少指定属性或构造函数参数的需要。 （在本章其他地方讨论的其他机制，如bean模板，在这方面也很有价值。）
* 自动装配可以随着对象的发展更新配置。 例如，如果需要向类添加依赖项，则可以自动满足该依赖项，而无需修改配置。 因此，自动装配在开发期间尤其有用，而不会在代码库变得更稳定时否定切换到显式布线的选项。

使用基于XML的配置元数据\[2\]时，可以使用&lt;bean /&gt;元素的autowire属性为bean定义指定autowire模式。 自动装配功能有四种模式。 您指定每个bean的自动装配，因此可以选择要自动装配的那些。

**Autowiring modes**

| 模式 | 说明 |
| :--- | :--- |
| no | （默认）无自动装配。 必须通过ref元素定义Bean引用。 不建议对较大的部署更改默认设置，因为明确指定协作者可以提供更好的控制和清晰度。 在某种程度上，它记录了系统的结构。 |
| byName | 按属性名称自动装配。 Spring查找与需要自动装配的属性同名的bean。 例如，如果bean定义按名称设置为autowire，并且它包含master属性（即，它具有setMaster（..）方法），则Spring会查找名为master的bean定义，并使用它来设置 属性。 |
| byType | 如果容器中只存在一个属性类型的bean，则允许自动装配属性。 如果存在多个，则抛出致命异常，这表示您不能对该bean使用byType自动装配。 如果没有匹配的bean，则没有任何反应; 该物业未设定。 |
| 构造器 | 类似于byType，但适用于构造函数参数。 如果容器中没有构造函数参数类型的一个bean，则会引发致命错误。 |

使用byType或构造函数自动装配模式，您可以连接数组和类型集合。 在这种情况下，提供容器内与预期类型匹配的所有autowire候选者以满足依赖性。 如果预期的键类型为String，则可以自动装配强类型映射。 自动装配的Maps值将包含与预期类型匹配的所有Bean实例，而Maps键将包含相应的bean名称。

您可以将autowire行为与依赖关系检查结合起来，这将在自动装配完成后执行。

#### 自动装配的局限和缺点

自动装配在项目中一致使用时效果最佳。 如果一般不使用自动装配，那么开发人员使用它来连接一个或两个bean定义可能会让人感到困惑。

考虑自动装配的局限和缺点：

* property和constructor-arg设置中的显式依赖项始终覆盖自动装配。 您无法自动装配所谓的简单属性，例如基元，字符串和类（以及此类简单属性的数组）。 这种限制是按设计的。
* 自动装配不如显式布线精确。 虽然如上表所示，Spring会小心避免在可能产生意外结果的歧义的情况下进行猜测，但不再明确记录Spring管理对象之间的关系。
* 可能无法为可能从Spring容器生成文档的工具提供接线信息。
* 容器中的多个bean定义可能与要自动装配的setter方法或构造函数参数指定的类型匹配。 对于数组，集合或地图，这不一定是个问题。 但是，对于期望单个值的依赖关系，这种模糊性不是任意解决的。 如果没有可用的唯一bean定义，则抛出异常。

在后一种情况下，您有几种选择：

* 放弃自动装配以支持显式布线。
* 通过将autowire-candidate属性设置为false，避免对bean定义进行自动装配，如下一节所述。
* 通过将其&lt;bean /&gt;元素的primary属性设置为true，将单个bean定义指定为主要候选者。
* 使用基于注释的配置实现更细粒度的控件，如第7.9节“基于注释的容器配置”中所述。

#### 从自动装配中排除一个bean

在每个bean的基础上，您可以从自动装配中排除bean。 在Spring的XML格式中，将&lt;bean /&gt;元素的autowire-candidate属性设置为false; 容器使特定的bean定义对自动装配基础结构不可用（包括注释样式配置，如@Autowired）。

_autowire-candidate属性旨在仅影响基于类型的自动装配。 它不会影响名称的显式引用，即使指定的bean未标记为autowire候选，也会解析它。 因此，如果名称匹配，按名称自动装配将注入bean。_

您还可以根据针对bean名称的模式匹配来限制autowire候选者。 顶级&lt;beans /&gt;元素在其default-autowire-candidates属性中接受一个或多个模式。 例如，要将autowire候选状态限制为名称以Repository结尾的任何bean，请提供值\* Repository。 要提供多个模式，请在逗号分隔的列表中定义它们。 bean定义autowire-candidate属性的显式值true或false始终优先，对于此类bean，模式匹配规则不适用。

这些技术对于您永远不希望通过自动装配注入其他bean的bean非常有用。 这并不意味着排除的bean本身不能使用自动装配进行配置。 相反，bean本身不是自动装配其他bean的候选者。

### 7.4.6 方法注入

在大多数应用程序场景中，容器中的大多数bean都是单例。 当单例bean需要与另一个单例bean协作，或者非单例bean需要与另一个非单例bean协作时，通常通过将一个bean定义为另一个bean的属性来处理依赖关系。 当bean生命周期不同时会出现问题。 假设单例bean A需要使用非单例（原型）bean B，可能是在A上的每个方法调用上。容器只创建一次单例bean A，因此只有一次机会来设置属性。 每次需要时，容器都不能为bean A提供bean B的新实例。

解决方案是放弃一些控制反转。 您可以通过实现ApplicationContextAware接口使bean A了解容器，并通过对容器进行getBean（“B”）调用，每次bean A需要时都要求（通常是新的）bean B实例。 以下是此方法的示例：

```
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

前面的内容是不可取的，因为业务代码知道并耦合到Spring Framework。 Method Injection是Spring IoC容器的一个高级功能，它允许以干净的方式处理这个用例。

You can read more about the motivation for Method Injection in[this blog entry](https://spring.io/blog/2004/08/06/method-injection/).

#### lookup方法注入

Lookup方法注入是容器覆盖容器托管bean上的方法的能力，以返回容器中另一个命名bean的查找结果。 查找通常涉及原型bean，如上一节中描述的场景。 Spring Framework通过使用CGLIB库中的字节码生成来实现此方法注入，以动态生成覆盖该方法的子类。

* 要使这个动态子类工作，Spring bean容器将子类化的类不能是final，并且要重写的方法也不能是final。
* 对具有抽象方法的类进行单元测试需要您自己对类进行子类化，并提供抽象方法的存根实现。
* 组件扫描也需要具体方法，这需要具体的类别来获取。
* 另一个关键限制是查找方法不适用于工厂方法，特别是配置类中的@Bean方法，因为容器在这种情况下不负责创建实例，因此无法创建运行时生成的子类on the fly。

查看前面代码片段中的CommandManager类，您会看到Spring容器将动态覆盖createCommand（）方法的实现。 您的CommandManager类将不具有任何Spring依赖项，如重新编写的示例中所示：

```
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

在包含要注入的方法的客户机类（本例中为CommandManager）中，要注入的方法需要以下形式的签名：

```
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法是抽象的，则动态生成的子类实现该方法。 否则，动态生成的子类将覆盖原始类中定义的具体方法。 例如：

```
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

标识为commandManager的bean在需要myCommand bean的新实例时调用自己的方法createCommand（）。 您必须小心将myCommand bean部署为原型，如果这实际上是需要的话。 如果它是一个单例，则每次都返回myCommand bean的相同实例。

或者，在基于注释的组件模型中，您可以通过@Lookup批注声明查找方法：

```
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

或者，更具惯用性，您可以依赖于针对查找方法的声明返回类型解析目标bean：

```
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```

请注意，您通常会使用具体的存根实现来声明这种带注释的查找方法，以使它们与Spring的组件扫描规则兼容，其中默认情况下抽象类被忽略。 此限制不适用于显式注册或显式导入的bean类。

_访问不同范围的目标bean的另一种方法是ObjectFactory / Provider注入点。 查看名为“Scoped beans as dependencies”的部分。_

_感兴趣的读者也可以找到ServiceLocatorFactoryBean（在org.springframework.beans.factory.config包中）。_

#### 任意方法更换

与查找方法注入相比，一种不太有用的方法注入形式是能够使用另一个方法实现替换托管bean中的任意方法。 用户可以安全地跳过本节的其余部分，直到实际需要该功能。

使用基于XML的配置元数据，您可以使用被替换的方法元素将已有的方法实现替换为已部署的bean。 考虑以下类，使用方法computeValue，我们要覆盖它：

```
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }

    // some other methods...
}
```

实现org.springframework.beans.factory.support.MethodReplacer接口的类提供了新的方法定义。

```
/**
 * meant to be used to override the existing computeValue(String)
 * implementation in MyValueCalculator
 */
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```

部署原始类并指定方法覆盖的bean定义如下所示：

```
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```

您可以在&lt;replacement-method /&gt;元素中使用一个或多个包含的&lt;arg-type /&gt;元素来指示被覆盖的方法的方法签名。 仅当方法重载且类中存在多个变体时，才需要参数的签名。 为方便起见，参数的类型字符串可以是完全限定类型名称的子字符串。 例如，以下所有内容都匹配java.lang.String：

```
java.lang.String
String
Str
```

因为参数的数量通常足以区分每个可能的选择，所以通过允许您只键入与参数类型匹配的最短字符串，此快捷方式可以节省大量的输入。

## 




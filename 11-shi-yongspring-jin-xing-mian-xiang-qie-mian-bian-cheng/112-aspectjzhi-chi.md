## 11.2  @AspectJ支持

@AspectJ指的是将方面声明为使用注释注释的常规Java类的样式。 作为AspectJ 5版本的一部分，AspectJ项目引入了@AspectJ样式。 Spring使用AspectJ提供的库解释与AspectJ 5相同的注释，用于切入点解析和匹配。 AOP运行时仍然是纯Spring AOP，并且不依赖于AspectJ编译器或weaver。

使用AspectJ编译器和weaver可以使用完整的AspectJ语言，并在第11.8节“将AspectJ与Spring应用程序一起使用”中进行了讨论。

### 11.2.1 启用@AspectJ支持

要在Spring配置中使用@AspectJ方面，您需要启用Spring支持，以便根据@AspectJ方面配置Spring AOP，并根据这些方面是否建议自动执行bean。 通过autoproxying我们的意思是，如果Spring确定bean被一个或多个方面建议，它将自动生成该bean的代理以拦截方法调用并确保根据需要执行建议。

可以使用XML或Java样式配置启用@AspectJ支持。 在任何一种情况下，您还需要确保AspectJ的aspectjweaver.jar库位于应用程序的类路径中（版本1.6.8或更高版本）。 该库可在AspectJ发行版的“lib”目录中或通过Maven Central存储库获得。

#### 使用Java配置启用@AspectJ支持

要使用Java @Configuration启用@AspectJ支持，请添加@EnableAspectJAutoProxy注释：

```
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}
```

#### 使用XML配置启用@AspectJ支持

要使用基于XML的配置启用@AspectJ支持，请使用aop：aspectj-autoproxy元素：

```
<aop:aspectj-autoproxy/>
```

这假定您正在使用第41章基于XML模式的配置中所述的模式支持。 有关如何在aop命名空间中导入标记，请参见第41.2.7节“aop模式”。

### 11.2.2 Declaring an aspect

在启用@AspectJ支持的情况下，在应用程序上下文中定义的任何bean都是一个@AspectJ方面的类（具有@Aspect注释）将由Spring自动检测并用于配置Spring AOP。 以下示例显示了非常有用的方面所需的最小定义：

应用程序上下文中的常规bean定义，指向具有@Aspect批注的bean类：

```
<bean id="myAspect" class="org.xyz.NotVeryUsefulAspect">
    <!-- configure properties of aspect here as normal -->
</bean>
```

和NotVeryUsefulAspect类定义，用org.aspectj.lang.annotation.Aspect注释注释;

```
package org.xyz;
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class NotVeryUsefulAspect {

}
```

方面（使用@Aspect注释的类）可能具有与任何其他类一样的方法和字段。 它们还可能包含切入点，建议和引入（类型间）声明。

您可以在Spring XML配置中将方面类注册为常规bean，或者通过类路径扫描自动检测它们 - 就像任何其他Spring管理的bean一样。 但是，请注意@Aspect注释不足以在类路径中自动检测：为此，您需要添加单独的@Component注释（或者根据Spring的组件扫描程序的规则添加符合条件的自定义构造型注释）。

在Spring AOP中，不可能将方面本身作为其他方面的建议目标。 类上的@Aspect注释将其标记为方面，因此将其从自动代理中排除。

### 11.2.3 宣布切入点 Declaring a pointcut

回想一下，切入点确定了感兴趣的连接点，从而使我们能够控制建议何时执行。 Spring AOP仅支持Spring bean的方法执行连接点，因此您可以将切入点视为匹配Spring bean上方法的执行。 切入点声明有两个部分：一个包含名称和任何参数的签名，以及一个精确确定我们感兴趣的方法执行的切入点表达式。在AOP的@AspectJ注释样式中，切入点签名由常规方法提供 定义，并使用@Pointcut注释指示切入点表达式（用作切入点签名的方法必须具有void返回类型）。

一个示例将有助于区分切入点签名和切入点表达式。 以下示例定义名为“anyOldTransfer”的切入点，该切入点将匹配名为“transfer”的任何方法的执行：

```
@Pointcut("execution(* transfer(..))")// the pointcut expression
private void anyOldTransfer() {}// the pointcut signature
```

形成@Pointcut注释值的切入点表达式是常规的AspectJ 5切入点表达式。 有关AspectJ的切入点语言的完整讨论，请参阅AspectJ编程指南（以及扩展，AspectJ 5开发人员笔记本）或AspectJ上的一本书，例如Colyer等的“Eclipse AspectJ”。人。 或Ramnivas Laddad的“AspectJ in Action”。

#### 支持的切入点指示符

Spring AOP支持以下AspectJ切入点指示符（PCD）用于切入点表达式：

其他切入点类型

完整的AspectJ切入点语言支持Spring中不支持的其他切入点指示符。 它们是：call，get，set，preinitialization，staticinitialization，initialization，handler，adviceexecution，withincode，cflow，cflowbelow，if，@ this和@withincode。 在Spring AOP解释的切入点表达式中使用这些切入点指示符将导致抛出IllegalArgumentException。

Spring AOP支持的切入点指示符集可以在将来的版本中进行扩展，以支持更多的AspectJ切入点指示符。

* _execution_ - 对于匹配方法执行连接点，这是在使用Spring AOP时将使用的主要切入点指示符
* within  - 限制匹配某些类型中的连接点（只是在使用Spring AOP时执行在匹配类型中声明的方法）
* this  - 限制匹配连接点（使用Spring AOP时执行方法），其中bean引用（Spring AOP代理）是给定类型的实例
* target  - 限制匹配到连接点（使用Spring AOP时执行方法），其中目标对象（被代理的应用程序对象）是给定类型的实例
* args  - 限制匹配连接点（使用Spring AOP时执行方法），其中参数是给定类型的实例
* @target  - 限制匹配到连接点（使用Spring AOP时执行方法），其中执行对象的类具有给定类型的注释
* @args  - 限制匹配到连接点（使用Spring AOP时执行方法），其中传递的实际参数的运行时类型具有给定类型的注释
* @within  - 限制匹配以连接具有给定注释的类型中的点（使用Spring AOP时在具有给定注释的类型中声明的方法的执行）
* @annotation  - 限制连接点的匹配，其中连接点的主题（在Spring AOP中执行的方法）具有给定的注释

由于Spring AOP仅限制与方法执行连接点的匹配，因此上面对切入点指示符的讨论给出了比在AspectJ编程指南中找到的更窄的定义。 此外，AspectJ本身具有基于类型的语义，并且在执行连接点，this和target都引用同一个对象 - 执行该方法的对象。 Spring AOP是一个基于代理的系统，它区分代理对象本身（绑定到此）和代理后面的目标对象（绑定到目标）。

由于Spring的AOP框架基于代理的特性，目标对象内的调用根据定义不会被截获。对于JDK代理，只能拦截代理上的公共接口方法调用。使用CGLIB，代理上的公共和受保护方法调用将被拦截，甚至包括必要的包可见方法。但是，通过代理进行的常见交互应始终通过公共签名进行设计。

请注意，切入点定义通常与任何截获的方法匹配。如果切入点严格意义上是公开的，即使在通过代理进行潜在非公共交互的CGLIB代理方案中，也需要相应地定义切入点。

如果你的拦截需要包括方法调用甚至是目标类中的构造函数，那么考虑使用Spring驱动的原生AspectJ编织而不是Spring的基于代理的AOP框架。这构成了具有不同特征的不同AOP使用模式，因此在做出决定之前一定要先熟悉编织。

Spring AOP还支持另一个名为bean的PCD。 此PCD允许您限制连接点与特定命名的Spring bean或一组命名的Spring bean（使用通配符时）的匹配。 bean PCD具有以下形式：

```
bean(idOrNameOfBean)
```

idOrNameOfBean标记可以是任何Spring bean的名称：提供了使用\*字符的有限通配符支持，因此如果为Spring bean建立一些命名约定，则可以非常轻松地编写bean PCD表达式来挑选它们。 与其他切入点指示符的情况一样，bean PCD可以被&&'，\|\|'和！ （否定）也是。

请注意，bean PCD仅在Spring AOP中受支持 - 而不是在本机AspectJ编织中。 它是AspectJ定义的标准PCD的Spring特定扩展，因此不适用于@Aspect模型中声明的方面。

bean PCD在实例级别（基于Spring bean名称概念）而不是仅在类型级别（这是基于编织的AOP限制）运行。 基于实例的切入点指示符是Spring基于代理的AOP框架的一种特殊功能，它与Spring bean工厂紧密集成，通过名称可以自然而直接地识别特定的bean。

#### 结合切入点表达式

可以使用'&&'，'\|\|'组合切入点表达式 和'！'。 也可以通过名称引用切入点表达式。 以下示例显示了三个切入点表达式：anyPublicOperation（如果方法执行连接点表示任何公共方法的执行，则匹配）; inTrading（如果方法执行在交易模块中，则匹配）和tradingOperation（如果方法执行代表交易模块中的任何公共方法，则匹配）。

```
@Pointcut("execution(public * *(..))")
private void anyPublicOperation() {}

@Pointcut("within(com.xyz.someapp.trading..*)")
private void inTrading() {}

@Pointcut("anyPublicOperation() && inTrading()")
private void tradingOperation() {}
```

如上所示，最佳实践是从较小的命名组件构建更复杂的切入点表达式。 当按名称引用切入点时，将应用常规Java可见性规则（您可以看到相同类型的私有切入点，层次结构中受保护的切入点，任何地方的公共切入点等等）。 可见性不会影响切入点匹配。

#### 共享通用切入点定义

使用企业应用程序时，您经常需要从几个方面引用应用程序的模块和特定的操作集。 我们建议定义一个“SystemArchitecture”方面，为此目的捕获常见的切入点表达式。 典型的这种方面看起来如下：

```
package com.xyz.someapp;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class SystemArchitecture {

    /**
     * A join point is in the web layer if the method is defined
     * in a type in the com.xyz.someapp.web package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.web..*)")
    public void inWebLayer() {}

    /**
     * A join point is in the service layer if the method is defined
     * in a type in the com.xyz.someapp.service package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.service..*)")
    public void inServiceLayer() {}

    /**
     * A join point is in the data access layer if the method is defined
     * in a type in the com.xyz.someapp.dao package or any sub-package
     * under that.
     */
    @Pointcut("within(com.xyz.someapp.dao..*)")
    public void inDataAccessLayer() {}

    /**
     * A business service is the execution of any method defined on a service
     * interface. This definition assumes that interfaces are placed in the
     * "service" package, and that implementation types are in sub-packages.
     *
     * If you group service interfaces by functional area (for example,
     * in packages com.xyz.someapp.abc.service and com.xyz.someapp.def.service) then
     * the pointcut expression "execution(* com.xyz.someapp..service.*.*(..))"
     * could be used instead.
     *
     * Alternatively, you can write the expression using the 'bean'
     * PCD, like so "bean(*Service)". (This assumes that you have
     * named your Spring service beans in a consistent fashion.)
     */
    @Pointcut("execution(* com.xyz.someapp..service.*.*(..))")
    public void businessService() {}

    /**
     * A data access operation is the execution of any method defined on a
     * dao interface. This definition assumes that interfaces are placed in the
     * "dao" package, and that implementation types are in sub-packages.
     */
    @Pointcut("execution(* com.xyz.someapp.dao.*.*(..))")
    public void dataAccessOperation() {}

}
```

在这样的方面定义的切入点可以被引用到您需要切入点表达式的任何地方。 例如，要使服务层具有事务性，您可以编写：

```
<aop:config>
    <aop:advisor
        pointcut="com.xyz.someapp.SystemArchitecture.businessService()"
        advice-ref="tx-advice"/>
</aop:config>

<tx:advice id="tx-advice">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```

&lt;aop：config&gt;和&lt;aop：advisor&gt;元素将在第11.3节“基于模式的AOP支持”中讨论。 第17章“事务管理”中讨论了事务元素。

#### 例子

Spring AOP用户可能最常使用执行切入点指示符。 执行表达式的格式为：

```
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern)
```

除返回类型模式（上面的代码段中的ret-type-pattern），名称模式和参数模式之外的所有部分都是可选的。返回类型模式确定方法的返回类型必须是什么才能匹配连接点。最常见的是，您将使用\*作为返回类型模式，它匹配任何返回类型。仅当方法返回给定类型时，完全限定类型名称才匹配。名称模式与方法名称匹配。您可以使用\*通配符作为名称模式的全部或部分。如果指定声明类型模式，则包括尾随。将其加入名称模式组件。参数模式稍微复杂一些:\(）匹配不带参数的方法，而（..）匹配任意数量的参数（零或更多）。模式（\*）匹配采用任何类型的一个参数的方法，（\*，String）匹配采用两个参数的方法，第一个可以是任何类型，第二个必须是String。有关更多信息，请参阅AspectJ编程指南的语言语义部分。

下面给出了常见切入点表达式的一些示例。

* the execution of any public method:

```
execution(public * *(..))
```

* the execution of any method with a name beginning with "set":

```
execution(* set*(..))
```

* the execution of any method defined by the

```
execution(* com.xyz.service.AccountService.*(..))
```

* the execution of any method defined in the service package:

```
execution(* com.xyz.service.*.*(..))
```

* the execution of any method defined in the service package or a sub-package:

```
execution(* com.xyz.service..*.*(..))
```

* 服务包中的任何连接点（仅在Spring AOP中执行方法）：

```
within(com.xyz.service.*)
```

* any join point \(method execution only in Spring AOP\) within the service package or a sub-package:

```
within(com.xyz.service..*)
```

* 代理实现AccountService接口的任何连接点（仅在Spring AOP中执行方法）：

```
this(com.xyz.service.AccountService)
```

'this'更常用于绑定形式： - 请参阅以下有关如何在建议体中提供代理对象的建议。

* 目标对象实现AccountService接口的任何连接点（仅在Spring AOP中执行方法）：

```
target(com.xyz.service.AccountService)
```

'target'更常用于绑定形式： - 请参阅以下有关如何在建议体中提供目标对象的advice。

* 任何连接点（仅在Spring AOP中执行的方法），它接受一个参数，并且在运行时传递的参数是Serializable：

```
args(java.io.Serializable)
```

'args'更常用于绑定形式： - 请参阅以下有关如何在建议体中提供方法参数的建议。

请注意，此示例中给出的切入点与执行不同（\* \*（java.io.Serializable））：如果在运行时传递的参数是Serializable，则args版本匹配，如果方法签名声明单个参数，则执行版本匹配 类型Serializable。

* 任何连接点（仅在Spring AOP中执行的方法），其中目标对象具有@Transactional注释：

```
@target(org.springframework.transaction.annotation.Transactional)
```

'@target'也可以用于绑定形式： - 请参阅以下有关如何在建议体中提供注释对象的建议。

* 任何连接点（仅在Spring AOP中执行方法），其中目标对象的声明类型具有@Transactional注释：

```
@within(org.springframework.transaction.annotation.Transactional)
```

'@within'也可以用于绑定形式： - 请参阅以下有关如何在建议体中提供注释对象的建议。

* 任何连接点（仅在Spring AOP中执行方法），其中执行方法具有@Transactional注释：

```
@annotation(org.springframework.transaction.annotation.Transactional)
```

'@annotation'也可以用于绑定形式： - 请参阅以下有关如何在建议体中提供注释对象的建议。

* 任何连接点（仅在Spring AOP中执行的方法），它接受一个参数，并且传递的参数的运行时类型具有@Classified注释：

```
@args(com.xyz.security.Classified)
```

'@args'也可以用于绑定形式： - 请参阅以下有关如何在建议体中提供注释对象的建议。

* 名为tradeService的Spring bean上的任何连接点（仅在Spring AOP中执行方法）：

```
bean(tradeService)
```

* 具有与通配符表达式\* Service匹配的名称的Spring bean上的任何连接点（仅在Spring AOP中执行方法）：

```
bean(*Service)
```

#### 写出好的切入点

在编译期间，AspectJ处理切入点以尝试和优化匹配性能。检查代码并确定每个连接点是否（静态地或动态地）匹配给定切入点是一个代价高昂的过程。 （动态匹配意味着无法通过静态分析完全确定匹配，并且将在代码中放置测试以确定代码运行时是否存在实际匹配）。在第一次遇到切入点声明时，AspectJ会将其重写为匹配过程的最佳形式。这是什么意思？基本上，切入点在DNF（析取范式）中重写，并且切入点的组件被排序，以便首先检查那些评估成本更低的组件。这意味着您不必担心了解各种切入点指示符的性能，并且可以在切入点声明中以任何顺序提供它们。

但是，AspectJ只能处理它所说的内容，并且为了获得最佳匹配性能，您应该考虑它们想要实现的目标，并在定义中尽可能缩小匹配的搜索空间。现有的指示符自然分为三组：kinded，scoping和context：

* Kinded指示符是选择特定类型的连接点的指示符。 例如：执行，获取，设置，调用，处理程序
* 范围界定指示符是那些选择一组感兴趣的连接点（可能是多种类型）的指示符。 例如：内部，内部代码
* 上下文指示符是基于上下文匹配（并且可选地绑定）的指示符。 例如：this，target，@ annotation

一个写得很好的切入点应该尝试包括至少前两种类型（kinded和scoping），而如果希望基于连接点上下文匹配，则可以包括上下文指示符，或者绑定该上下文以在建议中使用。 仅提供一个kinded指示符或仅提供上下文指示符将起作用，但由于所有额外的处理和分析，可能会影响编织性能（使用的时间和内存）。 范围界定指示符非常快速匹配，它们的使用意味着AspectJ可以非常快速地消除不应该进一步处理的连接点组 - 这就是为什么一个好的切入点应该总是包括一个如果可能的原因。

### 11.2.4 声明advice

建议与切入点表达式相关联，并在切入点匹配的方法执行之前，之后或周围运行。 切入点表达式可以是对命名切入点的简单引用，也可以是在适当位置声明的切入点表达式。

#### Before advice

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }

}
```

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class BeforeExample {

    @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }

}
```

#### 回复建议后 After returning advice

返回建议后，匹配的方法执行正常返回。 它是使用@AfterReturning注释声明的：

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doAccessCheck() {
        // ...
    }

}
```

注意：当然可以在同一方面内有多个建议声明和其他成员。 我们只是在这些例子中展示了一个建议声明，专注于当时正在讨论的问题。

有时您需要在建议体中访问返回的实际值。 您可以使用绑定返回值的@AfterReturning形式：

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterReturning;

@Aspect
public class AfterReturningExample {

    @AfterReturning(
        pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
        returning="retVal")
    public void doAccessCheck(Object retVal) {
        // ...
    }

}
```

  
returns属性中使用的名称必须与advice方法中的参数名称相对应。 当方法执行返回时，返回值将作为相应的参数值传递给advice方法。 返回子句还将匹配仅限于那些返回指定类型值的方法执行（在这种情况下为Object，它将匹配任何返回值）。

请注意，在使用返回后的建议时，无法返回完全不同的参考。

#### After throwing advice

抛出建议运行时，匹配的方法执行通过抛出异常退出。 它是使用@AfterThrowing注释声明的：

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doRecoveryActions() {
        // ...
    }

}
```

通常，您希望仅在抛出给定类型的异常时才运行建议，并且您还经常需要访问建议体中的抛出异常。 使用throwing属性来限制匹配（如果需要，使用Throwable作为异常类型），并将抛出的异常绑定到advice参数。

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;

@Aspect
public class AfterThrowingExample {

    @AfterThrowing(
        pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
        throwing="ex")
    public void doRecoveryActions(DataAccessException ex) {
        // ...
    }

}
```

throw属性中使用的名称必须与advice方法中的参数名称相对应。 当通过抛出异常退出方法时，异常将作为相应的参数值传递给advice方法。 throw子句还将匹配仅限于那些抛出指定类型异常的方法执行（在本例中为DataAccessException）。

#### 经过（终于）建议 After \(finally\) advice

在（最终）建议运行之后，匹配的方法执行退出。 它是使用@After注释声明的。 在建议必须准备好处理正常和异常返回条件。 它通常用于释放资源等。

```
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.After;

@Aspect
public class AfterFinallyExample {

    @After("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
    public void doReleaseLock() {
        // ...
    }

}
```

#### Around advice



#### Advice parameters

#### Access to the current JoinPoint

#### Passing parameters to advice

#### Advice parameters and generics 泛型

#### Determining argument names 确定参数名称

#### Proceeding with arguments 

#### Advice ordering

  


### 11.2.5 简介

### 11.2.6  Aspect实例化模型

### 11.2.7 例




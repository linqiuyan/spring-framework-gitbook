## 12.1 介绍

前一章描述了Spring使用@AspectJ和基于模式的方面定义对AOP的支持。 在本章中，我们将讨论较低级别的Spring AOP API以及Spring 1.2应用程序中使用的AOP支持。 对于新应用程序，我们建议使用前一章中描述的Spring 2.0及更高版本的AOP支持，但在使用现有应用程序时，或者在阅读书籍和文章时，您可能会遇到Spring 1.2样式示例。 Spring 4.0向后兼容Spring 1.2，本章中描述的所有内容在Spring 4.0中都得到了完全支持。

## 12.2  Spring中的Pointcut API

让我们来看看Spring如何处理关键的切入点概念。

### 12.2.1 概念

Spring的切入点模型使切入点重用独立于建议类型。 可以使用相同的切入点来定位不同的建议。

org.springframework.aop.Pointcut接口是中央接口，用于将建议定位到特定的类和方法。 完整的界面如下所示：

```
public interface Pointcut {

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();

}
```

将Pointcut接口拆分为两部分允许重用类和方法匹配部分，以及细粒度合成操作（例如与另一个方法匹配器执行“联合”）。

ClassFilter接口用于将切入点限制为给定的一组目标类。 如果matches（）方法始终返回true，则将匹配所有目标类：

```
public interface ClassFilter {

    boolean matches(Class clazz);
}
```

MethodMatcher接口通常更重要。 完整的界面如下所示：

```
public interface MethodMatcher {

    boolean matches(Method m, Class targetClass);

    boolean isRuntime();

    boolean matches(Method m, Class targetClass, Object[] args);
}
```

匹配（Method，Class）方法用于测试此切入点是否与目标类上的给定方法匹配。 可以在创建AOP代理时执行此评估，以避免对每个方法调用进行测试。 如果2参数matches方法对给定方法返回true，并且MethodMatcher的isRuntime（）方法返回true，则将在每次方法调用时调用3参数匹配方法。 这使切入点能够在执行目标通知之前立即查看传递给方法调用的参数。

大多数MethodMatchers都是静态的，这意味着它们的isRuntime（）方法返回false。 在这种情况下，永远不会调用3参数匹配方法。

如果可能，尝试使切入点成为静态，允许AOP框架在创建AOP代理时缓存切入点评估的结果。

### 12.2.2 切入点的操作

Spring支持对切入点的操作：特别是联合和交集。

* Union表示切入点匹配的方法。
* 交叉表示两个切入点匹配的方法。
* 联盟通常更有用。
* 可以使用org.springframework.aop.support.Pointcuts类中的静态方法或使用同一包中的ComposablePointcut类来组合切入点。 但是，使用AspectJ切入点表达式通常是一种更简单的方法。

### 12.2.3  AspectJ表达式切入点

从2.0开始，Spring使用的最重要的切入点类型是org.springframework.aop.aspectj.AspectJExpressionPointcut。 这是一个切入点，它使用AspectJ提供的库来解析AspectJ切入点表达式字符串。

有关受支持的AspectJ切入点基元的讨论，请参见上一章。

### 12.2.4 便利切入点实现

Spring提供了几种方便的切入点实现。 有些可以开箱即用; 其他的意图是在特定于应用程序的切入点中进行子类化。

#### 静态切入点

静态切入点基于方法和目标类，不能考虑方法的参数。 静态切入点对于大多数用途来说足够 - 而且最好。 当第一次调用方法时，Spring可能只评估一次静态切入点：之后，不需要再次使用每个方法调用来评估切入点。

让我们考虑Spring中包含的一些静态切入点实现。

#### 动态切入点

指定静态切入点的一种显而易见的方法是正则表达式。 除Spring之外的几个AOP框架使这成为可能。 org.springframework.aop.support.JdkRegexpMethodPointcut是一个通用的正则表达式切入点，使用JDK 1.4+中的正则表达式支持。

使用JdkRegexpMethodPointcut类，您可以提供模式字符串列表。 如果其中任何一个匹配，则切入点将评估为true。 （所以结果实际上是这些切入点的结合。）

用法如下所示：

```
<bean id="settersAndAbsquatulatePointcut"
        class="org.springframework.aop.support.JdkRegexpMethodPointcut">
    <property name="patterns">
        <list>
            <value>.*set.*</value>
            <value>.*absquatulate</value>
        </list>
    </property>
</bean>
```

Spring提供了一个便利类RegexpMethodPointcutAdvisor，它允许我们引用一个建议（记住建议可以是一个拦截器，建议之前，抛出建议等）。 在幕后，Spring将使用JdkRegexpMethodPointcut。 使用RegexpMethodPointcutAdvisor简化了布线，因为一个bean封装了切入点和建议，如下所示：

```
<bean id="settersAndAbsquatulateAdvisor"
        class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <property name="advice">
        <ref bean="beanNameOfAopAllianceInterceptor"/>
    </property>
    <property name="patterns">
        <list>
            <value>.*set.*</value>
            <value>.*absquatulate</value>
        </list>
    </property>
</bean>
```

RegexpMethodPointcutAdvisor可以与任何建议类型一起使用。

#### 属性驱动的切入点 Attribute-driven pointcuts

一种重要的静态切入点是元数据驱动的切入点。 这使用元数据属性的值：通常是源级元数据。

#### Dynamic pointcuts

与静态切入点相比，动态切入点的评估成本更高。 它们考虑了方法参数以及静态信息。 这意味着必须使用每个方法调用来评估它们; 参数不能缓存，因为参数会有所不同。

主要的例子是控制流切入点。

#### Control flow pointcuts

Spring控制流切入点在概念上类似于AspectJ cflow切入点，虽然功能较弱。 （目前无法指定切入点在另一个切入点匹配的连接点下执行。）控制流切入点与当前调用堆栈匹配。 例如，如果通过com.mycompany.web包中的方法或SomeCaller类调用连接点，则可能会触发它。 使用org.springframework.aop.support.ControlFlowPointcut类指定控制流切入点。

在运行时评估控制流切入点的成本远远高于其他动态切入点。 在Java 1.4中，成本大约是其他动态切入点的5倍。

### 12.2.5 切入点超类 Pointcut superclasses

Spring提供了有用的切入点超类来帮助您实现自己的切入点。

因为静态切入点最有用，所以您可能会将StaticMethodMatcherPointcut子类化，如下所示。 这需要实现一个抽象方法（尽管可以覆盖其他方法来自定义行为）：

```
class TestStaticPointcut extends StaticMethodMatcherPointcut {

    public boolean matches(Method m, Class targetClass) {
        // return true if custom criteria match
    }
}
```

还有动态切入点的超类。

您可以在Spring 1.0 RC2及更高版本中使用任何建议类型的自定义切入点。

### 12.2.6 自定义切入点

因为Spring AOP中的切入点是Java类，而不是语言功能（如在AspectJ中），所以可以声明自定义切入点，无论是静态还是动态。 Spring中的自定义切入点可以是任意复杂的。 但是，如果可能，建议使用AspectJ切入点表达式语言。

\[注意\]

更高版本的Spring可能会支持JAC提供的“语义切入点”：例如，“所有改变目标对象中实例变量的方法”。

## 12.3  Spring中的建议API

现在让我们来看看Spring AOP如何处理建议。

### 12.3.1 Advice生命周期

每个建议都是一个Spring bean。 建议实例可以在所有建议对象之间共享，也可以对每个建议对象唯一。 这对应于每个类或每个实例的建议。

每类建议最常使用。 它适用于交易顾问等通用建议。 这些不依赖于代理对象的状态或添加新状态; 他们只是按照方法和论点行事。

每个实例的建议适用于介绍，以支持mixin。 在这种情况下，建议将状态添加到代理对象。

可以在同一个AOP代理中混合使用共享和每个实例的建议。

### 12.3.2  Spring中的advice类型

Spring提供了几种开箱即用的建议类型，并且可以扩展以支持任意建议类型。 让我们看看基本概念和标准建议类型。

#### 拦截建议

Spring中最基本的建议类型是拦截建议。

Spring符合AOP Alliance接口，可以使用方法拦截来获取建议。 实现around建议的MethodInterceptors应该实现以下接口：

```java
public interface MethodInterceptor extends Interceptor {

    Object invoke(MethodInvocation invocation) throws Throwable;
}
```

invoke（）方法的MethodInvocation参数公开了被调用的方法; 目标连接点; AOP代理; 和方法的参数。 invoke（）方法应该返回调用的结果：连接点的返回值。

一个简单的MethodInterceptor实现如下所示：

```
public class DebugInterceptor implements MethodInterceptor {

    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("Before: invocation=[" + invocation + "]");
        Object rval = invocation.proceed();
        System.out.println("Invocation returned");
        return rval;
    }
}
```

请注意对MethodInvocation的proceed（）方法的调用。 这沿拦截器链向下进入连接点。 大多数拦截器都会调用此方法，并返回其返回值。 但是，与任何around建议一样，MethodInterceptor可以返回不同的值或抛出异常，而不是调用proceed方法。 但是，你没有充分的理由不想这样做！

\[注意\]

MethodInterceptors提供与其他符合AOP Alliance标准的AOP实现的互操作性。 本节其余部分讨论的其他建议类型实现了常见的AOP概念，但是采用Spring特定的方式。 虽然使用最具体的建议类型有一个优势，但如果您可能希望在另一个AOP框架中运行该方面，请坚持使用MethodInterceptor建议。 请注意，切入点目前在框架之间不可互操作，AOP联盟目前不定义切入点接口。

#### Before advice

更简单的建议类型是之前的建议。 这不需要MethodInvocation对象，因为它只在进入方法之前被调用。

之前建议的主要优点是不需要调用proceed（）方法，因此不会无意中无法继续拦截链。

MethodBeforeAdvice接口如下所示。 （Spring的API设计允许在建议之前提供字段，尽管通常的对象适用于字段拦截，并且Spring不太可能实现它）。

```
public interface MethodBeforeAdvice extends BeforeAdvice {

    void before(Method m, Object[] args, Object target) throws Throwable;
}
```

请注意，返回类型为void。 在建议之前可以在连接点执行之前插入自定义行为，但不能更改返回值。 如果before advice抛出异常，这将中止拦截器链的进一步执行。 异常将传播回拦截链。 如果未选中，或者在被调用方法的签名上，它将直接传递给客户端; 否则它将被AOP代理包装在未经检查的异常中。

Spring中一个before建议的示例，它计算所有方法调用：

```java
public class CountingBeforeAdvice implements MethodBeforeAdvice {

    private int count;

    public void before(Method m, Object[] args, Object target) throws Throwable {
        ++count;
    }

    public int getCount() {
        return count;
    }
}
```

\[提示\]

之前建议可以与任何切入点一起使用。

#### Throws advice

如果连接点引发异常，则在返回连接点后调用抛出建议。 Spring提供类型投掷建议。 请注意，这意味着org.springframework.aop.ThrowsAdvice接口不包含任何方法：它是一个标记接口，用于标识给定对象实现一个或多个类型化throws建议方法。 这些应该是以下形式：

```
afterThrowing([Method, args, target], subclassOfThrowable)
```

只需要最后一个参数。 方法签名可以有一个或四个参数，具体取决于通知方法是否对方法和参数感兴趣。 以下类是throws建议的示例。

如果抛出RemoteException（包括子类），则调用以下建议：

```
public class RemoteThrowsAdvice implements ThrowsAdvice {

    public void afterThrowing(RemoteException ex) throws Throwable {
        // Do something with remote exception
    }
}
```

如果抛出ServletException，则调用以下建议。 与上面的建议不同，它声明了4个参数，因此它可以访问被调用的方法，方法参数和目标对象：

```
public class ServletThrowsAdviceWithArguments implements ThrowsAdvice {

    public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
        // Do something with all arguments
    }
}
```

最后一个示例说明了如何在单个类中使用这两个方法，该类处理RemoteException和ServletException。 可以在单个类中组合任意数量的throws建议方法。

```
public static class CombinedThrowsAdvice implements ThrowsAdvice {

    public void afterThrowing(RemoteException ex) throws Throwable {
        // Do something with remote exception
    }

    public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
        // Do something with all arguments
    }
}
```

\[注意\]

如果throws-advice方法本身抛出异常，它将覆盖原始异常（即更改抛出给用户的异常）。 覆盖异常通常是RuntimeException; 这与任何方法签名兼容。 但是，如果throws-advice方法抛出一个已检查的异常，则它必须匹配目标方法的声明异常，因此在某种程度上耦合到特定的目标方法签名。 不要抛出与目标方法签名不兼容的未声明的已检查异常！

\[提示\]

抛出建议可以与任何切入点一起使用。

#### After Returning advice

在Spring中返回后的建议必须实现org.springframework.aop.AfterReturningAdvice接口，如下所示：

```
public interface AfterReturningAdvice extends Advice {

    void afterReturning(Object returnValue, Method m, Object[] args, Object target)
            throws Throwable;
}
```

返回后的建议可以访问返回值（它无法修改），调用方法，方法参数和目标。

返回通知后的以下内容计算所有未抛出异常的成功方法调用：

```
public class CountingAfterReturningAdvice implements AfterReturningAdvice {

    private int count;

    public void afterReturning(Object returnValue, Method m, Object[] args, Object target)
            throws Throwable {
        ++count;
    }

    public int getCount() {
        return count;
    }
}
```

此建议不会更改执行路径。 如果它抛出异常，则会抛出拦截器链而不是返回值。

\[提示\]

返回建议后可以使用任何切入点。

#### 介绍advice

Spring将介绍建议视为一种特殊的拦截建议。

简介需要一个Introduction Advisor和一个Introduction Interceptor，实现以下接口：

```
public interface IntroductionInterceptor extends MethodInterceptor {

    boolean implementsInterface(Class intf);
}
```

从AOP Alliance MethodInterceptor接口继承的invoke（）方法必须实现引入：即，如果被调用的方法在引入的接口上，则引入拦截器负责处理方法调用 - 它不能调用proceed（）。

引言建议不能与任何切入点一起使用，因为它仅适用于类，而不是方法，级别。 您只能在IntroductionAdvisor中使用介绍建议，其中包含以下方法：

```
public interface IntroductionAdvisor extends Advisor, IntroductionInfo {

    ClassFilter getClassFilter();

    void validateInterfaces() throws IllegalArgumentException;
}

public interface IntroductionInfo {

    Class[] getInterfaces();
}
```

没有MethodMatcher，因此没有与引入建议相关的Pointcut。 只有类过滤是合乎逻辑的。

getInterfaces（）方法返回此顾问程序引入的接口。

validateInterfaces（）方法在内部用于查看引入的接口是否可以由配置的IntroductionInterceptor实现。

让我们看一下Spring测试套件中的一个简单示例。 假设我们想要将以下接口引入一个或多个对象：

```
public interface Lockable {
    void lock();
    void unlock();
    boolean locked();
}
```

这说明了一个混合。我们希望能够将建议对象转换为Lockable，无论其类型如何，并调用锁定和解锁方法。如果我们调用lock（）方法，我们希望所有setter方法都抛出一个LockedException。因此，我们可以添加一个方面，提供使对象不可变的能力，而不需要它们知道它：AOP的一个很好的例子。

首先，我们需要一个可以完成繁重工作的IntroductionInterceptor。在这种情况下，我们扩展了org.springframework.aop.support.DelegatingIntroductionInterceptor方便类。我们可以直接实现IntroductionInterceptor，但使用DelegatingIntroductionInterceptor最适合大多数情况。

DelegatingIntroductionInterceptor旨在委托对引入的接口的实际实现的介绍，隐藏拦截的使用。可以使用构造函数参数将委托设置为任何对象;默认委托（当使用no-arg构造函数时）就是这个。因此，在下面的示例中，委托是DelegatingIntroductionInterceptor的LockMixin子类。给定委托（默认情况下），DelegatingIntroductionInterceptor实例查找委托实现的所有接口（除了IntroductionInterceptor），并支持对其中任何接口的介绍。 LockMixin等子类可以调用suppressInterface（Class intf）方法来抑制不应该公开的接口。但是，无论IntroductionInterceptor准备支持多少接口，使用的IntroductionAdvisor将控制实际公开的接口。引入的接口将隐藏目标对同一接口的任何实现。

因此，LockMixin扩展了DelegatingIntroductionInterceptor并实现了Lockable本身。超类自动选择可以支持Lockable引入，因此我们不需要指定。我们可以用这种方式引入任意数量的接口。

注意使用锁定的实例变量。这有效地将附加状态添加到目标对象中保存的状态。

```
public class LockMixin extends DelegatingIntroductionInterceptor implements Lockable {

    private boolean locked;

    public void lock() {
        this.locked = true;
    }

    public void unlock() {
        this.locked = false;
    }

    public boolean locked() {
        return this.locked;
    }

    public Object invoke(MethodInvocation invocation) throws Throwable {
        if (locked() && invocation.getMethod().getName().indexOf("set") == 0) {
            throw new LockedException();
        }
        return super.invoke(invocation);
    }

}
```

通常没有必要覆盖invoke（）方法：DelegatingIntroductionInterceptor实现 - 如果引入方法则调用委托方法，否则进入连接点 - 通常就足够了。 在本例中，我们需要添加一个检查：如果处于锁定模式，则不能调用setter方法。

需要的介绍顾问很简单。 它需要做的只是持有一个独特的LockMixin实例，并指定引入的接口 - 在这种情况下，只是Lockable。 更复杂的示例可能会引用引入拦截器（它将被定义为原型）：在这种情况下，没有与LockMixin相关的配置，因此我们只需使用new创建它。

```
public class LockMixinAdvisor extends DefaultIntroductionAdvisor {

    public LockMixinAdvisor() {
        super(new LockMixin(), Lockable.class);
    }
}
```

我们可以非常简单地应用这个顾问：它不需要配置。 （但是，有必要：没有IntroductionAdvisor就不可能使用IntroductionInterceptor。）与介绍一样，顾问程序必须是每个实例，因为它是有状态的。 对于每个建议的对象，我们需要一个不同的LockMixinAdvisor实例，因此需要LockMixin。 顾问包括建议对象的状态的一部分。

我们可以使用Advised.addAdvisor（）方法或（建议的方式）以编程方式应用此顾问程序，与任何其他顾问程序一样。 下面讨论的所有代理创建选项，包括“自动代理创建器”，正确处理引入和有状态混合。

## 12.4  Spring中的Advisor API

## 12.5 使用ProxyFactoryBean创建AOP代理

### 12.5.1 基本

### 12.5.2  JavaBean属性

### 12.5.3 基于JDK和CGLIB的代理

### 12.5.4 代理接口

### 12.5.5 代理课程

### 12.5.6 使用“全球”顾问

## 12.6 简明的代理定义

## 12.7 使用ProxyFactory以编程方式创建AOP代理

## 12.8 操纵建议的对象

## 12.9 使用“自动代理”工具

### 12.9.1  Autoproxy bean定义

#### 的BeanNameAutoProxyCreator

#### DefaultAdvisorAutoProxyCreator的

#### AbstractAdvisorAutoProxyCreator

### 12.9.2 使用元数据驱动的自动代理

## 12.10 使用TargetSources

### 12.10.1 热插拔目标源

### 12.10.2 汇集目标来源

### 12.10.3 原型目标来源

### 12.10.4  ThreadLocal目标源

12.11 定义新的建议类型

12.12 更多资源


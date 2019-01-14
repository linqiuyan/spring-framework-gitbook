## 7.12 基于Java的容器配置

### 7.12.1 基本概念：@Bean和@Configuration

Spring新的Java配置支持中的中心工件是@Configuration-annotated类和@ Bean-annotated方法。

@Bean注释用于指示方法实例化，配置和初始化由Spring IoC容器管理的新对象。 对于那些熟悉Spring的&lt;beans /&gt; XML配置的人来说，@ Boan注释扮演的角色与&lt;bean /&gt;元素相同。 您可以将@Bean带注释的方法与任何Spring @Component一起使用，但是，它们最常用于@Configuration bean。

使用@Configuration注释类表示其主要目的是作为bean定义的源。 此外，@ Configuration类允许通过简单地调用同一个类中的其他@Bean方法来定义bean间依赖关系。 最简单的@Configuration类如下：

```
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

上面的AppConfig类将等效于以下Spring &lt;beans /&gt; XML：

```
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

---

完整@Configuration vs 'lite'@Bean模式？

当@Bean方法在未使用@Configuration注释的类中声明时，它们被称为以'lite'模式处理。 在@Component或甚至普通旧类中声明的Bean方法将被视为'lite'，其中包含类的主要目的不同，而@Bean方法只是一种奖励。 例如，服务组件可以通过在每个适用的组件类上使用额外的@Bean方法将管理视图公开给容器。 在这种情况下，@ Bean方法是一种简单的通用工厂方法机制。

与完整的@Configuration不同，lite @Bean方法不能声明bean间依赖关系。 相反，它们对其包含组件的内部状态进行操作，并且可选地对它们可以声明的参数进行操作。 因此，这样的@Bean方法不应该调用其他@Bean方法; 每个这样的方法实际上只是一个特定bean引用的工厂方法，没有任何特殊的运行时语义。 这里的积极副作用是不必在运行时应用CGLIB子类，因此在类设计方面没有限制（即包含类可能是最终的等等）。

在常见的场景中，@ Bean方法将在@Configuration类中声明，确保始终使用“完整”模式，因此交叉方法引用将被重定向到容器的生命周期管理。 这将防止通过常规Java调用意外地调用相同的@Bean方法，这有助于减少在“精简”模式下操作时难以跟踪的细微错误。

---

@Bean和@Configuration注释将在下面的部分中进行深入讨论。 首先，我们将介绍使用基于Java的配置创建弹簧容器的各种方法。

### 7.12.2 使用AnnotationConfigApplicationContext实例化Spring容器

#### 结构简单

#### 使用register（Class &lt;？&gt; ...）以编程方式构建容器

#### 使用扫描启用组件扫描（String ...）

#### 使用AnnotationConfigWebApplicationContext支持Web应用程序

### 7.12.3 使用@Bean注释

#### 声明一个bean

#### Bean依赖项

#### 接收生命周期回调

#### 指定bean范围

#### 自定义bean命名

#### Bean别名

#### bean的描述

### 7.12.4 使用@Configuration注释

#### 注入bean间依赖关系

#### 查找方法注入

#### 有关基于Java的配置如何在内部工作的更多信息

### 7.12.5 编写基于Java的配置

#### 使用@Import注释

#### 有条件地包括@Configuration类或@Bean方法

#### 结合Java和XML配置




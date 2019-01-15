## 7.16  BeanFactory

BeanFactory API为Spring的IoC功能提供了基础。它的特定契约主要用于与Spring的其他部分和相关的第三方框架集成，其DefaultListableBeanFactory实现是更高级别GenericApplicationContext容器中的密钥委托。

BeanFactory和相关接口（如BeanFactoryAware，InitializingBean，DisposableBean）是其他框架组件的重要集成点：不需要任何注释甚至反射，它们允许容器与其组件之间的非常有效的交互。应用程序级bean可以使用相同的回调接口，但通常更喜欢通过注释或通过编程配置进行声明性依赖注入。

请注意，核心BeanFactory API级别及其DefaultListableBeanFactory实现不会对配置格式或要使用的任何组件注释做出假设。所有这些风格都通过扩展（例如XmlBeanDefinitionReader和AutowiredAnnotationBeanPostProcessor）进行，在共享BeanDefinition对象上作为核心元数据表示进行操作。这是使Spring的容器如此灵活和可扩展的本质。

以下部分解释了BeanFactory和ApplicationContext容器级别之间的差异以及对引导的影响。

### 7.16.1  BeanFactory或ApplicationContext？

使用ApplicationContext，除非您有充分的理由不这样做，使用GenericApplicationContext及其子类AnnotationConfigApplicationContext作为自定义引导的常见实现。这些是Spring用于所有常见目的的核心容器的主要入口点：加载配置文件，触发类路径扫描，以编程方式注册bean定义和带注释的类。

因为ApplicationContext包含BeanFactory的所有功能，所以通常建议使用简单的BeanFactory，除了需要完全控制bean处理的场景。在诸如GenericApplicationContext实现的ApplicationContext中，将按惯例（即通过bean名称或bean类型）检测几种bean，特别是后处理器，而普通的DefaultListableBeanFactory对任何特殊bean都是不可知的。

对于许多扩展容器功能，例如注释处理和AOP代理，BeanPostProcessor扩展点是必不可少的。如果仅使用普通的DefaultListableBeanFactory，则默认情况下不会检测到并激活此类后处理器。这种情况可能会令人困惑，因为您的bean配置实际上没有任何问题;更确切地说，在这种情况下，需要通过其他设置完全引导容器。

下表列出了BeanFactory和ApplicationContext接口和实现提供的功能。

**Feature Matrix**

| Feature | `BeanFactory` | `ApplicationContext` |
| :--- | :--- | :--- |
| Bean instantiation/wiring | Yes | Yes |
| Integrated lifecycle management | No | Yes |
| Automatic`BeanPostProcessor`registration | No | Yes |
| Automatic`BeanFactoryPostProcessor`registration | No | Yes |
| Convenient`MessageSource`access \(for internalization\) | No | Yes |
| Built-in`ApplicationEvent`publication mechanism | No | Yes |

要使用DefaultListableBeanFactory显式注册bean后处理器，您需要以编程方式调用addBeanPostProcessor：

```
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// populate the factory with bean definitions

// now register any needed BeanPostProcessor instances
factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
factory.addBeanPostProcessor(new MyBeanPostProcessor());

// now start using the factory
```

要将BeanFactoryPostProcessor应用于普通的DefaultListableBeanFactory，需要调用其postProcessBeanFactory方法：

```
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));

// bring in some property values from a Properties file
PropertyPlaceholderConfigurer cfg = new PropertyPlaceholderConfigurer();
cfg.setLocation(new FileSystemResource("jdbc.properties"));

// now actually do the replacement
cfg.postProcessBeanFactory(factory);
```

在这两种情况下，显式注册步骤都不方便，这就是为什么各种ApplicationContext变体优先于Spring支持的应用程序中的普通DefaultListableBeanFactory，尤其是在典型企业设置中依赖BeanFactoryPostProcessors和BeanPostProcessors来扩展容器功能时。

AnnotationConfigApplicationContext具有开箱即用的所有通用注释后处理器，并且可以通过配置注释（例如@EnableTransactionManagement）在封面下引入额外的处理器。 在Spring的基于注释的配置模型的抽象级别，bean后处理器的概念变成仅仅是内部容器细节。

### 7.16.2 耦合代码与不良单例

最好以依赖注入（DI）方式编写大多数应用程序代码，其中该代码由Spring IoC容器提供，在创建容器时由容器提供自己的依赖关系，并且完全不知道容器。但是，对于有时需要将其他代码绑定在一起的小型胶代码层，有时需要对Spring IoC容器进行单例（或准单例）样式访问。例如，第三方代码可能会尝试直接构造新对象（Class.forName（）样式），而无法从Spring IoC容器中获取这些对象。如果由第三方代码构造的对象很小然后，存根或代理使用对Spring IoC容器的单一样式访问来获取委托的真实对象，然后对于大多数代码（来自容器的对象）仍然实现了控制的反转。因此，大多数代码仍然没有意识到容器或它是如何被访问的，并且仍然与其他代码分离，并带来所有后续的好处。 EJB还可以使用此存根/代理方法委派给从Spring IoC容器检索的普通Java实现对象。虽然Spring IoC容器本身在理想情况下不必是单例，但就内存使用或初始化时间（在Spring IoC容器中使用bean，例如Hibernate SessionFactory）而言，每个bean使用自己的bean时，它可能是不现实的，非单例Spring IoC容器。

在服务定位器样式中查找应用程序上下文有时是访问共享的Spring管理组件的唯一选项，例如在EJB 2.1环境中，或者您希望在WAR文件中共享单个ApplicationContext作为WebApplicationContexts的父项。在这种情况下，您应该考虑使用此Spring团队博客条目中描述的实用程序类ContextSingletonBeanFactoryLocator定位器。




## 7.10 类路径扫描和托管组件（Classpath scanning and managed components）

本章中的大多数示例都使用XML来指定在Spring容器中生成每个BeanDefinition的配置元数据。 上一节（第7.9节“基于注释的容器配置”）演示了如何通过源级注释提供大量配置元数据。 但是，即使在这些示例中，“基本”bean定义也在XML文件中明确定义，而注释仅驱动依赖项注入。 本节介绍通过扫描类路径隐式检测候选组件的选项。 候选组件是与过滤条件匹配的类，并且具有向容器注册的相应bean定义。 这消除了使用XML来执行bean注册的需要; 相反，您可以使用注释（例如@Component），AspectJ类型表达式或您自己的自定义筛选条件来选择哪些类将使用容器注册bean定义。

从Spring 3.0开始，Spring JavaConfig项目提供的许多功能都是核心Spring Framework的一部分。 这允许您使用Java而不是使用传统的XML文件来定义bean。 有关如何使用这些新功能的示例，请查看@Configuration，@ Both，@ Import和@DependsOn注释。

### 7.10.1 @Component和进一步的构造型注释（@Component and further stereotype annotations）

@Repository注释是任何满足存储库角色或构造型（也称为数据访问对象或DAO）的类的标记。该标记的用途包括自动翻译异常，如第20.2.2节“异常翻译”中所述。

Spring提供了进一步的构造型注释：@Component，@Service和@Controller。 @Component是任何Spring管理组件的通用构造型。 @Repository，@Service和@Controller是@Component的特化，用于更具体的用例，例如，分别在持久性，服务和表示层中。因此，您可以使用@Component注释组件类，但是通过使用@Repository，@Service或@Controller注释它们，您的类更适合通过工具处理或与方面关联。例如，这些刻板印象注释成为切入点的理想目标。在未来的Spring Framework版本中，@Repository，@Service和@Controller也可能带有额外的语义。因此，如果您选择在服务层使用@Component或@Service，@Service显然是更好的选择。同样，如上所述，已经支持@Repository作为持久层中自动异常转换的标记。

### 7.10.2 元注释



### 7.10.3 自动检测类并注册bean定义

### 7.10.4 使用过滤器自定义扫描

### 7.10.5 在组件中定义bean元数据

### 7.10.6 命名自动检测的组件

### 7.10.7 为自动检测的组件提供范围

### 7.10.8 提供带注释的限定符元数据




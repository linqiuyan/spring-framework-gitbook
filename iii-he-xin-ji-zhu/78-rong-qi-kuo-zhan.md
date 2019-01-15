## 7.8  容器扩展点

通常，应用程序开发人员不需要子类化ApplicationContext实现类。 相反，可以通过插入特殊集成接口的实现来扩展Spring IoC容器。 接下来的几节将介绍这些集成接口。

### 7.8.1 使用BeanPostProcessor定制bean

BeanPostProcessor接口定义了可以实现的回调方法，以提供您自己的（或覆盖容器的默认）实例化逻辑，依赖关系解析逻辑等。 如果要在Spring容器完成实例化，配置和初始化bean之后实现某些自定义逻辑，则可以插入一个或多个BeanPostProcessor实现。

您可以配置多个BeanPostProcessor实例，并且可以通过设置order属性来控制这些BeanPostProcessors的执行顺序。 仅当BeanPostProcessor实现Ordered接口时，才能设置此属性; 如果您编写自己的BeanPostProcessor，则应考虑实现Ordered接口。 有关更多详细信息，请参阅BeanPostProcessor和Ordered接口的javadoc。 另请参阅下面有关BeanPostProcessors编程注册的说明。

#### 示例：Hello World，BeanPostProcessor样式

#### 示例：RequiredAnnotationBeanPostProcessor

### 7.8.2 使用BeanFactoryPostProcessor定制配置元数据

#### 示例：类名替换PropertyPlaceholderConfigurer

#### 示例：PropertyOverrideConfigurer

### 7.8.3 使用FactoryBean自定义实例化逻辑




## 7.7  Bean定义继承

## 7.8 集装箱扩建点

### 7.8.1 使用BeanPostProcessor定制bean

示例：Hello World，BeanPostProcessor样式

示例：RequiredAnnotationBeanPostProcessor

### 7.8.2 使用BeanFactoryPostProcessor定制配置元数据

示例：类名替换PropertyPlaceholderConfigurer

示例：PropertyOverrideConfigurer

### 7.8.3 使用FactoryBean自定义实例化逻辑

## 7.9 基于注释的容器配置

### 7.9.1  @Required

### 7.9.2  @Autowired

### 7.9.3 使用@Primary微调基于注释的自动装配

### 7.9.4 使用限定符微调基于注释的自动装配

### 7.9.5 使用泛型作为自动装配限定符

### 7.9.6  CustomAutowireConfigurer上

### 7.9.7  @Resource

### 7.9.8  @PostConstruct和@PreDestroy

## 7.10 类路径扫描和托管组件

### 7.10.1  @Component和进一步的构造型注释

### 7.10.2 元注释

### 7.10.3 自动检测类并注册bean定义

### 7.10.4 使用过滤器自定义扫描

### 7.10.5 在组件中定义bean元数据

### 7.10.6 命名自动检测的组件

### 7.10.7 为自动检测的组件提供范围

### 7.10.8 提供带注释的限定符元数据

## 7.11 使用JSR 330标准注释

### 7.11.1 使用@Inject和@Named进行依赖注入

### 7.11.2  @Named和@ManagedBean：@Component注释的标准等价物

### 7.11.3  JSR-330标准注释的局限性

## 7.12 基于Java的容器配置

### 7.12.1 基本概念：@Bean和@Configuration

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

7.12.5 编写基于Java的配置

使用@Import注释

有条件地包括@Configuration类或@Bean方法

结合Java和XML配置

7.13 环境抽象

7.13.1  Bean定义配置文件

@Profile

XML bean定义配置文件

激活profile

默认profile

7.13.2  PropertySource抽象

7.13.3  @PropertySource

7.13.4 占位符决议在陈述中

7.14 注册LoadTimeWeaver

7.15  ApplicationContext的其他功能

7.15.1 使用MessageSource进行国际化

7.15.2 标准和自定义活动

基于注释的事件侦听器

异步监听器

订购听众

通用事件

7.15.3 方便地访问低级资源

7.15.4 方便的Web应用程序的ApplicationContext实例化

7.15.5 将Spring ApplicationContext部署为Java EE RAR文件

7.16  BeanFactory

7.16.1  BeanFactory或ApplicationContext？

7.16.2 耦合代码与不良单例


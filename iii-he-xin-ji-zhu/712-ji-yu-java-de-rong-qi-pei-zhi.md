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


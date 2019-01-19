## 10.1 介绍

Spring Expression Language（简称SpEL）是一种强大的表达式语言，支持在运行时查询和操作对象图。语言语法类似于Unified EL，但提供了其他功能，最值得注意的是方法调用和基本字符串模板功能。

虽然还有其他几种Java表达式语言，例如OGNL，MVEL和JBoss EL，但是创建Spring表达语言是为了向Spring社区提供一种支持良好的表达式语言，可以在所有产品中使用。春季组合。其语言功能受Spring组合项目要求的驱动，包括基于eclipse的Spring Tool Suite中代码完成支持的工具要求。也就是说，SpEL基于技术无关的API，允许在需要时集成其他表达式语言实现。

虽然SpEL是Spring组合中表达式评估的基础，但它并不直接与Spring绑定，可以单独使用。为了自包含，本章中的许多示例都使用SpEL，就像它是一种独立的表达式语言一样。这需要创建一些引导基础结构类，例如解析器。大多数Spring用户不需要处理此基础结构，而只需编写表达式字符串以进行评估。这种典型用法的一个示例是将SpEL集成到创建XML或基于注释的bean定义中，如表达式支持定义bean定义一节所示。

本章介绍表达式语言的功能，API及其语言语法。在几个地方，Inventor和Inventor的Society类被用作表达式评估的目标对象。这些类声明和用于填充它们的数据列在本章末尾。

表达式语言支持以下功能：

Literal expressions 文字表达

* Boolean and relational operators 布尔和关系运算符
* Regular expressions 常用表达
* Class expressions 类表达式
* Accessing properties, arrays, lists, maps 访问属性，数组，列表，映射
* Method invocation 方法调用
* Relational operators 关系运算符
* Assignment 分配
* Calling constructors 调用构造函数
* Bean references Bean引用
* Array construction 阵列构造
* Inline lists 内联列表
* Inline maps 内联地图
* Ternary operator  三元运算符
* Variables 变量
* User defined functions 用户定义的功能
* Collection projection  
* Collection selection 
* Templated expressions 模板化的表达




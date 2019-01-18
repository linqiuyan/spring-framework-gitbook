## 9.4 Bean操作和BeanWrapper

org.springframework.beans包遵循Oracle提供的JavaBeans标准。 JavaBean只是一个具有默认无参数构造函数的类，它遵循命名约定，其中（作为示例）名为bingoMadness的属性将具有setter方法setBingoMadness（..）和getter方法getBingoMadness（）。 有关JavaBeans和规范的更多信息，请参阅Oracle的网站（javabeans）。

beans包中一个非常重要的类是BeanWrapper接口及其相应的实现（BeanWrapperImpl）。 从javadocs引用，BeanWrapper提供了设置和获取属性值（单独或批量），获取属性描述符以及查询属性以确定它们是可读还是可写的功能。 此外，BeanWrapper还支持嵌套属性，使子属性的属性设置为无限深度。 然后，BeanWrapper支持添加标准JavaBeans PropertyChangeListeners和VetoableChangeListeners的功能，而无需在目标类中支持代码。 最后但同样重要的是，BeanWrapper支持索引属性的设置。 BeanWrapper通常不直接由应用程序代码使用，而是由DataBinder和BeanFactory使用。

The way the

`BeanWrapper`

works is partly indicated by its name:

_it wraps a bean_

to perform actions on that bean, like setting and retrieving properties.

### 9.4.1 设置和获取基本和嵌套属性

### 9.4.2 内置PropertyEditor实现

#### 注册其他自定义PropertyEditors




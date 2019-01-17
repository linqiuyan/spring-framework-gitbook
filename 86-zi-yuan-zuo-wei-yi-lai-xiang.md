## 8.6  资源作为依赖项

如果bean本身将通过某种动态过程确定并提供资源路径，那么bean使用ResourceLoader接口加载资源可能是有意义的。 以某种模板的加载为例，其中所需的特定资源取决于用户的角色。 如果资源是静态的，那么完全消除ResourceLoader接口的使用是有意义的，只需让bean公开它所需的Resource属性，并期望将它们注入到它中。

然后注入这些属性变得微不足道的是，所有应用程序上下文都注册并使用特殊的JavaBeans PropertyEditor，它可以将String路径转换为Resource对象。 因此，如果myBean具有Resource类型的模板属性，则可以使用该资源的简单字符串进行配置，如下所示：

```java
<bean id="myBean" class="...">
    <property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
```

请注意，资源路径没有前缀，因为应用程序上下文本身将用作ResourceLoader，资源本身将通过ClassPathResource，FileSystemResource或ServletContextResource（视情况而定）加载，具体取决于上下文的确切类型。

如果需要强制使用特定的资源类型，则可以使用前缀。 以下两个示例显示如何强制ClassPathResource和UrlResource（后者用于访问文件系统文件）。

```java
<property name="template" value="classpath:some/resource/path/myTemplate.txt">
<property name="template" value="file:///some/resource/path/myTemplate.txt"/>
```




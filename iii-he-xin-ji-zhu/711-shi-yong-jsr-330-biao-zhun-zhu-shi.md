## 7.11 使用JSR 330标准注释

从Spring 3.0开始，Spring提供对JSR-330标准注释（依赖注入）的支持。 这些注释的扫描方式与Spring注释相同。 您只需要在类路径中包含相关的jar。

如果您使用的是Maven，则javax.inject工件可在标准Maven存储库中找到（[http://repo1.maven.org/maven2/javax/inject/javax.inject/1/）。](http://repo1.maven.org/maven2/javax/inject/javax.inject/1/）。) 您可以将以下依赖项添加到文件pom.xml：

```
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

### 7.11.1 使用@Inject和@Named进行依赖注入

而不是@Autowired，@ javax.inject.Inject可以如下使用：



### 7.11.2  @Named和@ManagedBean：@Component注释的标准等价物

### 7.11.3  JSR-330标准注释的局限性




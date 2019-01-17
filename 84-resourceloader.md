## 8.4  ResourceLoader

ResourceLoader接口旨在由可以返回（即加载）Resource实例的对象实现。

```java
public interface ResourceLoader {

    Resource getResource(String location);

}
```

所有application contexts都实现ResourceLoader接口，因此可以使用所有应用程序上下文来获取Resource实例。

当您在特定应用程序上下文上调用getResource（）并且指定的位置路径没有特定前缀时，您将获得适合该特定应用程序上下文的Resource类型。 例如，假设针对ClassPathXmlApplicationContext实例执行了以下代码片段：

```java
Resource template = ctx.getResource("some/resource/path/myTemplate.txt");
```

将返回的是ClassPathResource; 如果对FileSystemXmlApplicationContext实例执行相同的方法，则会返回FileSystemResource。 对于WebApplicationContext，您将获得ServletContextResource，依此类推。

因此，您可以以适合特定应用程序上下文的方式加载资源。

另一方面，您也可以通过指定特殊的类路径来强制使用ClassPathResource，而不管应用程序上下文类型如何：prefix：

```java
Resource template = ctx.getResource("classpath:some/resource/path/myTemplate.txt");
```

类似地，可以通过指定任何标准java.net.URL前缀来强制使用UrlResource：

```java
Resource template = ctx.getResource("file:///some/resource/path/myTemplate.txt");
```

```java
Resource template = ctx.getResource("http://myhost.com/resource/path/myTemplate.txt");
```

下表总结了将字符串转换为资源的策略：

| Prefix | Example | Explanation |
| :--- | :--- | :--- |
| classpath: | `classpath:com/myapp/config.xml` | 从类路径加载。 |
| file: | [`file:///data/config.xml`](file:///data/config.xml) | Loaded as a`URL`, from the filesystem.[\[1\]](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#ftn.d5e6190) |
| http: | [`http://myserver/logo.png`](http://myserver/logo.png) | Loaded as a`URL`. |
| \(none\) | `/data/config.xml` | 取决于底层的ApplicationContext。 |
|  |  | [\[1\]](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#d5e6190)But see also[Section 8.7.3, “FileSystemResource caveats”](https://docs.spring.io/spring/docs/4.3.21.RELEASE/spring-framework-reference/htmlsingle/#resources-filesystemresource-caveats).\[1\]但另见第8.7.3节“FileSystemResource告诫”。 |




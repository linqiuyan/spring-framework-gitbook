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

```
import javax.inject.Inject;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.findMovies(...);
        ...
    }
}
```

与@Autowired一样，可以在字段级别，方法级别和构造函数 - 参数级别使用@Inject。 此外，您可以将注入点声明为提供者，允许按需访问较短范围的bean或通过Provider.get（）调用对其他bean的延迟访问。 作为上述示例的变体：

```
import javax.inject.Inject;
import javax.inject.Provider;

public class SimpleMovieLister {

    private Provider<MovieFinder> movieFinder;

    @Inject
    public void setMovieFinder(Provider<MovieFinder> movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.get().findMovies(...);
        ...
    }
}
```

如果要为应该注入的依赖项使用限定名称，则应使用@Named批注，如下所示：

```
import javax.inject.Inject;
import javax.inject.Named;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

与@Autowired一样，@ Inject也可以与java.util.Optional或@Nullable一起使用。 这更适用于此，因为@Inject没有必需的属性。

```
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

```
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

### 7.11.2  @Named和@ManagedBean：@Component注释的等价物

不同于@Component，@ javax.inject.Named或javax.annotation.ManagedBean可以使用如下：

```
import javax.inject.Inject;
import javax.inject.Named;

@Named("movieListener")  // @ManagedBean("movieListener") could be used as well
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

在不指定组件名称的情况下使用@Component是很常见的。 @Named可以以类似的方式使用：

```
import javax.inject.Inject;
import javax.inject.Named;

@Named
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

使用@Named或@ManagedBean时，可以使用与使用Spring注释时完全相同的方式使用组件扫描：

```
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```

与@Component相比，JSR-330 @Named和JSR-250 ManagedBean注释不可组合。 请使用Spring的构造型模型来构建自定义组件注释。

### 7.11.3  JSR-330标准注释的局限性

使用标准注释时，重要的是要知道某些重要功能不可用，如下表所示：

Spring组件模型元素与JSR-330变体

| Spring |
| :--- |


|  | javax.inject.\* | javax.inject restrictions / comments |
| :--- | :--- | :--- |
| @Autowired | @Inject | @Inject没有'required'属性; 可以与Java 8的Optional一起使用。 |
| @Component | @Named / @ManagedBean | JSR-330不提供可组合模型，只是一种识别命名组件的方法。 |
| @Scope\("singleton"\) | @Singleton | JSR-330的默认范围就像Spring的原型。 但是，为了使其与Spring的一般默认值保持一致，默认情况下，Spring容器中声明的JSR-330 bean是一个单例。 为了使用除单例之外的范围，您应该使用Spring的@Scope注释。 javax.inject还提供了@Scope注释。 然而，这个仅用于创建自己的注释。 |
| @Qualifier | @Qualifier / @Named | javax.inject.Qualifieris只是一个用于构建自定义限定符的元注释。 具体字符串限定符（如Spring的@Qualifier具有值）可以通过javax.inject.Named关联。 |
| @Value | - | no equivalent |
| @Required | - | no equivalent |
| @Lazy | - | no equivalent |
| ObjectFactory | Provider | javax.inject.Provider是Spring的ObjectFactory的直接替代品，只是使用更短的get（）方法名称。 它也可以与Spring的@Autowired结合使用，也可以与非注释的构造函数和setter方法结合使用。 |




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

### 7.11.2  @Named和@ManagedBean：@Component注释的标准等价物

### 7.11.3  JSR-330标准注释的局限性




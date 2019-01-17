## 8.2  资源界面

Spring的Resource接口旨在成为一个更有能力的接口，用于抽象对low-level资源的访问。

```java
public interface Resource extends InputStreamSource {

    boolean exists();

    boolean isOpen();

    URL getURL() throws IOException;

    File getFile() throws IOException;

    Resource createRelative(String relativePath) throws IOException;

    String getFilename();

    String getDescription();

}
```

```java
public interface InputStreamSource {

    InputStream getInputStream() throws IOException;

}
```

Resource接口中一些最重要的方法是：

* getInputStream\(\)：找到并打开资源，返回一个InputStream以便从资源中读取。 预计每次调用都会返回一个新的InputStream。 呼叫者有责任关闭流。
* exists\(\)：返回一个布尔值，指示此资源是否实际存在于物理形式中。
* isOpen\(\)：返回一个布尔值，指示此资源是否表示具有开放流的句柄。 **如果为true，则无法多次读取InputStream**，并且必须仅读取一次然后关闭以避免资源泄漏。 除了InputStreamResource之外，所有常规资源实现都将为false。
* getDescription\(\)：返回此资源的描述，用于处理资源时的错误输出。 这通常是完全限定的文件名或资源的实际URL。

其他方法允许您获取表示资源的实际URL或File对象（如果底层实现兼容，并支持该功能）。

资源抽象在Spring本身广泛使用，在需要资源时作为许多方法签名中的参数类型。 某些Spring API中的其他方法（例如各种ApplicationContext实现的构造函数）采用一个非简单或简单形式的String来创建适合该上下文实现的资源，或者通过String路径上的特殊前缀，允许调用者 指定必须创建和使用特定的Resource实现。

尽管Spring接口和Spring使用了很多资源接口，但在您自己的代码中使用它作为通用实用程序类实际上非常有用，用于访问资源，即使您的代码不知道或不关心任何其他Spring的一部分。 虽然这会将您的代码耦合到Spring，但它实际上只将它耦合到这一小组实用程序类，这些实用程序类作为URL的更有能力的替代品，并且可以被认为等同于您将用于此目的的任何其他库。

重要的是要注意资源抽象不会取代功能：它尽可能地包装它。 例如，UrlResource包装一个URL，并使用包装的URL来完成其工作。




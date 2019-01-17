## 8.7 应用程序上下文和资源路径

### 8.7.1  构建应用程序上下文

应用程序上下文构造函数（对于特定的应用程序上下文类型）通常将字符串或字符串数组作为资源的位置路径（例如构成上下文定义的XML文件）。

当这样的位置路径没有前缀时，从该路径构建并用于加载bean定义的特定资源类型取决于并且适合于特定的应用程序上下文。 例如，如果您创建ClassPathXmlApplicationContext，如下所示：

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("conf/appContext.xml");
```

bean定义将从类路径加载，因为将使用ClassPathResource。 但是如果您按如下方式创建FileSystemXmlApplicationContext：

```
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("conf/appContext.xml");
```

bean定义将从文件系统位置加载，在这种情况下相对于当前工作目录。

请注意，在位置路径上使用特殊类路径前缀或标准URL前缀将覆盖为加载定义而创建的默认资源类型。 所以这个FileSystemXmlApplicationContext ......

```
ApplicationContext ctx =
    new FileSystemXmlApplicationContext("classpath:conf/appContext.xml");
```

实际上将从类路径加载其bean定义。 但是，它仍然是FileSystemXmlApplicationContext。 如果它随后用作ResourceLoader，则任何未加前缀的路径仍将被视为文件系统路径。

#### 构造ClassPathXmlApplicationContext实例 - 快捷方式

ClassPathXmlApplicationContext公开了许多构造函数，以实现方便的实例化。 基本思想是，只提供一个字符串数组，只包含XML文件本身的文件名（没有前导路径信息），还有一个提供类; ClassPathXmlApplicationContext将从提供的类派生路径信息。

一个例子有望清楚地表明这一点。 考虑一个如下所示的目录布局：

```
com/
  foo/
    services.xml
    daos.xml
    MessengerService.class
```

由'services.xml'和'daos.xml'中定义的bean组成的ClassPathXmlApplicationContext实例可以像这样实例化...

```
ApplicationContext ctx = new ClassPathXmlApplicationContext(
    new String[] {"services.xml", "daos.xml"}, MessengerService.class);
```

有关各种构造函数的详细信息，请参阅ClassPathXmlApplicationContext javadocs。

### 8.7.2 应用程序上下文构造函数资源路径中的通配符

#### Ant-style的图案

#### classpath \*：前缀

#### 与通配符有关的其他说明

### 8.7.3  FileSystemResource需要注意




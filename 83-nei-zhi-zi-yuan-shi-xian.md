## 8.3  内置资源实现

有许多资源实现在Spring中直接提供：

### 8.3.1 UrlResource对象

UrlResource包装java.net.URL，可用于访问通常可通过URL访问的任何对象，例如文件，HTTP目标，FTP目标等。所有URL都具有标准化的字符串表示形式，例如适当的标准化前缀用于表示另一个URL类型。这包括文件：用于访问文件系统路径，http：用于通过HTTP协议访问资源，ftp：用于通过FTP访问资源等。

UrlResource是由Java代码使用UrlResource构造函数显式创建的，但是当您调用API方法时，通常会隐式创建它，该方法接受一个表示路径的String参数。对于后一种情况，JavaBeans PropertyEditor最终将决定要创建哪种类型的Resource。如果路径字符串包含一些众所周知的（对于它）前缀，例如classpath：，它将为该前缀创建适当的专用资源。但是，如果它无法识别前缀，则会假定这只是一个标准的URL字符串，并将创建一个UrlResource。

### 8.3.2 使用ClassPathResource

此类表示应从类路径获取的资源。 这使用线程上下文类加载器，给定的类加载器或给定的类来加载资源。

如果类路径资源驻留在文件系统中，则此Resource实现支持解析为java.io.File，但不支持驻留在jar中且尚未扩展（通过servlet引擎或任何环境）的类路径资源的解析 文件系统。 为了解决这个问题，各种Resource实现始终支持作为java.net.URL的解析。

ClassPathResource是由Java代码使用ClassPathResource构造函数显式创建的，但是当您调用一个API方法时，它通常会隐式创建，该方法接受一个表示路径的String参数。 对于后一种情况，JavaBeans PropertyEditor将识别字符串路径上的特殊前缀classpath：并在该情况下创建ClassPathResource。

### 8.3.3 FileSystemResource

### 8.3.4 ServletContextResource

### 8.3.5 InputStreamResource

### 8.3.6 使用ByteArrayResource




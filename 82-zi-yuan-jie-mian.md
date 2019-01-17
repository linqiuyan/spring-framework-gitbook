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








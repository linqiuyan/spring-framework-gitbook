## 9.5 Spring类型转换

Spring 3引入了一个core.convert包，它提供了一个通用的类型转换系统。 系统定义了一个SPI来实现类型转换逻辑，以及一个在运行时执行类型转换的API。 在Spring容器中，此系统可用作PropertyEditors的替代方法，以将外部化的bean属性值字符串转换为必需的属性类型。 公共API也可以在您的应用程序中需要进行类型转换的任何地方使用。

### 9.5.1 转换器SPI（service provider interface）

用于实现类型转换逻辑的SPI简单且强类型化：

```java
package org.springframework.core.convert.converter;

public interface Converter<S, T> {

    T convert(S source);
}
```



### 9.5.2 ConverterFactory

### 9.5.3 GenericConverter

### ConditionalGenericConverter

### 9.5.4 ConversionService API

### 9.5.5 配置ConversionService

### 9.5.6 以编程方式使用ConversionService




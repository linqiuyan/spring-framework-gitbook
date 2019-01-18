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

要创建自己的转换器，只需实现上面的接口即可。 将S作为要转换的类型参数化，将T作为要转换的类型。 如果S的集合或数组需要转换为T的数组或集合，也可以透明地应用这样的转换器，前提是已经注册了委托数组/集合转换器（DefaultConversionService默认情况下也是如此）。

对于每次转换（S）的调用，源参数保证为非null。 如果转换失败，您的Converter可能会抛出任何未经检查的异常; 具体而言，应抛出IllegalArgumentException以报告无效的源值。 请注意确保您的Converter实现是线程安全的。

为方便起见，core.convert.support包中提供了几个转换器实现。 这些包括从字符串到数字的转换器和其他常见类型。 将StringToInteger视为典型Converter实现的示例：

```java
package org.springframework.core.convert.support;

final class StringToInteger implements Converter<String, Integer> {

    public Integer convert(String source) {
        return Integer.valueOf(source);
    }
}
```

### 9.5.2 ConverterFactory

当您需要集中整个类层次结构的转换逻辑时，例如，从String转换为java.lang.Enum对象时，实现ConverterFactory：

```java
package org.springframework.core.convert.converter;

public interface ConverterFactory<S, R> {

    <T extends R> Converter<S, T> getConverter(Class<T> targetType);
}
```



### 9.5.3 GenericConverter

### ConditionalGenericConverter

### 9.5.4 ConversionService API

### 9.5.5 配置ConversionService

### 9.5.6 以编程方式使用ConversionService




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

参数化S为要转换的类型，R为定义可转换为的类范围的基本类型。 然后实现getConverter（Class &lt;T&gt;），其中T是R的子类。

以StringToEnum ConverterFactory为例：

```java
package org.springframework.core.convert.support;

final class StringToEnumConverterFactory implements ConverterFactory<String, Enum> {

    public <T extends Enum> Converter<String, T> getConverter(Class<T> targetType) {
        return new StringToEnumConverter(targetType);
    }

    private final class StringToEnumConverter<T extends Enum> implements Converter<String, T> {

        private Class<T> enumType;

        public StringToEnumConverter(Class<T> enumType) {
            this.enumType = enumType;
        }

        public T convert(String source) {
            return (T) Enum.valueOf(this.enumType, source.trim());
        }
    }
}
```

### 9.5.3 GenericConverter

当您需要复杂的Converter实现时，请考虑GenericConverter接口。 通过更灵活但不太强类型的签名，GenericConverter支持在多个源类型和目标类型之间进行转换。 此外，GenericConverter提供了在实现转换逻辑时可以使用的源和目标字段上下文。 这样的上下文允许类型转换由字段注释或在字段签名上声明的通用信息驱动。

```java
package org.springframework.core.convert.converter;

public interface GenericConverter {

    public Set<ConvertiblePair> getConvertibleTypes();

    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
}
```

要实现GenericConverter，请让getConvertibleTypes\(\)返回支持的源→目标类型对。 然后实现convert（Object，TypeDescriptor，TypeDescriptor）来实现转换逻辑。 源TypeDescriptor提供对包含要转换的值的源字段的访问。 目标TypeDescriptor提供对将设置转换值的目标字段的访问。

GenericConverter的一个很好的例子是在Java Array和Collection之间进行转换的转换器。 这样的ArrayToCollectionConverter会对声明目标Collection类型的字段进行内省，以解析Collection的元素类型。 这允许在目标字段上设置Collection之前，将源数组中的每个元素转换为Collection元素类型。

_因为GenericConverter是一个更复杂的SPI接口，所以只在需要时使用它。 喜欢Converter或ConverterFactory以满足基本的类型转换需求。_

#### ConditionalGenericConverter

有时，如果特定条件成立，您只需要执行转换器。 例如，如果目标字段上存在特定注释，则可能只想执行Converter。 或者，如果在目标类上定义了特定方法（例如静态valueOf方法），则可能只想执行Converter。 ConditionalGenericConverter是GenericConverter和ConditionalConverter接口的并集，允许您定义此类自定义匹配条件：

```
public interface ConditionalConverter {

    boolean matches(TypeDescriptor sourceType, TypeDescriptor targetType);
}

public interface ConditionalGenericConverter extends GenericConverter, ConditionalConverter {
}
```

ConditionalGenericConverter的一个很好的例子是EntityConverter，它在持久实体标识符和实体引用之间进行转换。 如果目标实体类型声明静态查找器方法，则这样的EntityConverter可能仅匹配，例如，findAccount（Long）。 您将在匹配的实现（TypeDescriptor，TypeDescriptor）中执行这样的finder方法检查。

### 9.5.4 ConversionService API

ConversionService定义了一个统一的API，用于在运行时执行类型转换逻辑。 转换器通常在此Facade接口后面执行：

```java
package org.springframework.core.convert;

public interface ConversionService {

    boolean canConvert(Class<?> sourceType, Class<?> targetType);

    <T> T convert(Object source, Class<T> targetType);

    boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);

    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);

}
```

大多数ConversionService实现还实现了ConverterRegistry，它提供了一个用于注册转换器的SPI。 在内部，ConversionService实现委托其注册的转换器执行类型转换逻辑。

core.convert.support包中提供了一个强大的ConversionService实现。 GenericConversionService是适用于大多数环境的通用实现。 ConversionServiceFactory为创建常见的ConversionService配置提供了方便的工厂。

### 9.5.5 配置ConversionService

ConversionService是一个无状态对象，旨在在应用程序启动时实例化，然后在多个线程之间共享。 在Spring应用程序中，通常为每个Spring容器（或ApplicationContext）配置一个ConversionService实例。 那个ConversionService将被Spring选中，然后在框架需要执行类型转换时使用。 您也可以将此ConversionService注入任何bean并直接调用它。

如果没有向Spring注册ConversionService，则使用基于PropertyEditor的原始系统。

要使用Spring注册默认的ConversionService，请使用id conversionService添加以下bean定义：

```
<bean id="conversionService"
    class="org.springframework.context.support.ConversionServiceFactoryBean"/>
```



### 9.5.6 以编程方式使用ConversionService




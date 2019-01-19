## 9.6 Spring Field格式

如上一节所述，core.convert是一种通用类型转换系统。它提供了统一的ConversionService API以及强类型转换器SPI，用于实现从一种类型到另一种类型的转换逻辑。 Spring Container使用此系统绑定bean属性值。此外，Spring Expression Language（SpEL）和DataBinder都使用此系统绑定字段值。例如，当SpEL需要强制Short to a Long来完成expression.setValue（Object bean，Object value）尝试时，core.convert系统会执行强制。

现在考虑典型客户端环境（如Web或桌面应用程序）的类型转换要求。在这样的环境中，您通常从String转换为支持客户端回发过程，以及返回String以支持视图呈现过程。此外，您经常需要本地化String值。更通用的core.convert Converter SPI不直接解决此类格式化要求。为了直接解决这些问题，Spring 3引入了一个方便的Formatter SPI，为客户端环境提供了一个简单而强大的PropertyEditors替代方案。

通常，在需要实现通用类型转换逻辑时使用Converter SPI;例如，用于在java.util.Date和java.lang.Long之间进行转换。当您在客户端环境（例如Web应用程序）中工作时，请使用Formatter SPI，并且需要解析和打印本地化的字段值。 ConversionService为两个SPI提供统一的类型转换API。

### 9.6.1 格式化SPI\(Formatter SPI\)

用于实现字段格式化逻辑的Formatter SPI简单且强类型化：

```
package org.springframework.format;

public interface Formatter<T> extends Printer<T>, Parser<T> {
}
```

Formatter从Printer和Parser构建块接口扩展的位置：

```
public interface Printer<T> {

    String print(T fieldValue, Locale locale);
}
```

```
import java.text.ParseException;

public interface Parser<T> {

    T parse(String clientValue, Locale locale) throws ParseException;
}
```

要创建自己的Formatter，只需实现上面的Formatter接口即可。将T将参数化为要格式化的对象类型，例如java.util.Date。实现print（）操作以打印T的实例以在客户端语言环境中显示。实现parse（）操作以从客户端语言环境返回的格式化表示中解析T的实例。如果解析尝试失败，您的Formatter应抛出ParseException或IllegalArgumentException。请注意确保您的Formatter实现是线程安全的。

为方便起见，在格式子包中提供了几种Formatter实现。 number包提供NumberStyleFormatter，CurrencyStyleFormatter和PercentStyleFormatter，以使用java.text.NumberFormat格式化java.lang.Number对象。 datetime包提供了一个DateFormatter，用java.text.DateFormat格式化java.util.Date对象。 datetime.joda包基于Joda-Time库提供全面的日期时间格式支持。

将DateFormatter视为Formatter实现的示例：

```java
package org.springframework.format.datetime;

public final class DateFormatter implements Formatter<Date> {

    private String pattern;

    public DateFormatter(String pattern) {
        this.pattern = pattern;
    }

    public String print(Date date, Locale locale) {
        if (date == null) {
            return "";
        }
        return getDateFormat(locale).format(date);
    }

    public Date parse(String formatted, Locale locale) throws ParseException {
        if (formatted.length() == 0) {
            return null;
        }
        return getDateFormat(locale).parse(formatted);
    }

    protected DateFormat getDateFormat(Locale locale) {
        DateFormat dateFormat = new SimpleDateFormat(this.pattern, locale);
        dateFormat.setLenient(false);
        return dateFormat;
    }
}
```

Spring团队欢迎社区驱动的Formatter贡献; 请参阅jira.spring.io进行投稿。

### 9.6.2 注释驱动的格式

如您所见，字段格式可以通过字段类型或注释进行配置。 要将Annotation绑定到格式化程序，请实现AnnotationFormatterFactory：

```
package org.springframework.format;

public interface AnnotationFormatterFactory<A extends Annotation> {

    Set<Class<?>> getFieldTypes();

    Printer<?> getPrinter(A annotation, Class<?> fieldType);

    Parser<?> getParser(A annotation, Class<?> fieldType);
}
```

### 格式注释API

如您所见，字段格式可以通过字段类型或注释进行配置。 要将Annotation绑定到格式化程序，请实现AnnotationFormatterFactory：

```java
package org.springframework.format;

public interface AnnotationFormatterFactory<A extends Annotation> {

    Set<Class<?>> getFieldTypes();

    Printer<?> getPrinter(A annotation, Class<?> fieldType);

    Parser<?> getParser(A annotation, Class<?> fieldType);
}
```

参数化A是要与格式化逻辑关联的字段annotationType，例如org.springframework.format.annotation.DateTimeFormat。 getFieldTypes\(\)返回可以使用注释的字段类型。 让getPrinter\(\)返回一个打印机来打印带注释字段的值。 让getParser\(\)返回一个Parser来解析带注释字段的clientValue。

下面的示例AnnotationFormatterFactory实现将@NumberFormat Annotation绑定到格式化程序。 此注释允许指定数字样式或模式：

```
public final class NumberFormatAnnotationFormatterFactory
        implements AnnotationFormatterFactory<NumberFormat> {

    public Set<Class<?>> getFieldTypes() {
        return new HashSet<Class<?>>(asList(new Class<?>[] {
            Short.class, Integer.class, Long.class, Float.class,
            Double.class, BigDecimal.class, BigInteger.class }));
    }

    public Printer<Number> getPrinter(NumberFormat annotation, Class<?> fieldType) {
        return configureFormatterFrom(annotation, fieldType);
    }

    public Parser<Number> getParser(NumberFormat annotation, Class<?> fieldType) {
        return configureFormatterFrom(annotation, fieldType);
    }

    private Formatter<Number> configureFormatterFrom(NumberFormat annotation, Class<?> fieldType) {
        if (!annotation.pattern().isEmpty()) {
            return new NumberStyleFormatter(annotation.pattern());
        } else {
            Style style = annotation.style();
            if (style == Style.PERCENT) {
                return new PercentStyleFormatter();
            } else if (style == Style.CURRENCY) {
                return new CurrencyStyleFormatter();
            } else {
                return new NumberStyleFormatter();
            }
        }
    }
}
```

要触发格式化，只需使用@NumberFormat注释字段：

```java
public class MyModel {

    @NumberFormat(style=Style.CURRENCY)
    private BigDecimal decimal;
}
```

### 格式注释API

org.springframework.format.annotation包中存在可移植格式注释API。 使用@NumberFormat格式化java.lang.Number字段。 使用@DateTimeFormat格式化java.util.Date，java.util.Calendar，java.util.Long或Joda-Time字段。

下面的示例使用@DateTimeFormat将java.util.Date格式化为ISO Date（yyyy-MM-dd）：

```
public class MyModel {

    @DateTimeFormat(iso=ISO.DATE)
    private Date date;
}
```

### 9.6.3 FormatterRegistry SPI

FormatterRegistry是一个用于注册格式化程序和转换器的SPI。 FormattingConversionService是FormatterRegistry的一个实现，适用于大多数环境。 可以使用FormattingConversionServiceFactoryBean以编程方式或声明方式将此实现配置为Spring bean。 因为此实现还实现了ConversionService，所以可以直接配置它以与Spring的DataBinder和Spring Expression Language（SpEL）一起使用。

查看下面的FormatterRegistry SPI：

```
package org.springframework.format;

public interface FormatterRegistry extends ConverterRegistry {

    void addFormatterForFieldType(Class<?> fieldType, Printer<?> printer, Parser<?> parser);

    void addFormatterForFieldType(Class<?> fieldType, Formatter<?> formatter);

    void addFormatterForFieldType(Formatter<?> formatter);

    void addFormatterForAnnotation(AnnotationFormatterFactory<?, ?> factory);
}
```

如上所示，可以通过fieldType或注释注册Formatters。

FormatterRegistry SPI允许您集中配置格式规则，而不是在控制器之间复制此类配置。 例如，您可能希望强制所有Date字段以某种方式格式化，或者具有特定注释的字段以某种方式格式化。 使用共享的FormatterRegistry，您可以定义一次这些规则，并在需要格式化时应用它们。

### 9.6.4 FormatterRegistrar SPI

FormatterRegistrar是一个SPI，用于通过FormatterRegistry注册格式化程序和转换器：

```
package org.springframework.format;

public interface FormatterRegistrar {

    void registerFormatters(FormatterRegistry registry);
}
```

在为给定格式类别注册多个相关转换器和格式化程序时，FormatterRegistrar非常有用，例如日期格式。 在声明性注册不充分的情况下，它也很有用。 例如，格式化程序需要在与其自己的&lt;T&gt;不同的特定字段类型下或在注册打印机/分析程序对时编制索引。 下一节提供有关转换器和格式化程序注册的更多信息。

### 9.6.5 在Spring MVC中配置格式化

请参见Spring MVC章节中的第22.16.3节“转换和格式化”。




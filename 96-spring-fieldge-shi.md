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



### 9.6.2 注释驱动的格式

### 格式注释API

### 9.6.3 FormatterRegistry SPI

### 9.6.4 FormatterRegistrar SPI

### 9.6.5 在Spring MVC中配置格式化




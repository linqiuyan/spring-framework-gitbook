## 9.6 Spring Field格式

如上一节所述，core.convert是一种通用类型转换系统。它提供了统一的ConversionService API以及强类型转换器SPI，用于实现从一种类型到另一种类型的转换逻辑。 Spring Container使用此系统绑定bean属性值。此外，Spring Expression Language（SpEL）和DataBinder都使用此系统绑定字段值。例如，当SpEL需要强制Short to a Long来完成expression.setValue（Object bean，Object value）尝试时，core.convert系统会执行强制。

现在考虑典型客户端环境（如Web或桌面应用程序）的类型转换要求。在这样的环境中，您通常从String转换为支持客户端回发过程，以及返回String以支持视图呈现过程。此外，您经常需要本地化String值。更通用的core.convert Converter SPI不直接解决此类格式化要求。为了直接解决这些问题，Spring 3引入了一个方便的Formatter SPI，为客户端环境提供了一个简单而强大的PropertyEditors替代方案。

通常，在需要实现通用类型转换逻辑时使用Converter SPI;例如，用于在java.util.Date和java.lang.Long之间进行转换。当您在客户端环境（例如Web应用程序）中工作时，请使用Formatter SPI，并且需要解析和打印本地化的字段值。 ConversionService为两个SPI提供统一的类型转换API。

### 9.6.1 格式化SPI

### 9.6.2 注释驱动的格式

### 格式注释API

### 9.6.3 FormatterRegistry SPI

### 9.6.4 FormatterRegistrar SPI

### 9.6.5 在Spring MVC中配置格式化




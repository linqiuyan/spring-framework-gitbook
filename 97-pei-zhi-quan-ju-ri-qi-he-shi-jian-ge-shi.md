## 9.7 配置全局日期和时间格式

默认情况下，未使用@DateTimeFormat注释的日期和时间字段使用DateFormat.SHORT样式从字符串转换。 如果您愿意，可以通过定义自己的全局格式来更改此设置。

您需要确保Spring不会注册默认格式化程序，而是应该手动注册所有格式化程序。 根据您是否使用Joda-Time库，使用org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar或org.springframework.format.datetime.DateFormatterRegistrar类。

例如，以下Java配置将注册全局“yyyyMMdd”格式。 此示例不依赖于Joda-Time库：

```java
@Configuration
public class AppConfig {

    @Bean
    public FormattingConversionService conversionService() {

        // Use the DefaultFormattingConversionService but do not register defaults
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService(false);

        // Ensure @NumberFormat is still supported
        conversionService.addFormatterForFieldAnnotation(new NumberFormatAnnotationFormatterFactory());

        // Register date conversion with a specific global format
        DateFormatterRegistrar registrar = new DateFormatterRegistrar();
        registrar.setFormatter(new DateFormatter("yyyyMMdd"));
        registrar.registerFormatters(conversionService);

        return conversionService;
    }
}
```

如果您更喜欢基于XML的配置，则可以使用FormattingConversionServiceFactoryBean。 这是相同的例子，这次使用Joda Time：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd>

    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="registerDefaultFormatters" value="false" />
        <property name="formatters">
            <set>
                <bean class="org.springframework.format.number.NumberFormatAnnotationFormatterFactory" />
            </set>
        </property>
        <property name="formatterRegistrars">
            <set>
                <bean class="org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar">
                    <property name="dateFormatter">
                        <bean class="org.springframework.format.datetime.joda.DateTimeFormatterFactoryBean">
                            <property name="pattern" value="yyyyMMdd"/>
                        </bean>
                    </property>
                </bean>
            </set>
        </property>
    </bean>
</beans>
```

Joda-Time提供单独的不同类型来表示日期，时间和日期时间值。 应使用JodaTimeFormatterRegistrar的dateFormatter，timeFormatter和dateTimeFormatter属性为每种类型配置不同的格式。 DateTimeFormatterFactoryBean提供了一种创建格式化程序的便捷方法。

如果您正在使用Spring MVC，请记住明确配置所使用的转换服务。 对于基于Java的@Configuration，这意味着扩展WebMvcConfigurationSupport类并覆盖mvcConversionService（）方法。 对于XML，您应该使用mvc：annotation-driven元素的'conversion-service'属性。 有关详细信息，请参见第22.16.3节“转换和格式化”。








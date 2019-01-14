## 7.14 注册LoadTimeWeaver

Spring使用LoadTimeWeaver在类加载到Java虚拟机（JVM）时动态转换类。

要启用加载时编织，请将@EnableLoadTimeWeaving添加到您的@Configuration类之一：

```
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```

或者，对于XML配置，请使用context：load-time-weaver元素：

```
<beans>
    <context:load-time-weaver/>
</beans>
```

一旦为ApplicationContext配置。 ApplicationContext中的任何bean都可以实现LoadTimeWeaverAware，从而接收对load-time weaver实例的引用。 这与Spring的JPA支持结合使用特别有用，其中JPA类转换可能需要加载时编织。 有关更多详细信息，请参阅LocalContainerEntityManagerFactoryBean javadocs。 有关AspectJ加载时编织的更多信息，请参见第11.8.4节“在Spring Framework中使用AspectJ进行加载时编织”。




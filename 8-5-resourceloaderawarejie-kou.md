## 8.5 ResourceLoaderAware接口

ResourceLoaderAware接口是一个特殊的标记接口，用于标识希望随ResourceLoader引用提供的对象。

```
public interface ResourceLoaderAware {

    void setResourceLoader(ResourceLoader resourceLoader);
}
```

当类实现ResourceLoaderAware并部署到应用程序上下文（作为Spring管理的bean）时，它被应用程序上下文识别为ResourceLoaderAware。 然后，应用程序上下文将调用setResourceLoader（ResourceLoader），将其自身作为参数提供（请记住，Spring中的所有applicationContext都实现了ResourceLoader接口）。

当然，由于ApplicationContext是一个ResourceLoader，bean也可以实现ApplicationContextAware接口并直接使用提供的应用程序上下文来加载资源，但一般情况下，最好使用专用的ResourceLoader接口，如果只需要它。 代码只是耦合到资源加载接口，可以将其视为实用程序接口，而不是整个Spring ApplicationContext接口。

从Spring 2.5开始，您可以依赖ResourceLoader的自动装配作为实现ResourceLoaderAware接口的替代方法。 “传统”构造函数和byType自动装配模式（如第7.4.5节“自动装配协作者”中所述）现在能够分别为构造函数参数或setter方法参数提供ResourceLoader类型的依赖项。 为了获得更大的灵活性（包括自动装配字段和多参数方法的能力），请考虑使用新的基于注释的自动装配功能。 在这种情况下，只要有问题的字段，构造函数或方法带有@Autowired注释，ResourceLoader就会自动装入一个期望ResourceLoader类型的字段，构造函数参数或方法参数。 有关更多信息，请参见第7.9.2节“@Autowired”。




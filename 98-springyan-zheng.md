## 9.8 Spring验证

Spring 3为其验证支持引入了一些增强功能。 首先，现在完全支持JSR-303 Bean Validation API。 其次，当以编程方式使用时，Spring的DataBinder现在可以验证对象以及绑定它们。 第三，Spring MVC现在支持声明性地验证@Controller输入。

### 9.8.1 JSR-303 Bean Validation API概述

JSR-303标准化了Java平台的验证约束声明和元数据。 使用此API，您可以使用声明性验证约束来注释域模型属性，并且运行时会强制执行它们。 您可以利用许多内置约束。 您还可以定义自己的自定义约束。

为了说明，请考虑一个具有两个属性的简单PersonForm模型：

```
public class PersonForm {
    private String name;
    private int age;
}
```

JSR-303允许您为这些属性定义声明性验证约束：

```
public class PersonForm {

    @NotNull
    @Size(max=64)
    private String name;

    @Min(0)
    private int age;
}
```

当JSR-303 Validator验证此类的实例时，将强制执行这些约束。

有关JSR-303 / JSR-349的一般信息，请参阅Bean Validation网站。 有关默认参考实现的特定功能的信息，请参阅Hibernate Validator文档。 要了解如何将Bean Validation提供程序设置为Spring bean，请继续阅读。

### 9.8.2 配置Bean验证提供程序

Spring提供对Bean Validation API的完全支持。 这包括方便地支持将JSR-303 / JSR-349 Bean Validation提供程序作为Spring bean引导。 这允许在应用程序中需要验证的任何地方注入javax.validation.ValidatorFactory或javax.validation.Validator。

使用LocalValidatorFactoryBean将默认Validator配置为Spring bean：

```
<bean id="validator"
    class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>
```

上面的基本配置将触发Bean Validation使用其默认引导机制进行初始化。 JSR-303 / JSR-349提供程序（如Hibernate Validator）预计会出现在类路径中并自动检测。

#### 注入验证器

LocalValidatorFactoryBean实现了javax.validation.ValidatorFactory和javax.validation.Validator，以及Spring的org.springframework.validation.Validator。 您可以将这些接口中的任何一个引用注入到需要调用验证逻辑的bean中。

如果您希望直接使用Bean Validation API，请注入对javax.validation.Validator的引用：

```
import javax.validation.Validator;

@Service
public class MyService {

    @Autowired
    private Validator validator;
```

如果您的bean需要Spring Validation API，则引用对org.springframework.validation.Validator的引用：

```
import org.springframework.validation.Validator;

@Service
public class MyService {

    @Autowired
    private Validator validator;
}
```



#### 配置自定义约束

每个Bean Validation约束由两部分组成。 首先，一个声明约束及其可配置属性的@Constraint注释。 第二，实现约束行为的javax.validation.ConstraintValidator接口的实现。 要将声明与实现相关联，每个@Constraint注释都引用相应的ConstraintValidator实现类。 在运行时，ConstraintValidatorFactory在域模型中遇到约束注释时实例化引用的实现。

默认情况下，LocalValidatorFactoryBean配置一个SpringConstraintValidatorFactory，它使用Spring创建ConstraintValidator实例。 这允许您的自定义ConstraintValidators像其他任何Spring bean一样受益于依赖注入。

下面显示的是自定义@Constraint声明的示例，后跟关联的ConstraintValidator实现，该实现使用Spring进行依赖项注入：

```
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy=MyConstraintValidator.class)
public @interface MyConstraint {
}
```

```
import javax.validation.ConstraintValidator;

public class MyConstraintValidator implements ConstraintValidator {

    @Autowired;
    private Foo aDependency;

    ...
}
```

正如您所看到的，ConstraintValidator实现可能与其他任何Spring bean一样具有@Autowired的依赖关系。



#### spring驱动的方法验证（Spring-driven Method Validation）

Bean Validation 1.1支持的方法验证功能，以及Hibernate Validator 4.3的自定义扩展，可以通过MethodValidationPostProcessor bean定义集成到Spring上下文中：

```
<bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor"/>
```

为了有资格进行Spring驱动的方法验证，所有目标类都需要使用Spring的@Validated注释进行注释，可以选择声明要使用的验证组。 使用Hibernate Validator和Bean Validation 1.1提供程序查看MethodValidationPostProcessor javadocs以获取设置详细信息。

#### 其他配置选项

对于大多数情况，默认的LocalValidatorFactoryBean配置应该足够了。 从消息插值到遍历解析，各种Bean Validation构造有许多配置选项。 有关这些选项的更多信息，请参见LocalValidatorFactoryBean javadocs。

### 9.8.3 配置DataBinder

从Spring 3开始，可以使用Validator配置DataBinder实例。 配置完成后，可以通过调用binder.validate（）来调用Validator。 任何验证错误都会自动添加到活页夹的BindingResult中。

以编程方式使用DataBinder时，可以在绑定到目标对象后使用它来调用验证逻辑：

```
Foo target = new Foo();
DataBinder binder = new DataBinder(target);
binder.setValidator(new FooValidator());

// bind to the target object
binder.bind(propertyValues);

// validate the target object
binder.validate();

// get BindingResult that includes any validation errors
BindingResult results = binder.getBindingResult();
```

还可以通过dataBinder.addValidators和dataBinder.replaceValidators为DataBinder配置多个Validator实例。 将全局配置的Bean验证与在DataBinder实例上本地配置的Spring验证器组合时，这非常有用。 见???。

### 9.8.4 Spring MVC 3验证

请参见Spring MVC章节中的第22.16.4节“验证”。


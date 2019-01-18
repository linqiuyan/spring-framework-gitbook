## 9.2 使用Spring的Validator接口进行验证

Spring具有Validator接口，可用于验证对象。 Validator接口使用Errors对象工作，以便在验证时，验证器可以向Errors对象报告验证失败。

让我们考虑一个小数据对象：

```
public class Person {

    private String name;
    private int age;

    // the usual getters and setters...
}
```

我们将通过实现org.springframework.validation.Validator接口的以下两个方法来为Person类提供验证行为：

* 支持（类） - 此Validator可以验证提供的类的实例吗？
* validate（Object，org.springframework.validation.Errors） - 验证给定对象，如果出现验证错误，请注册具有给定Errors对象的对象

实现Validator非常简单，特别是当您知道Spring Framework提供的ValidationUtils帮助器类时。

```java
public class PersonValidator implements Validator {

    /**
     * This Validator validates *just* Person instances
     */
    public boolean supports(Class clazz) {
        return Person.class.equals(clazz);
    }

    public void validate(Object obj, Errors e) {
        ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
        Person p = (Person) obj;
        if (p.getAge() < 0) {
            e.rejectValue("age", "negativevalue");
        } else if (p.getAge() > 110) {
            e.rejectValue("age", "too.darn.old");
        }
    }
}
```

如您所见，ValidationUtils类上的静态rejectIfEmpty（..）方法用于拒绝'name'属性（如果它为null或空字符串）。 看看ValidationUtils javadocs，看看它提供的功能除了前面的例子。

虽然可以实现一个Validator类来验证富对象中的每个嵌套对象，但最好将每个嵌套对象类的验证逻辑封装在自己的Validator实现中。 “富”对象的一个简单示例是Customer，它由两个String属性（第一个和第二个名称）和一个复杂的Address对象组成。 地址对象可以独立于Customer对象使用，因此实现了不同的AddressValidator。 如果您希望CustomerValidator重用AddressValidator类中包含的逻辑而不诉诸复制和粘贴，您可以在CustomerValidator中依赖注入或实例化AddressValidator，并像这样使用它：

```java
public class CustomerValidator implements Validator {

    private final Validator addressValidator;

    public CustomerValidator(Validator addressValidator) {
        if (addressValidator == null) {
            throw new IllegalArgumentException("The supplied [Validator] is " +
                "required and must not be null.");
        }
        if (!addressValidator.supports(Address.class)) {
            throw new IllegalArgumentException("The supplied [Validator] must " +
                "support the validation of [Address] instances.");
        }
        this.addressValidator = addressValidator;
    }

    /**
     * This Validator validates Customer instances, and any subclasses of Customer too
     */
    public boolean supports(Class clazz) {
        return Customer.class.isAssignableFrom(clazz);
    }

    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "field.required");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "surname", "field.required");
        Customer customer = (Customer) target;
        try {
            errors.pushNestedPath("address");
            ValidationUtils.invokeValidator(this.addressValidator, customer.getAddress(), errors);
        } finally {
            errors.popNestedPath();
        }
    }
}
```






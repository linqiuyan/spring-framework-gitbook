## 9.4 Bean操作和BeanWrapper

org.springframework.beans包遵循Oracle提供的JavaBeans标准。 JavaBean只是一个具有默认无参数构造函数的类，它遵循命名约定，其中（作为示例）名为bingoMadness的属性将具有setter方法setBingoMadness（..）和getter方法getBingoMadness（）。 有关JavaBeans和规范的更多信息，请参阅Oracle的网站（javabeans）。

beans包中一个非常重要的类是BeanWrapper接口及其相应的实现（BeanWrapperImpl）。 从javadocs引用，BeanWrapper提供了设置和获取属性值（单独或批量），获取属性描述符以及查询属性以确定它们是可读还是可写的功能。 此外，BeanWrapper还支持嵌套属性，使子属性的属性设置为无限深度。 然后，BeanWrapper支持添加标准JavaBeans PropertyChangeListeners和VetoableChangeListeners的功能，而无需在目标类中支持代码。 最后但同样重要的是，BeanWrapper支持索引属性的设置。 BeanWrapper通常不直接由应用程序代码使用，而是由DataBinder和BeanFactory使用。

BeanWrapper的工作方式部分由其名称表示：它包装bean以对该bean执行操作，如设置和检索属性。

### 9.4.1 设置和获取基本和嵌套属性

设置和获取属性是使用setPropertyValue（s）和getPropertyValue（s）方法完成的，这两种方法都带有几个重载变体。 它们都在Spring附带的javadocs中有更详细的描述。 重要的是要知道有一些用于指示对象属性的约定。 几个例子：

| Expression | 说明 |
| :--- | :--- |
| `name` | 指示与方法getName（）或isName（）和setName（..）对应的属性名称 |
| `account.name` | 表示对应的属性帐户的嵌套属性名称，例如 方法getAccount\(\)。setName\(\)或getAccount\(\)。getName\(\) |
| `account[2]` | 指示索引属性帐户的第三个元素。 索引属性可以是数组，列表或其他自然排序的集合 |
| `account[COMPANYNAME]` | 指示由Map属性帐户的KEYNAME键索引的映射条目的值 |

下面你会找到一些使用BeanWrapper来获取和设置属性的例子。

（如果您不打算直接使用BeanWrapper，那么下一部分对您来说并不重要。如果您只是使用DataBinder和BeanFactory及其开箱即用的实现，您应该跳到 有关PropertyEditors的部分。）

考虑以下两个类：

```java
public class Company {

    private String name;
    private Employee managingDirector;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Employee getManagingDirector() {
        return this.managingDirector;
    }

    public void setManagingDirector(Employee managingDirector) {
        this.managingDirector = managingDirector;
    }
}
```

```java
public class Employee {

    private String name;

    private float salary;

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public float getSalary() {
        return salary;
    }

    public void setSalary(float salary) {
        this.salary = salary;
    }
}
```

以下代码片段显示了如何检索和操作实例化Company和Employee的某些属性的一些示例：

```java
BeanWrapper company = new BeanWrapperImpl(new Company());
// setting the company name..
company.setPropertyValue("name", "Some Company Inc.");
// ... can also be done like this:
PropertyValue value = new PropertyValue("name", "Some Company Inc.");
company.setPropertyValue(value);

// ok, let's create the director and tie it to the company:
BeanWrapper jim = new BeanWrapperImpl(new Employee());
jim.setPropertyValue("name", "Jim Stravinsky");
company.setPropertyValue("managingDirector", jim.getWrappedInstance());

// retrieving the salary of the managingDirector through the company
Float salary = (Float) company.getPropertyValue("managingDirector.salary");
```

### 9.4.2 内置PropertyEditor实现

Spring使用PropertyEditors的概念来实现Object和String之间的转换。 如果您考虑一下，有时可能会以与对象本身不同的方式表示属性。 例如，Date可以用人类可读的方式表示（如String'2007-14-09'），而我们仍然能够将人类可读的表单转换回原始日期（甚至更好：转换任何 日期以人类可读的形式输入，返回日期对象）。 可以通过注册java.beans.PropertyEditor类型的自定义编辑器来实现此行为。 在BeanWrapper上注册自定义编辑器，或者如前一章所述，在特定的IoC容器中注册自定义编辑器，使其了解如何将属性转换为所需类型。 阅读Oracle提供的java.beans包的javadocs中有关PropertyEditors的更多信息。

在Spring中使用属性编辑的几个示例：

* 在bean上设置属性是使用PropertyEditors完成的。 当提到java.lang.String作为你在XML文件中声明的某个bean的属性的值时，Spring将（如果相应属性的setter具有Class参数）使用ClassEditor尝试将参数解析为 一个Class对象。
* 解析Spring的MVC框架中的HTTP请求参数是使用各种PropertyEditor完成的，您可以在CommandController的所有子类中手动绑定它们。

Spring有许多内置的PropertyEditor，可以让生活变得轻松。 其中每个都列在下面，它们都位于org.springframework.beans.propertyeditors包中。 大多数（但不是全部）（如下所示）默认由BeanWrapperImpl注册。 如果属性编辑器可以某种方式配置，您当然可以注册自己的变体来覆盖默认变量：

| Class | Explanation |
| :--- | :--- |
| `ByteArrayPropertyEditor` | 字节数组的编辑器。 字符串将简单地转换为其对应的字节表示。 BeanWrapperImpl默认注册。 |
| `ClassEditor` | 解析表示类到实际类的字符串，反之亦然。 找不到类时，抛出IllegalArgumentException。 BeanWrapperImpl默认注册。 |
| `CustomBooleanEditor` | 布尔属性的可自定义属性编辑器。 默认情况下由BeanWrapperImpl注册，但是，可以通过将其自定义实例注册为自定义编辑器来覆盖。 |
| `CustomCollectionEditor` | 集合的属性编辑器，将任何源集合转换为给定的目标集合类型。 |
| `CustomDateEditor` | java.util.Date的可自定义属性编辑器，支持自定义DateFormat。 没有默认注册。 必须根据需要以适当的格式注册用户。 |
| `CustomNumberEditor` | Customizable property editor for any Number subclass like`Integer`,`Long`,`Float`,`Double`. Registered by default by`BeanWrapperImpl`, but can be overridden by registering custom instance of it as a custom editor. |
| `FileEditor` | Capable of resolving Strings to`java.io.File`objects. Registered by default by`BeanWrapperImpl`. |
| `InputStreamEditor` | One-way property editor, capable of taking a text string and producing \(via an intermediate`ResourceEditor`and`Resource`\) an`InputStream`, so`InputStream`properties may be directly set as Strings. Note that the default usage will not close the`InputStream`for you! Registered by default by`BeanWrapperImpl`. |
| `LocaleEditor` | 能够将字符串解析为Locale对象，反之亦然（String格式为\[country\] \[variant\]，这与Locale提供的toString（）方法相同）。 BeanWrapperImpl默认注册。 |
| `PatternEditor` | Capable of resolving Strings to`java.util.regex.Pattern`objects and vice versa. |
| `PropertiesEditor` | Capable of converting Strings \(formatted using the format as defined in the javadocs of the`java.util.Properties`class\) to`Properties`objects. Registered by default by`BeanWrapperImpl`. |
| `StringTrimmerEditor` | Property editor that trims Strings. Optionally allows transforming an empty string into a`null`value. NOT registered by default; must be user registered as needed. |
| `URLEditor` | Capable of resolving a String representation of a URL to an actual`URL`object. Registered by default by`BeanWrapperImpl`. |

Spring使用java.beans.PropertyEditorManager来设置可能需要的属性编辑器的搜索路径。 搜索路径还包括sun.bean.editors，其中包括Font，Color和大多数基本类型等类型的PropertyEditor实现。 另请注意，标准JavaBeans基础结构将自动发现PropertyEditor类（无需显式注册），如果它们与它们处理的类位于同一个包中，并且与该类具有相同的名称，并附加“编辑器”; 例如，可以使用以下类和包结构，这足以使FooEditor类被识别并用作Foo类型属性的PropertyEditor。

```
com
  chank
    pop
      Foo
      FooEditor // the PropertyEditor for the Foo class
```

请注意，您也可以在此处使用标准BeanInfo JavaBeans机制（在此处未详细描述）。 下面是使用BeanInfo机制显式注册一个或多个PropertyEditor实例以及相关类的属性的示例。

```
com
  chank
    pop
      Foo
      FooBeanInfo // the BeanInfo for the Foo class
```

这是引用的FooBeanInfo类的Java源代码。 这会将CustomNumberEditor与Foo类的age属性相关联。

```java
public class FooBeanInfo extends SimpleBeanInfo {

    public PropertyDescriptor[] getPropertyDescriptors() {
        try {
            final PropertyEditor numberPE = new CustomNumberEditor(Integer.class, true);
            PropertyDescriptor ageDescriptor = new PropertyDescriptor("age", Foo.class) {
                public PropertyEditor createPropertyEditor(Object bean) {
                    return numberPE;
                };
            };
            return new PropertyDescriptor[] { ageDescriptor };
        }
        catch (IntrospectionException ex) {
            throw new Error(ex.toString());
        }
    }
}
```

#### 注册其他自定义PropertyEditors

将bean属性设置为字符串值时，Spring IoC容器最终使用标准JavaBeans PropertyEditors将这些字符串转换为属性的复杂类型。 Spring预先注册了许多自定义PropertyEditor（例如，将表示为字符串的类名转换为真正的Class对象）。此外，Java的标准JavaBeans PropertyEditor查找机制允许对类的PropertyEditor进行适当的命名，并将其放置在与其提供支持的类相同的包中，以便自动找到。

如果需要注册其他自定义PropertyEditor，可以使用多种机制。通常不方便或推荐的最常用的方法是简单地使用ConfigurableBeanFactory接口的registerCustomEditor（）方法，假设您有BeanFactory引用。另一种稍微方便的机制是使用一个名为CustomEditorConfigurer的特殊bean工厂后处理器。虽然bean工厂后处理器可以与BeanFactory实现一起使用，但CustomEditorConfigurer具有嵌套属性设置，因此强烈建议将它与ApplicationContext一起使用，它可以以类似于任何其他bean的方式部署，并自动检测并应用。

请注意，所有bean工厂和应用程序上下文都会自动使用许多内置属性编辑器，通过使用称为BeanWrapper的东西来处理属性转换。 BeanWrapper注册的标准属性编辑器在上一节中列出。此外，ApplicationContexts还会覆盖或添加其他数量的编辑器，以适合特定应用程序上下文类型的方式处理资源查找。

标准JavaBeans PropertyEditor实例用于将表示为字符串的属性值转换为属性的实际复杂类型。 CustomEditorConfigurer是一个bean工厂后处理器，可用于方便地将其他PropertyEditor实例的支持添加到ApplicationContext。

考虑用户类ExoticType，以及需要将ExoticType设置为属性的另一个类DependsOnExoticType：

```java
package example;

public class ExoticType {

    private String name;

    public ExoticType(String name) {
        this.name = name;
    }
}

public class DependsOnExoticType {

    private ExoticType type;

    public void setType(ExoticType type) {
        this.type = type;
    }
}
```

在正确设置内容之后，我们希望能够将type属性指定为字符串，PropertyEditor将在后台将其转换为实际的ExoticType实例：

```
<bean id="sample" class="example.DependsOnExoticType">
    <property name="type" value="aNameForExoticType"/>
</bean>
```

PropertyEditor实现看起来类似于：

```java
// converts string representation to ExoticType object
package example;

public class ExoticTypeEditor extends PropertyEditorSupport {

    public void setAsText(String text) {
        setValue(new ExoticType(text.toUpperCase()));
    }
}
```

最后，我们使用CustomEditorConfigurer向ApplicationContext注册新的PropertyEditor，然后可以根据需要使用它：

```
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
    <property name="customEditors">
        <map>
            <entry key="example.ExoticType" value="example.ExoticTypeEditor"/>
        </map>
    </property>
</bean>
```




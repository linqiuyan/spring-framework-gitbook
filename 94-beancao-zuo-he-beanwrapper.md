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



#### 注册其他自定义PropertyEditors




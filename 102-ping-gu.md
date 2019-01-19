## 10.2 评估

本节介绍SpEL接口及其表达式语言的简单使用。 完整的语言参考可以在语言参考一节中找到。

以下代码介绍了用于评估文字字符串表达式“Hello World”的SpEL API。

```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'");
String message = (String) exp.getValue();
```

消息变量的值只是“Hello World”。

您最有可能使用的SpEL类和接口位于包org.springframework.expression及其子包和spel.support中。

ExpressionParser接口负责解析表达式字符串。 在此示例中，表达式字符串是由周围的单引号表示的字符串文字。 接口Expression负责评估先前定义的表达式字符串。 在分别调用parser.parseExpression和exp.getValue时，可以抛出两个异常，ParseException和EvaluationException。

SpEL支持广泛的功能，例如调用方法，访问属性和调用构造函数。

作为方法调用的示例，我们在字符串文字上调用concat方法。

```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')");
String message = (String) exp.getValue();
```

消息的价值现在是'Hello World！'。

作为调用JavaBean属性的示例，可以调用String属性Bytes，如下所示。

```
ExpressionParser parser = new SpelExpressionParser();

// invokes 'getBytes()'
Expression exp = parser.parseExpression("'Hello World'.bytes");
byte[] bytes = (byte[]) exp.getValue();
```

SpEL还支持使用标准点表示法的嵌套属性，即prop1.prop2.prop3和属性值的设置

也可以访问公共字段。

```
ExpressionParser parser = new SpelExpressionParser();

// invokes 'getBytes().length'
Expression exp = parser.parseExpression("'Hello World'.bytes.length");
int length = (Integer) exp.getValue();
```

可以调用String的构造函数而不是使用字符串文字。

```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("new String('hello world').toUpperCase()");
String message = exp.getValue(String.class);
```

注意使用泛型方法public &lt;T&gt; T getValue（Class &lt;T&gt; desiredResultType）。 使用此方法无需将表达式的值强制转换为所需的结果类型。 如果无法将值强制转换为类型T或使用已注册的类型转换器转换，则抛出EvaluationException。

SpEL的更常见用法是提供针对特定对象实例（称为根对象）计算的表达式字符串。 该示例显示如何从Inventor类的实例检索name属性或创建布尔条件：

```
// Create and set a calendar
GregorianCalendar c = new GregorianCalendar();
c.set(1856, 7, 9);

// The constructor arguments are name, birthday, and nationality.
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();

Expression exp = parser.parseExpression("name");
String name = (String) exp.getValue(tesla);
// name == "Nikola Tesla"

exp = parser.parseExpression("name == 'Nikola Tesla'");
boolean result = exp.getValue(tesla, Boolean.class);
// result == true
```



### 10.2.1 EvaluationContext

在评估表达式以解析属性，方法，字段以及帮助执行类型转换时，将使用接口EvaluationContext。 有两个开箱即用的实现。

* SimpleEvaluationContext  - 为不需要SpEL语言语法的完整范围的表达式类别公开必要的SpEL语言特性和配置选项的子集，并且应该被有意义地限制。 示例包括但不限于数据绑定表达式，基于属性的过滤器等。
* StandardEvaluationContext  - 公开完整的SpEL语言功能和配置选项。 您可以使用它来指定默认根对象，并配置每个可用的与评估相关的策略。

SimpleEvaluationContext旨在仅支持SpEL语言语法的子集。 它排除了Java类型引用，构造函数和bean引用。 它还需要明确选择表达式中属性和方法的支持级别。 默认情况下，create（）静态工厂方法仅启用对属性的读访问权限。 您还可以获取构建器以配置所需的确切支持级别，定位以下某个或以下组合：

1. 仅限自定义PropertyAccessor（无反射）。
2. 只读访问的数据绑定属性。
3. 读写的数据绑定属性。

#### 类型转换 Type conversion

默认情况下，SpEL使用Spring核心中提供的转换服务（org.springframework.core.convert.ConversionService）。 此转换服务附带了许多内置的转换器，可用于常见转换，但也可完全扩展，因此可以添加类型之间的自定义转换。 此外，它具有通用感知的关键功能。 这意味着当在表达式中使用泛型类型时，SpEL将尝试转换以维护它遇到的任何对象的类型正确性。

这在实践中意味着什么？ 假设使用setValue（）的赋值用于设置List属性。 该属性的类型实际上是List &lt;Boolean&gt;。 SpEL将识别列表中的元素在放入其中之前需要转换为布尔值。 一个简单的例子：

```java
class Simple {
    public List<Boolean> booleanList = new ArrayList<Boolean>();
}

Simple simple = new Simple();
simple.booleanList.add(true);

SimpleEvaluationContext context = SimpleEvaluationContext().create();

// false is passed in here as a string. SpEL and the conversion service will
// correctly recognize that it needs to be a Boolean and convert it

parser.parseExpression("booleanList[0]").setValue(context, simple, "false");

// b will be false
Boolean b = simple.booleanList.get(0);
```



### 10.2.2  解析器配置

可以使用解析器配置对象（org.springframework.expression.spel.SpelParserConfiguration）配置SpEL表达式解析器。 配置对象控制某些表达式组件的行为。 例如，如果索引到数组或集合并且指定索引处的元素为null，则可以自动创建该元素。 当使用由一系列属性引用组成的表达式时，这很有用。 如果索引到数组或列表并指定超出数组或列表当前大小末尾的索引，则可以自动增大数组或列表以适应该索引。

```
class Demo {
    public List<String> list;
}

// Turn on:
// - auto null reference initialization
// - auto collection growing
SpelParserConfiguration config = new SpelParserConfiguration(true,true);

ExpressionParser parser = new SpelExpressionParser(config);

Expression expression = parser.parseExpression("list[3]");

Demo demo = new Demo();

Object o = expression.getValue(demo);

// demo.list will now be a real collection of 4 entries
// Each entry is a new empty String
```

也可以配置SpEL表达式编译器的行为。

### 10.2.3  SpEL编译

Spring Framework 4.1包含一个基本的表达式编译器。表达式通常被解释为在评估期间提供了很多动态灵活性，但是没有提供最佳性能。对于偶尔的表达式使用，这很好，但是当其他组件（如Spring Integration）使用时，性能可能非常重要，并且不需要动态。

新的SpEL编译器旨在满足这一需求。编译器将在评估期间动态生成一个真正的Java类，它体现了表达式行为并使用它来实现更快的表达式评估。由于缺少表达式类型，编译器使用在执行编译时对表达式的解释评估期间收集的信息。例如，它不完全从表达式中知道属性引用的类型，但在第一次解释的求值期间，它将找出它是什么。当然，如果各种表达元素的类型随时间变化，那么基于此信息的编译可能会导致麻烦。因此，编译最适合于在重复评估时类型信息不会改变的表达式。

对于这样的基本表达式：

```
someArray[0].someProperty.someOtherProperty < 0.1

```

它涉及数组访问，一些属性derefencing和数字操作，性能增益可以非常明显。 在50000次迭代的微基准测试示例中，仅使用解释器评估75ms，使用表达式的编译版本仅需3ms。



#### 编译器配置

默认情况下，编译器未打开，但有两种方法可以打开它。 当SpEL使用嵌入在另一个组件中时，可以使用前面讨论的解析器配置过程或通过系统属性打开它。 本节讨论这两个选项。

重要的是要理解编译器可以运行的一些模式，在枚举（org.springframework.expression.spel.SpelCompilerMode）中捕获。 模式如下：

* OFF  - 关闭编译器; 这是默认值。
* IMMEDIATE  - 在立即模式下，表达式会尽快编译。 这通常是在第一次解释评估之后。 如果编译的表达式失败（通常由于类型更改，如上所述），则表达式求值的调用者将收到异常。
* MIXED  - 在混合模式下，表达式随着时间的推移在解释和编译模式之间静默切换。 经过一些解释后的运行，它们将切换到编译形式，如果编译后的表单出现问题（如类型更改，如上所述），表达式将自动再次切换回解释形式。 稍后它可能会生成另一个已编译的表单并切换到它。 基本上，用户进入IMMEDIATE模式的异常是在内部处理。

存在IMMEDIATE模式，因为MIXED模式可能会导致具有副作用的表达式出现问题。 如果编译后的表达式在部分成功后爆炸，则可能已经完成了影响系统状态的事情。 如果发生这种情况，调用者可能不希望它以解释模式静默重新运行，因为表达式的一部分可能正在运行两次。

选择模式后，使用SpelParserConfiguration配置解析器：

```
SpelParserConfiguration config = new SpelParserConfiguration(SpelCompilerMode.IMMEDIATE,
    this.getClass().getClassLoader());

SpelExpressionParser parser = new SpelExpressionParser(config);

Expression expr = parser.parseExpression("payload");

MyMessage message = new MyMessage();

Object payload = expr.getValue(message);
```

指定编译器模式时，也可以指定类加载器（允许传递null）。 编译表达式将在任何提供的子类加载器中定义。 重要的是要确保是否指定了类加载器，它可以看到表达式评估过程中涉及的所有类型。 如果未指定，则将使用默认的类加载器（通常是表达式求值期间运行的线程的上下文类加载器）。

配置编译器的第二种方法是在SpEL嵌入到某个其他组件中时使用，并且可能无法通过配置对象进行配置。 在这些情况下，可以使用系统属性。 spring.expression.compiler.mode属性可以设置为SpelCompilerMode枚举值之一（off，immediate或mixed）。

#### 编译器限制

使用Spring Framework 4.1，基本的编译框架已经到位。 但是，该框架尚不支持编译各种表达式。 最初的重点是可能在性能关键环境中使用的常用表达式。 目前无法编译这些表达式：

* 涉及转让的表达
* 表达式依赖于转换服务
* 使用自定义解析器或访问器的表达式
* 使用选择或投影的表达式

将来可以编写越来越多的表达类型。




## 10.4  语言参考

### 10.4.1  文字表达

支持的文字表达式的类型是字符串，数值（int，real，hex），boolean和null。 字符串由单引号分隔。 要将单引号本身放在字符串中，请使用两个单引号字符。

以下清单显示了文字的简单用法。 通常，它们不会像这样单独使用，而是作为更复杂表达式的一部分使用，例如在逻辑比较运算符的一侧使用文字。

```java
ExpressionParser parser = new SpelExpressionParser();

// evals to "Hello World"
String helloWorld = (String) parser.parseExpression("'Hello World'").getValue();

double avogadrosNumber = (Double) parser.parseExpression("6.0221415E+23").getValue();

// evals to 2147483647
int maxValue = (Integer) parser.parseExpression("0x7FFFFFFF").getValue();

boolean trueValue = (Boolean) parser.parseExpression("true").getValue();

Object nullValue = parser.parseExpression("null").getValue();
```

数字支持使用负号，指数表示法和小数点。 默认情况下，使用Double.parseDouble（）解析实数。

### 10.4.2  Properties, Arrays, Lists, Maps, Indexers

使用属性引用进行导航很简单：只需使用句点来指示嵌套属性值。 Inventor类，pupin和tesla的实例填充了示例中使用的类中列出的数据。 为了“向下”导航并获得特斯拉的出生年份和普平的出生城市，使用以下表达方式。

```
// evals to 1856
int year = (Integer) parser.parseExpression("Birthdate.Year + 1900").getValue(context);

String city = (String) parser.parseExpression("placeOfBirth.City").getValue(context);
```

属性名称的第一个字母允许不区分大小写。 使用方括号表示法获得数组和列表的内容。

```java
ExpressionParser parser = new SpelExpressionParser();
SimpleEvaluationContext context = SimpleEvaluationContext.create();

// Inventions Array

// evaluates to "Induction motor"
String invention = parser.parseExpression("inventions[3]").getValue(
        context, tesla, String.class);

// Members List

// evaluates to "Nikola Tesla"
String name = parser.parseExpression("Members[0].Name").getValue(
        context, ieee, String.class);

// List and Array navigation
// evaluates to "Wireless communication"
String invention = parser.parseExpression("Members[0].Inventions[6]").getValue(
        context, ieee, String.class);
```

通过指定括号内的文字键值来获取映射的内容。 在这种情况下，因为人员映射的键是字符串，我们可以指定字符串文字。

```java
// Officer's Dictionary

Inventor pupin = parser.parseExpression("Officers['president']").getValue(
        societyContext, Inventor.class);

// evaluates to "Idvor"
String city = parser.parseExpression("Officers['president'].PlaceOfBirth.City").getValue(
        societyContext, String.class);

// setting values
parser.parseExpression("Officers['advisors'][0].PlaceOfBirth.Country").setValue(
        societyContext, "Croatia");
```

### 10.4.3  内联列表 Inline lists

列表可以使用{}表示法直接表达在表达式中。

```
// evaluates to a Java list containing the four numbers
List numbers = (List) parser.parseExpression("{1,2,3,4}").getValue(context);

List listOfLists = (List) parser.parseExpression("{{'a','b'},{'x','y'}}").getValue(context);
```

{}本身就是一个空列表。 出于性能原因，如果列表本身完全由固定文字组成，则创建常量列表以表示表达式，而不是在每个评估上构建新列表。

### 10.4.4  内联地图 Inline Maps

也可以使用{key：value}表示法直接在表达式中表达地图。

```
// evaluates to a Java map containing the two entries
Map inventorInfo = (Map) parser.parseExpression("{name:'Nikola',dob:'10-July-1856'}").getValue(context);

Map mapOfMaps = (Map) parser.parseExpression("{name:{first:'Nikola',last:'Tesla'},dob:{day:10,month:'July',year:1856}}").getValue(context);
```

{：}本身就是一张空地图。 出于性能原因，如果地图本身由固定文字或其他嵌套常量结构（列表或地图）组成，则会创建一个常量地图来表示表达式，而不是在每次评估时构建新地图。 引用地图键是可选的，上面的示例不使用引用键。

### 10.4.5  阵列构造 Array construction

可以使用熟悉的Java语法构建数组，可选地提供初始化程序以在构造时填充数组。

```
int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue(context);

// Array with initializer
int[] numbers2 = (int[]) parser.parseExpression("new int[]{1,2,3}").getValue(context);

// Multi dimensional array
int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue(context);
```

当前不允许在构造多维阵列时提供初始化器。



### 10.4.6  方法

使用典型的Java编程语法调用方法。 您也可以在文字上调用方法。 Varargs也受支持。

```
// string literal, evaluates to "bc"
String bc = parser.parseExpression("'abc'.substring(1, 3)").getValue(String.class);

// evaluates to true
boolean isMember = parser.parseExpression("isMember('Mihajlo Pupin')").getValue(
        societyContext, Boolean.class);
```

### 10.4.7 运算符 Operators

#### 关系运算符

关系运算符; 使用标准运算符表示法支持等于，不等于，小于，小于或等于，大于等于或等于。

```
// evaluates to true
boolean trueValue = parser.parseExpression("2 == 2").getValue(Boolean.class);

// evaluates to false
boolean falseValue = parser.parseExpression("2 < -5.0").getValue(Boolean.class);

// evaluates to true
boolean trueValue = parser.parseExpression("'black' < 'block'").getValue(Boolean.class);
```

大于/小于与null的比较遵循一个简单的规则：null在此处被视为空（即不为零）。 因此，任何其他值始终大于null（X&gt; null始终为true），并且其他任何值都不会小于任何值（X &lt;null始终为false）。

如果您更喜欢数字比较，请避免基于数字的空比较，以支持与零进行比较（例如X&gt; 0或X &lt;0）。

除标准关系运算符外，SpEL还支持基于instanceof和正则表达式的匹配运算符。

```
// evaluates to false
boolean falseValue = parser.parseExpression(
        "'xyz' instanceof T(Integer)").getValue(Boolean.class);

// evaluates to true
boolean trueValue = parser.parseExpression(
        "'5.00' matches '\^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);

//evaluates to false
boolean falseValue = parser.parseExpression(
        "'5.0067' matches '\^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);
```

小心原始类型，因为它们立即被装箱到包装类型，因此1个实例的T（int）求值为false，而1个实例的T（整数）求值为true，如预期的那样。

每个符号运算符也可以指定为纯字母等价运算符。 这避免了所使用的符号对于嵌入表达式的文档类型具有特殊含义的问题（例如，XML文档）。 文本等价物如下所示：lt（&lt;），gt（&gt;），le（⇐），ge（&gt; =），eq（==），ne（！=），div（/），mod（％）， 不是（！）。 这些不区分大小写。



#### 逻辑运算符

支持的逻辑运算符是和，或，而不是。 它们的用途如下所示。

```
// -- AND --

// evaluates to false
boolean falseValue = parser.parseExpression("true and false").getValue(Boolean.class);

// evaluates to true
String expression = "isMember('Nikola Tesla') and isMember('Mihajlo Pupin')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- OR --

// evaluates to true
boolean trueValue = parser.parseExpression("true or false").getValue(Boolean.class);

// evaluates to true
String expression = "isMember('Nikola Tesla') or isMember('Albert Einstein')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- NOT --

// evaluates to false
boolean falseValue = parser.parseExpression("!true").getValue(Boolean.class);

// -- AND and NOT --
String expression = "isMember('Nikola Tesla') and !isMember('Mihajlo Pupin')";
boolean falseValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);
```

#### 数学运算符

加法运算符可用于数字和字符串。 减法，乘法和除法只能用于数字。 支持的其他数学运算符是模数（％）和指数幂（^）。 强制执行标准运算符优先级。 这些运算符如下所示。

```
// Addition
int two = parser.parseExpression("1 + 1").getValue(Integer.class); // 2

String testString = parser.parseExpression(
        "'test' + ' ' + 'string'").getValue(String.class); // 'test string'

// Subtraction
int four = parser.parseExpression("1 - -3").getValue(Integer.class); // 4

double d = parser.parseExpression("1000.00 - 1e4").getValue(Double.class); // -9000

// Multiplication
int six = parser.parseExpression("-2 * -3").getValue(Integer.class); // 6

double twentyFour = parser.parseExpression("2.0 * 3e0 * 4").getValue(Double.class); // 24.0

// Division
int minusTwo = parser.parseExpression("6 / -3").getValue(Integer.class); // -2

double one = parser.parseExpression("8.0 / 4e0 / 2").getValue(Double.class); // 1.0

// Modulus
int three = parser.parseExpression("7 % 4").getValue(Integer.class); // 3

int one = parser.parseExpression("8 / 5 % 2").getValue(Integer.class); // 1

// Operator precedence
int minusTwentyOne = parser.parseExpression("1+2-3*8").getValue(Integer.class); // -21
```

### 10.4.8  分配

通过使用赋值运算符来设置属性。 这通常在调用setValue时完成，但也可以在调用getValue时完成。

```
Inventor inventor = new Inventor();
SimpleEvaluationContext context = SimpleEvaluationContext.create();

parser.parseExpression("Name").setValue(context, inventor, "Alexander Seovic2");

// alternatively

String aleks = parser.parseExpression(
        "Name = 'Alexandar Seovic'").getValue(context, inventor, String.class);
```

### 10.4.9 类型

特殊的T运算符可用于指定java.lang.Class（类型）的实例。 也使用此运算符调用静态方法。 StandardEvaluationContext使用TypeLocator查找类型，StandardTypeLocator（可以替换）是在了解java.lang包的基础上构建的。 这意味着对java.lang中的类型的T（）引用不需要完全限定，但所有其他类型引用必须是。

```java
Class dateClass = parser.parseExpression("T(java.util.Date)").getValue(Class.class);

Class stringClass = parser.parseExpression("T(String)").getValue(Class.class);

boolean trueValue = parser.parseExpression(
        "T(java.math.RoundingMode).CEILING < T(java.math.RoundingMode).FLOOR")
        .getValue(Boolean.class);
```

### 10.4.10 构造函数

### 10.4.11 变量

#### \#this和\#root变量

### 10.4.12  功能

### 10.4.13  Bean引用

如果已使用bean解析器配置了评估上下文，则可以使用（@）符号从表达式中查找bean。

```

ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = StandardEvaluationContext.create();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"foo") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("@foo").getValue(context);
```

要访问工厂bean本身，bean名称应该以（＆）符号为前缀。

```
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = StandardEvaluationContext.create();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"&foo") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("&foo").getValue(context);
```

### 10.4.14  三元运算符（If-Then-Else）

### 10.4.15  猫王运营商

### 10.4.16  安全导航操作员

### 10.4.17  收藏选择

### 10.4.18  收集投影

### 10.4.19  表达模板




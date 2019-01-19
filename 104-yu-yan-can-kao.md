## 10.4  语言参考

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



### 10.4.1  文字表达

### 10.4.2  属性，数组，列表，地图，索引器

### 10.4.3  内联列表

### 10.4.4  内联地图

### 10.4.5  阵列构造

### 10.4.6  方法

### 10.4.7 运营商

#### 关系运算符

#### 逻辑运算符

#### 数学运算符

### 10.4.8  分配

### 10.4.9 类型

### 10.4.10 构造函数

### 10.4.11 变量

#### \#this和\#root变量

### 10.4.12  功能

### 10.4.13  Bean引用

### 10.4.14  三元运算符（If-Then-Else）

### 10.4.15  猫王运营商

### 10.4.16  安全导航操作员

### 10.4.17  收藏选择

### 10.4.18  收集投影

### 10.4.19  表达模板




## 10.3  bean定义中的表达式

SpEL表达式可以与XML或基于注释的配置元数据一起使用，以定义BeanDefinitions。 在这两种情况下，定义表达式的语法都是＃{&lt;expression string&gt;}形式。

### 10.3.1 XML配置

可以使用表达式设置属性或构造函数-arg值，如下所示。

```bash
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>

    <!-- other properties -->
</bean>
```



### 10.3.2 注释配置




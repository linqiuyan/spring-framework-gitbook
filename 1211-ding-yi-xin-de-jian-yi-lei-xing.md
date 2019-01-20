## 12.11 定义新的建议类型 Defining new Advice types

Spring AOP旨在可扩展。 虽然拦截实现策略目前在内部使用，但除了围绕建议的开箱即用拦截之外，还可以支持任意建议类型，之前，抛出建议和返回建议之后。

org.springframework.aop.framework.adapter包是一个SPI包，允许在不更改核心框架的情况下添加对新的自定义通知类型的支持。 自定义建议类型的唯一约束是它必须实现org.aopalliance.aop.Advice标记接口。

有关更多信息，请参阅org.springframework.aop.framework.adapter javadocs。






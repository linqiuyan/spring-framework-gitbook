## 8.1  介绍

遗憾的是，Java的标准java.net.URL类和各种URL前缀的标准处理程序不足以完全访问低级资源。 例如，没有标准化的URL实现可用于访问需要从类路径或相对于ServletContext获取的资源。 虽然可以为专用的URL前缀注册新的处理程序（类似于http :\)这样的前缀的现有处理程序，但这通常非常复杂，并且URL接口仍然缺少一些理想的功能，例如检查存在的方法 被指向的资源。




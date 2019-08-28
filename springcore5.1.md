Spring5学习
=========

摘自[spring官网](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/core.html#beans)

# 1.IoC容器

## 1.1.IoC容器与Beans的介绍

本章涉及了Spring框架实现的控制反转（IoC）的原理。（参见[Inversion of Control](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/overview.html#background-ioc)）IoC也称为依赖注入（DI）。这是一个对象定义其依赖关系的过程，对象定义其依赖只能通过构造函数传入参数，工厂方法传入参数或者对构造函数或工厂方法返回的实例设置其属性。当Spring容器创建bean时，它会将这些依赖注入bean属性中。这个过程就相当于原本bean自己去获取自己的依赖（主动获取资源，使用new关键字），而现在变成了Spring容器实例化好资源，并由Spring来注入到bean中对应的属性中（被动接受资源）。

`org.springframework.beans`和`org.springframework.context`这两个包是实现Spring框架中IoC容器的基础包。[BeanFactory](https://docs.spring.io/spring-framework/docs/5.1.0.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)接口
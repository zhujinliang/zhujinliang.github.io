---
layout: post
title: "Spring注解@Component、@Repository、@Service、@Controller区别"
category: "Java"
date: 2016-03-06
---


## 简介

Spring 2.5 中除了提供 @Component 注释外，还定义了几个拥有特殊语义的注释，它们分别是：@Repository、@Service 、@Controller。

在目前的 Spring 版本中，这 3 个注释和 @Component 是等效的，但是从注释类的命名上，很容易看出这 3 个注释分别和持久层、业务层和控制层（Web 层）相对应。虽然目前这 3个注释和 @Component 相比没有什么新意，但 Spring 将在以后的版本中为它们添加特殊的功能。所以，如果 Web应用程序采用了经典的三层分层结构的话，最好这么分层使用：

* `@Repository` 持久层DAO
* `@Service` 业务层Service
* `@Controller` 控制层Controller
* `@Component` 对那些比较中立的类

<!-- more -->

## 使用

在一个稍大的项目中，通常会有上百个组件，如果这些组件采用xml的bean定义来配置，显然会增加配置文件的体积，查找以及维护起来也不太方便。 Spring2.5为我们引入了组件自动扫描机制，他可以在类路径底下寻找标注了@Component,@Service,@Controller,@Repository注解的类，并把这些类纳入进spring容器中管理。它的作用和在xml文件中使用bean节点配置组件时一样的。要使用自动扫描机制，我们需要打开以下配置信息： 

```
<?xml version="1.0" encoding="UTF-8" ?>
<beansxmlns="http://www.springframework.org/schema/beans"xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"xmlns:context="http://www.springframework.org/schema/context"xsi:schemaLocation="http://www.springframework.org/schema/beans  http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context-2.5.xsd" 

 <context:annotation-config/>
 <context:component-scan base-package=”cn.zhujinliang.hello”>   
</beans>   


```

其中`base-package`为需要扫描的包（含所有子包）。

Java代码：
  
```
@Service
public class HelloServiceImpl implements HelloService{   
} 

@Repository
public class HelloDaoImpl implements HelloDao { 
}

// 可以使用以下方式指定初始化方法和销毁方法（方法名任意）：

@PostConstruct
public void init() { 
} 

@PreDestroy
public void destory() { 
} 

```
*注意：* getBean的默认名称是类名（头字母小写），如果想自定义，可以`@Service("helloService")`这样来指定。这种bean默认是单例的，如果想改变，可以使用

```
@Service("helloService")
@Scope("prototype")
public class HelloServiceImpl implements HelloService{   
} 

```
来改变。


### 注入方式 

把DAO实现类注入到service实现类中，把service的接口(注意不要是service的实现类)注入到controller中，注入时不要new 这个注入的类，因为spring会自动注入，如果手动再new的话会出现错误，然后属性加上`@Autowired`后不需要getter()和setter()方法，Spring也会自动注入。

在接口前面标上`@Autowired`和`@Qualifier`注释使得接口可以被容器注入，当接口存在两个实现类的时候必须指定其中一个来注入，使用实现类首字母小写的字符串来注入，如： 

```
public class HelloView {
    @Autowired  
    @Qualifier("chinese")
    private Man man; 
}
```   
否则可以省略，只写`@Autowired`。 


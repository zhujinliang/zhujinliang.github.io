---
layout: post
title: "Python中的类方法、类实例方法、静态方法的使用与区别"
category: "Python"
date: 2015-05-26
---

# 介绍

`function`就是可以通过名字可以调用的一段代码,我们可以传参数进去，得到返回值。所有的参数都是明确的传递过去的。

`method`是function与对象的结合。我们调用一个方法的时候，有些参数是隐含的传递过去的。

一般定义`class`的时候，会用到三种`method`，分别是：

* instance method   实例方法
* class method    类方法
* static method   静态方法

每一种的应用场景和定义方式有些区别，接下来我们详细介绍下。

<!-- more -->

## instance method

```
In [1]: class Human(object):
   ...:     def __init__(self, weight):
   ...:         self.weight = weight
   ...:     def get_weight(self):
   ...:         return self.weight
   ...:    
 
In [2]: Human.get_weight
Out[2]: <unbound method Human.get_weight>
```

这告诉我们get_weight是一个没有被绑定方法，什么叫做未绑定呢？继续看下去。

```
In [7]: Human.get_weight()
---------------------------------------------------------------------------
TypeError  Traceback (most recent call last)
/home/yao/learn/insight_python/<ipython-input-7-a2b2c5cd2f8d> in <module>()
----> 1 Human.get_weight()
 
TypeError: unbound method get_weight() must be called with Human instance as first argument (got nothing instead)
```

未绑定的方法必须使用一个Human实例作为第一个参数来调用啊。那我们来试试

```
In [10]: Human.get_weight(Human(45))
Out[10]: 45
```

果然成功了，但是一般情况下我们习惯这么使用。

```
In [11]: person = Human(45)
 
In [12]: person.get_weight()
Out[12]: 45
```

这两种方式的结果一模一样。我们看下官方文档是怎么解释这种现象的。


When an instance attribute is referenced that isn’t a data attribute, its class is searched.
If the name denotes a valid class attribute that is a function object, a method object is
created by packing (pointers to) the instance object and the function object just found together
in an abstract object: this is the method object. When the method object is called with an
argument list, a new argument list is constructed from the instance object and the argument list,
and the function object is called with this new argument list.

原来我们常用的调用方法`person.get_weight()`是把调用的实例隐藏的作为一个参数`self`传递过去了, self 只是一个普通的参数名称,不是关键字。这跟Java中的方法引用有点类似，如果上面的Human类是Java代码实现的话，通过方法引用`Human::get_weight`，那么再调用的时候，默认会把第一个参数变成方法的接收者，并且其他参数也传递给该方法。

    
```
In [13]: person.get_weight
Out[13]: <bound method Human.get_weight of <__main__.Human object at 0x8e13bec>>

In [14]: person
Out[14]: <__main__.Human at 0x8e13bec>
```

我们看到get_weight被绑定在了 person 这个实例对象上。

#### 总结

* instance method 就是实例对象与函数的结合。

* 使用类调用，第一个参数明确的传递过去一个实例。

* 使用实例调用，调用的实例被作为第一个参数被隐含的传递过去。

## class method

```
In [1]: class Human(object):
  ...:     weight = 12
  ...:     @classmethod
  ...:     def get_weight(cls):
  ...:         return cls.weight

In [2]: Human.get_weight
Out[2]: <bound method type.get_weight of <class '__main__.Human'>>
```

我们看到get_weight是一个绑定在 Human 这个类上的method。调用下看看

```
In [3]: Human.get_weight()
Out[3]: 12
In [4]: Human().get_weight()
Out[4]: 12
```

类和类的实例都能调用 get_weight 而且调用结果完全一样。
我们看到 weight 是属于 Human 类的属性，当然也是 Human 的实例的属性。那传递过去的参数 cls 是类还是实例呢？

```
In [1]: class Human(object):
   ...:     weight = 12
   ...:     @classmethod
   ...:     def get_weight(cls):
   ...:         print cls
 
In [2]: Human.get_weight()
<class '__main__.Human'>
 
In [3]: Human().get_weight()
<class '__main__.Human'>
```

我们看到传递过去的都是 Human 类,不是 Human 的实例，两种方式调用的结果没有任何区别。
`cls` 只是一个普通的函数参数，调用时被隐含的传递过去。

#### 总结

* classmethod 是类对象与函数的结合。

* 可以使用和类的实例调用，但是都是将类作为隐含参数传递过去。

* 使用类来调用 classmethod 可以避免将类实例化的开销。

## static method

```  
In [1]: class Human(object):
   ...:     @staticmethod
   ...:     def add(a, b):
   ...:         return a + b
   ...:     def get_weight(self):
   ...:         return self.add(1, 2)
 
In [2]: Human.add
Out[2]: <function __main__.add>
 
In [3]: Human().add
Out[3]: <function __main__.add>
 
In [4]: Human.add(1, 2)
Out[4]: 3
 
In [5]: Human().add(1, 2)
Out[5]: 3
```

我们看到 add 在无论是类还是实例上都只是一个普通的函数，并没有绑定在任何一个特定的类或者实例上。
可以使用**类**或者**类的实例**调用，并且**没有任何隐含参数的传入**。

```
In [6]: Human().add is Human().add
Out[6]: True
 
In [7]: Human().get_weight is Human().get_weight
Out[7]: False
```

add 在两个实例上也是同一个对象。
instance method 就不一样了，每次都会创建一个新的 get_weight 对象。

Java 中static 修饰的方法即是类方法。

#### 总结

* 当一个函数逻辑上属于一个类又不依赖与类的属性的时候，可以使用 staticmethod。

* 使用 staticmethod 可以避免每次使用的时都会创建一个对象的开销。

* staticmethod 可以使用类和类的实例调用。但是不依赖于类和类的实例的状态。

## 类方法、类实例方法、静态方法的使用与区别

@classmethod

@staticmethod

```
class A(object):
    def foo(self,x):
 
        print "executing foo(%s,%s)"%(self,x)
 
    @classmethod
    def class_foo(cls,x):
 
        print "executing class_foo(%s,%s)"%(cls,x)
 
    @staticmethod
    def static_foo(x):
 
        print "executing static_foo(%s)"%x

a = A()
 
a.foo(1)
 
a.class_foo(1)
A.class_foo(1)
 
a.static_foo(1)
A.static_foo(1)

输出的结果为
executing foo(<__main__.A object at 0x004E0730>,1)
executing class_foo(<class '__main__.A'>,1)
executing class_foo(<class '__main__.A'>,1)
executing static_foo(1)
executing static_foo(1)
```

　　首先，实例方法，很清楚，打印了该实例化的对象的信息（在内存中的地址）

　　然后是类方法，cls这个参数打印出的是 类A这个对象（Python中任何都是对象），不管是否是实例化的调用

　　最后是静态方法，与调用也无关，并且这个方法不依赖任何对象。

#### 总结
* 类方法和静态方法都可以被类和类实例调用，类实例方法仅可以被类实例调用

* 类方法的隐含调用参数是类，而类实例方法的隐含调用参数是类的实例，静态方法没有隐含调用参数

* 实例化以后，类方法和静态方法也可以使用

* 类方法和静态方法不能访问需要实例化的属性

* 实例方法是由一个类实例化后产生的，所以能访问实例化后的对象的属性

* 因为在Python中，类也是对象，所以，类方法相对静态方法的一大特点是可以访问类具有的属性，但是静态方法不行　！

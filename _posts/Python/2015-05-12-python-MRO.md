---
layout: post
title: "Python MRO 与多继承"
category: "Python"
date: 2015-05-12
---


# Python MRO 与多继承


MRO: Method Resolution Order（方法解析顺序）

MRO虽然叫方法解析顺序，但是它不仅针对方法搜索，对于类中的数据属性也适用。


目前版本 Python 中的 class 分为 classical 和 new-style 两大类。
其中 classical 是 python 一直沿用的，而 new-style 是 2.2 才开始引入的东西。

只要 class 继承于 object，或 bases class 里面任意一个继承于 object，这个 class 都是 new-style。

对于复杂的继承结构，class 中 method 的调用顺序（MRO）也是不同的。

* classical，古典类用的是简单的自左至右的深度优先方法，简单概括为**深入优先**。
* new-style 则是C3 MRO搜索方法，简单概括为**广度优先**。

<!-- more -->

Classical  深度优先:
    
    class D:
        def foo(self):
            print "class D"
               
    class B(D):
        pass
           
    class C(D):   
        def foo(self):
            print "class C"
               
    class A(B, C):
        pass
           
    f = A()
    f.foo()
    
    结果为：
    
    class D
先找父类B的foo方法，B类没有，再找B的父类D的foo方法，D有foo方法，就调用D的foo方法。


New-style   广度优先: 

    class D(object):
        def foo(self):
            print "class D"
                
    class B(D):
        pass
            
    class C(D):   
        def foo(self):
            print "class C"
                
    class A(B, C):
        pass
                
        f = A()
        f.foo()
    
    结果为：
    
    class C

先找父类B的foo方法，B没有，则找父类C的foo方法，C有，则调用C的foo方法。

两者的区别简单地可以通过下图来解释：

![多继承比较](http://zhujinliang.qiniudn.com/img/blog/python/python-mro.jpg)

关于MRO的搜索顺序我们也可以在new-style类中通过查看`__mro__`属性得到验证，输出如下：

    (__main__.A, __main__.B, __main__.C, __main__.D, object)




























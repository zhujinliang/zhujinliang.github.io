---
layout: post
title: "Python functools模块"
category: "Python"
date: 2015-03-05
---


# Python functools模块


functools主要有以下包：

* partial
* reduce
* update_wrapper
* wraps

`dir(functools)`查看

    ['WRAPPER_ASSIGNMENTS',
     'WRAPPER_UPDATES',
     '__builtins__',
     '__doc__',
     '__file__',
     '__name__',
     '__package__',
     'cmp_to_key',
     'partial',
     'reduce',
     'total_ordering',
     'update_wrapper',
     'wraps']


## partial

`partial`可以重新绑定函数的可选参数，生成一个callable的partial对象

`partial`常用场景：

* 修改函数签名
* 使用一些参数固化，以提供一个更简单的函数供以后调用

用法：

    from functools import partial
    from operator import add
    
    add1 = partial(add, 1)
    add1(3)
    # 结果
    4

## reduce
reduce, 和python内置的reduce是一样的

## update_wrapper

update_wrapper是wraps的主要功能提供者，它负责考贝原函数的属性，默认是：’__module__’, ‘__name__’, ‘__doc__’， ‘__dict__’。

update_wrapper把被封装函数的__name__、__module__、__doc__和 __dict__都复制到封装函数去

## wrapper

其实也就是相当于在调用了partial，使用wraps可以把整个function的属性都带上。

wraps主要是用来包装函数，使被包装含数更像原函数，它是对partial(update_wrapper, …)的简单包装.

    from functools import wraps
    def my_decorator(f):
        @wraps(f)
        def wrapper(*args, **kwds):
            print 'Calling decorated function'
            return f(*args, **kwds)
        return wrapper
    
    @my_decorator
    def example():
        """Docstring"""
        print 'Called example function'
    example()
    print example.__name__
    print example.__doc__
    
    # 结果
    Calling decorated function
    Called example function
    example
    Docstring
    



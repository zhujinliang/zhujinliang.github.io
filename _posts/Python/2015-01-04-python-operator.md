---
layout: post
title: "Python operator模块"
category: "Python"
date: 2015-01-04
---


# Python operator模块

operator模块是python中内置的操作符函数接口，它定义了一些算术和比较内置操作的函数。它是用c实现的，所以执行速度比python代码快。

<!-- more -->

## 逻辑操作

```
>>> a=2
>>> b=5
>>> from operator import *
>>> not_(a)
False
>>> truth(a)
True
>>> is_(a, b)
False
>>> is_not(a,b)
True
```

## 比较操作符
所有富比较操作符都得到支持。

```
from operator import *
a = 1
b = 5
for func in (lt, le, eq, ne, ge, gt):
    print funct(a,b)
```
这些函数等价于<、<=、==、>=和>的表达式语法。

## 算术操作符
处理数字的算术操作符也得到支持。
主要有:

* 正反操作

```
abs
neg
pos
```
* 算术操作

```
add
div
floordiv 整除除法
mod
mul
pow
sub
truediv(a,b) 浮点数除法
```
* 位操作

```
and_ 按位与
invert 取反
lshift(c,d) 左移位
or_(c,d) 按位或
rshift(d,c) 右移位
xor(c,d) 异或
```

## 序列操作符

处理序列的操作符可以分为四组：建立序列，搜索元素，访问内容和从序列中删除元素。

* 建立序列

```
a = [1,2,3]
b = ['a', 'b', 'c']
print concat(a,b)
print repeat(a,3)
```
* 搜索序列

```
print contains(a,1)    
print contains(b,"d")   
print countOf(a)
print countOf(b,"d")
print indexOf(a,1)
```

* 访问序列

```
>>> from operator import *
>>> a=[1,2,3]
>>> b=['a','b','c']
>>> getitem(b,1)
'b'
>>> getslice(a,1,3)
[2, 3]
>>> setitem(b,1,'d')
>>> b
['a', 'd', 'c']
>>> setslice(a,1,3,[4,5])
>>> a
[1, 4, 5]
从序列中删除元素
>>> delitem(b,1)
>>> b
['a', 'c']
>>> delslice(a,1,3)
>>> a
```

**注意** setitem和delitem会原地修改序列，而不会返回值。

* 原地操作符
除了标准操作符之外，很多对象类型还通过一些特殊操作符支持原地修改。

```
>>> a=-1
>>> b=5
>>> c=[1,2,3]
>>> d=['a','b','c']
>>> a=iadd(a,b)
>>> a
4
>>> c=iconcat(c,d)
>>> c
[1, 2, 3, 'a', 'b', 'c']
```

## 属性和元素的获取方法
operator模块最特别的特性之一就是获取方法的概念，获取方法是运行时构造的一些可回调对象，用来获取对象的属性或序列的内容，获取方法在处理迭代器或生成器序列的时候特别有用，它们引入的开销会大大降低lambda或Python函数的开销。

```
from operator import *
class MyObj(object):
    def __init__(self, arg):
        super(MyObj, self).__init__()
        self.arg = arg
    def __repr__(self):
        return 'MyObj(%s)' % self.arg

objs = [MyObj(i) for i in xrange(5)]
print "Object:", objs

g = attrgetter("arg")
vals = [g(i) for i in objs]
print "arg values:", vals

objs.reverse()
print "reversed:", objs
print "sorted:", sorted(objs, key=g)
结果

Object: [MyObj(0), MyObj(1), MyObj(2), MyObj(3), MyObj(4)]
arg values: [0, 1, 2, 3, 4]
reversed: [MyObj(4), MyObj(3), MyObj(2), MyObj(1), MyObj(0)]
sorted: [MyObj(0), MyObj(1), MyObj(2), MyObj(3), MyObj(4)]
```

属性获取方法类似于`lambda x, n='attrname':getattr(x,nz）`

元素获取方法类似于`lambda x,y=5:x[y]`

```

from operator import *

l = [dict(val=-1*i) for i in xrange(4)]
print "dictionaries:", l
g = itemgetter("val")
vals  = [g(i) for i in l]
print "values: ", vals
print "sorted:", sorted(l, key=g)

l = [(i,i*-2) for i in xrange(4)]
print "tuples: ", l
g = itemgetter(1)
vals = [g(i) for i in l]
print "values:", vals
print "sorted:", sorted(l, key=g)
结果如下：

dictionaries: [{'val': 0}, {'val': -1}, {'val': -2}, {'val': -3}]
values:  [0, -1, -2, -3]
sorted: [{'val': -3}, {'val': -2}, {'val': -1}, {'val': 0}]
tuples:  [(0, 0), (1, -2), (2, -4), (3, -6)]
values: [0, -2, -4, -6]
sorted: [(3, -6), (2, -4), (1, -2), (0, 0)]
```

除了序列之外，元素获取方法还适用于映射。

## 结合操作符和定制类
operator模块中的函数通过相应操作的标准Python接口完成工作，所以它们不仅适用于内置类型，还适用于用户自定义类型。

```
from operator import *

class MyObj(object):
    def __init__(self, val):
        super(MyObj, self).__init__()
        self.val = val
        return 

    def __str__(self):
        return "MyObj(%s)" % self.val

    def __lt__(self, other):
        return self.val < other.val

    def __add__(self, other):
        return MyObj(self.val + other.val)

a = MyObj(1)
b = MyObj(2)

print lt(a, b)
print add(a,b)
结果如下所示：

True
MyObj(3)
```
## 类型检查
operator 模块还包含一些函数用来测试映射、数字和序列类型的API兼容性。

```
from operator import *

class NoType(object):
    pass

class MultiType(object):
    def __len__(self):
        return 0

    def __getitem__(self, name):
        return "mapping"

    def __int__(self):
        return 0

o = NoType()
t = MultiType()

for func in [isMappingType, isNumberType, isSequenceType]:
    print "%s(o):" % func.__name__, func(o)
    print "%s(t):" % func.__name__, func(t)
结果如下：

isMappingType(o): False
isMappingType(t): True
isNumberType(o): False
isNumberType(t): True
isSequenceType(o): False
isSequenceType(t): True
```

但是这些测试并不完善，因为接口没有严格定义。

## 获取对象方法
使用`methodcaller`可以获取对象的方法。

```
from operator import methodcaller

class Student(object):
    def __init__(self, name):
        self.name = name

    def get_name(self):
        return self.name

stu = Student("Jim")
func = methodcaller('get_name')
print func(stu)   # 输出Jim
还可以给方法传递参数：

f=methodcaller('name', 'foo', bar=1)
f(b)    # return   b.name('foo', bar=1)
```
methodcaller方法等价于下面这个函数：

```
def methodcaller(name, *args,  **kwargs):
      def caller(obj):
            return getattr(obj, name)(*args, **kwargs)
      return caller
```


---
layout: post
title: "Python中的高级数据结构"
category: "Python"
date: 2015-05-14
---

# 介绍

在Python中有四种内建的数据结构，分别是List、Tuple、Dictionary以及Set。大部分情况下，这四种结构都能解决问题，不需要其他类型的数据结构，但有很多高级数据结构可以了解下，以便解决问题的时候可供选择，例如：

* Collection
* Array
* Heapq
* Bisect
* Weakref
* Copy
* Pprint

<!-- more -->

## Collections

collections模块包含了内建类型之外的一些有用的工具，例如Counter、defaultdict、OrderedDict、deque以及nametuple。其中Counter、deque以及defaultdict是最常用的类。
之前一篇文章[《Python数据结构 collections》](http://zhujinliang.cn/blog/python/2015/04/09/python-collections.html)已经详细介绍了。

## Array

array模块定义了一个很像list的新对象类型，不同之处在于它限定了这个类型只能装一种类型的元素。array元素的类型是在创建并使用的时候确定的。

如果你的程序需要优化内存的使用，并且你确定你希望在list中存储的数据都是同样类型的，那么使用array模块很合适。举个例子，如果需要存储一千万个整数，如果用list，那么你至少需要160MB的存储空间，然而如果使用array，你只需要40MB。但虽然说能够节省空间，array上几乎没有什么基本操作能够比在list上更快。
在使用array进行计算的时候，需要特别注意那些创建list的操作。例如，使用列表推导式(list comprehension)的时候，会将array整个转换为list，使得存储空间膨胀。一个可行的替代方案是使用生成器表达式创建新的array。

```
import array
 
a = array.array("i", [1,2,3,4,5])
b = array.array(a.typecode, (2*x for x in a))
```

因为使用array是为了节省空间，所以更倾向于使用in-place操作。一种更高效的方法是使用enumerate：

```
import array
 
a = array.array("i", [1,2,3,4,5])
for i, x in enumerate(a):
    a[i] = 2*x
```

对于较大的array，这种in-place修改能够比用生成器创建一个新的array至少提升15%的速度。
那么什么时候使用array呢？是当你在考虑计算的因素之外，还需要得到一个像C语言里一样统一元素类型的数组时。

```
import array
from timeit import Timer
 
def arraytest():
    a = array.array("i", [1, 2, 3, 4, 5])
    b = array.array(a.typecode, (2 * x for x in a))
 
def enumeratetest():
    a = array.array("i", [1, 2, 3, 4, 5])
    for i, x in enumerate(a):
        a[i] = 2 * x
 
if __name__=='__main__':
    m = Timer("arraytest()", "from __main__ import arraytest")
    n = Timer("enumeratetest()", "from __main__ import enumeratetest")
 
    print m.timeit() # 5.22479210582
    print n.timeit() # 4.34367196717
```

## Heapq

heapq模块使用一个用堆实现的优先级队列。堆是一种简单的有序列表，并且置入了堆的相关规则。
堆是一种树形的数据结构，树上的子节点与父节点之间存在顺序关系。二叉堆(binary heap)能够用一个经过组织的列表或数组结构来标识，在这种结构中，元素N的子节点的序号为2*N+1和2*N+2(下标始于0)。简单来说，这个模块 中的所有函数都假设序列是有序的，所以序列中的第一个元素(seq[0])是最小的，序列的其他部分构成一个二叉树，并且seq[i]节点的子节点分别为 seq[2*i+1]以及seq[2*i+2]。当对序列进行修改时，相关函数总是确保子节点大于等于父节点。

```
import heapq
 
heap = []
 
for value in [20, 10, 30, 50, 40]:
    heapq.heappush(heap, value)
 
while heap:
    print heapq.heappop(heap)
```

heapq模块有两个函数nlargest()和nsmallest()，顾名思义，让我们来看看它们的用法。
类似对一个list排序之后，取前n个最大或最小的元素，同时排序的key是可以指定的。


```
import heapq
 
nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
print(heapq.nlargest(3, nums)) # Prints [42, 37, 23]
print(heapq.nsmallest(3, nums)) # Prints [-4, 1, 2]

两个函数也能够通过一个键参数使用更为复杂的数据结构，例如：

import heapq
 
portfolio = [
{'name': 'IBM', 'shares': 100, 'price': 91.1},
{'name': 'AAPL', 'shares': 50, 'price': 543.22},
{'name': 'FB', 'shares': 200, 'price': 21.09},
{'name': 'HPQ', 'shares': 35, 'price': 31.75},
{'name': 'YHOO', 'shares': 45, 'price': 16.35},
{'name': 'ACME', 'shares': 75, 'price': 115.65}
]
cheap = heapq.nsmallest(3, portfolio, key=lambda s: s['price'])
expensive = heapq.nlargest(3, portfolio, key=lambda s: s['price'])
 
print cheap
 
# [{'price': 16.35, 'name': 'YHOO', 'shares': 45},
# {'price': 21.09, 'name': 'FB', 'shares': 200}, {'price': 31.75, 'name': 'HPQ', 'shares': 35}]
 
print expensive
 
# [{'price': 543.22, 'name': 'AAPL', 'shares': 50}, {'price': 115.65, 'name': 'ACME',
# 'shares': 75}, {'price': 91.1, 'name': 'IBM', 'shares': 100}]
```

来看看如何实现一个根据给定优先级进行排序，并且每次pop操作都返回优先级最高的元素的队列例子。


```
import heapq
 
class Item:
    def __init__(self, name):
        self.name = name
 
    def __repr__(self):
        return 'Item({!r})'.format(self.name)
 
class PriorityQueue:
    def __init__(self):
        self._queue = []
        self._index = 0
 
    def push(self, item, priority):
        heapq.heappush(self._queue, (-priority, self._index, item))
        self._index += 1
 
    def pop(self):
        return heapq.heappop(self._queue)[-1]
 
q = PriorityQueue()
q.push(Item('foo'), 1)
q.push(Item('bar'), 5)
q.push(Item('spam'), 4)
q.push(Item('grok'), 1)
 
print q.pop() # Item('bar')
print q.pop() # Item('spam')
print q.pop() # Item('foo')
print q.pop() # Item('grok')
```


## Bisect

bisect模块能够提供保持list元素序列的支持。它使用了二分法完成大部分的工作。它在向一个list插入元素的同时维持list是有序的。 在某些情况下，这比重复的对一个list进行排序更为高效，并且对于一个较大的list来说，对每步操作维持其有序也比对其排序要高效。

```
假设你有一个range集合：
a = [(0, 100), (150, 220), (500, 1000)]
如果我想添加一个range (250, 400)，我可能会这么做：

import bisect
 
a = [(0, 100), (150, 220), (500, 1000)]
 
bisect.insort_right(a, (250,400))
 
print a 
# [(0, 100), (150, 220), (250, 400), (500, 1000)]
我们可以使用bisect()函数来寻找插入点：

import bisect
 
a = [(0, 100), (150, 220), (500, 1000)]
 
bisect.insort_right(a, (250,400))
bisect.insort_right(a, (399, 450))
print a 
# [(0, 100), (150, 220), (250, 400), (500, 1000)]
 
print bisect.bisect(a, (550, 1200)) # 5
bisect(sequence, item) => index 返回元素应该的插入点，但序列并不被修改。

import bisect
 
a = [(0, 100), (150, 220), (500, 1000)]
 
bisect.insort_right(a, (250,400))
bisect.insort_right(a, (399, 450))
print a # [(0, 100), (150, 220), (250, 400), (500, 1000)]
 
print bisect.bisect(a, (550, 1200)) # 5
bisect.insort_right(a, (550, 1200))
print a # [(0, 100), (150, 220), (250, 400), (399, 450), (500, 1000), (550, 1200)]
新元素被插入到第5的位置。
```


## Weakref

weakref模块能够帮助我们创建Python引用，却不会阻止对象的销毁操作。这一节包含了weak reference的基本用法，并且引入一个代理类。
在开始之前，我们需要明白什么是strong reference。strong reference是一个对对象的引用次数、生命周期以及销毁时机产生影响的指针。strong reference如你所见，就是当你将一个对象赋值给一个变量的时候产生的：

```
>>> a = [1,2,3]
>>> b = a
在这种情况下，这个列表有两个strong reference，分别是a和b。在这两个引用都被释放之前，这个list不会被销毁。

class Foo(object):
    def __init__(self):
        self.obj = None
        print 'created'
 
    def __del__(self):
        print 'destroyed'
 
    def show(self):
        print self.obj
 
    def store(self, obj):
        self.obj = obj
 
a = Foo() # created
b = a
del a
del b # destroyed
```
Weak reference则是对对象的引用计数器不会产生影响。当一个对象存在weak reference时，并不会影响对象的撤销。这就说，如果一个对象仅剩下weak reference，那么它将会被销毁。
你可以使用weakref.ref函数来创建对象的weak reference。这个函数调用需要将一个strong reference作为第一个参数传给函数，并且返回一个weak reference。

```
>>> import weakref
>>> a = Foo()
created
>>> b = weakref.ref(a)
>>> b
一个临时的strong reference可以从weak reference中创建，即是下例中的b()：

>>> a == b()
True
>>> b().show()
None
请注意当我们删除strong reference的时候，对象将立即被销毁。

>>> del a
destroyed
如果试图在对象被摧毁之后通过weak reference使用对象，则会返回None：

>>> b() is None
True
```
若是使用`weakref.proxy`，就能提供相对于weakref.ref更透明的可选操作。同样是使用一个strong reference作为第一个参数并且返回一个weak reference，proxy更像是一个strong reference，但当对象不存在时会抛出异常。

```
>>> a = Foo()
created
>>> b = weakref.proxy(a)
>>> b.store('fish')
>>> b.show()
fish
>>> del a
destroyed
>>> b.show()
Traceback (most recent call last):
  File "", line 1, in ?
ReferenceError: weakly-referenced object no longer exists
```

引用计数器是由Python的垃圾回收器使用的，当一个对象的应用计数器变为0，则其将会被垃圾回收器回收。
最好将weak reference用于开销较大的对象，或避免循环引用。

```
import weakref
import gc
 
class MyObject(object):
    def my_method(self):
        print 'my_method was called!'
 
obj = MyObject()
r = weakref.ref(obj)
 
gc.collect()
assert r() is obj #r() allows you to access the object referenced: it's there.
 
obj = 1 #Let's change what obj references to
gc.collect()
assert r() is None #There is no object left: it was gc'ed.
```
**提示**：只有library模块中定义的class instances、functions、methods、sets、frozen sets、files、generators、type objects和certain object types(例如sockets、arrays和regular expression patterns)支持weakref。内建函数以及大部分内建类型如lists、dictionaries、strings和numbers则不支持。

## Copy()

通过shallow或deep copy语法提供复制对象的函数操作。
shallow和deep copying的不同之处在于对于混合型对象的操作(混合对象是包含了其他类型对象的对象，例如list或其他类实例)。

对于shallow copy而言，它创建一个新的混合对象，并且将原对象中其他对象的引用插入新对象。
对于deep copy而言，它创建一个新的对象，并且递归地复制源对象中的其他对象并插入新的对象中。


```
import copy
 
a = [1,2,3]
b = [4,5]
 
c = [a,b]
 
# Normal Assignment
d = c
 
print id(c) == id(d)          # True - d is the same object as c
print id(c[0]) == id(d[0])    # True - d[0] is the same object as c[0]
 
# Shallow Copy
d = copy.copy(c)
 
print id(c) == id(d)          # False - d is now a new object
print id(c[0]) == id(d[0])    # True - d[0] is the same object as c[0]
 
# Deep Copy
d = copy.deepcopy(c)
 
print id(c) == id(d)          # False - d is now a new object
print id(c[0]) == id(d[0])    # False - d[0] is now a new object
shallow copy (copy())操作创建一个新的容器，其包含的引用指向原对象中的对象。
deep copy (deepcopy())创建的对象包含的引用指向复制出来的新对象。
```

## Pprint()

Pprint模块能够提供比较优雅的数据结构打印方式，如果你需要打印一个结构较为复杂，层次较深的字典或是JSON对象时，使用Pprint能够提供较好的打印结果。
假定你需要打印一个矩阵，当使用普通的print时，你只能打印出普通的列表，不过如果使用pprint，你就能打出漂亮的矩阵结构

```
import pprint
 
matrix = [ [1,2,3], [4,5,6], [7,8,9] ]
a = pprint.PrettyPrinter(width=20)
a.pprint(matrix)
 
# [[1, 2, 3],
#  [4, 5, 6],
#  [7, 8, 9]]
```

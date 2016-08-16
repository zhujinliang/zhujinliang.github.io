---
layout: post
title: "Python数据结构 collections"
category: "Python"
date: 2015-04-09
---


# Python数据结构 collections

Python包括很多标准编程数据结构，如list,tuple,dict,set，这些属于内置类型。

`collections`模块包含多种数据结构的实现，扩展了其他模块中相应的结构：

1. deque是一个双端队列，允许从任意一端增加或删除元素。
2. defaultdict是一个字典，如果找不到某个键，会相应一个默认值。
3. OrderedDict会记住增加元素的序列。
4. nametuple扩展了一般的tuple,除了为每个成员元素提供一个数值索引外还提供了一个属性名。
 
<!-- more -->

## Counter
Counter作为一个容器，可以跟踪相同的值增加了多少次。这个类可以用来实现其他语言常用包或多集合数据结构来实现的算法。
 
初始化
Counter支持3种形式的初始化。调用Counter的构造函数时可以提供一个元素序列或者一个包含键和计数的字典，
还可以使用关键字参数将字符串名映射到计数。

    import collections
    print collections.Counter(['a', 'b', 'c', 'a', 'b', 'b'])
    print collections.Counter({'a':2, 'b':3, 'c':1})
    print collections.Counter(a=2, b=3, c=1)
    这三种形式的初始化结构都是一样的。
    >>> ================================ RESTART ================================
    >>> 
    Counter({'b': 3, 'a': 2, 'c': 1})
    Counter({'b': 3, 'a': 2, 'c': 1})
    Counter({'b': 3, 'a': 2, 'c': 1})

如果不提供任何参数，可以构造一个空的Counter，然后通过update()方法填充。

    import collections
    c = collections.Counter()
    print 'Initial  :', c
    c.update('abcdcaa')
    print 'Sequencel:', c
    c.update({'a':1, 'd':6})
    print 'Dict     :', c
    计数值将根据新数据增加，替换数据不会改变计数。
    >>> ================================ RESTART ================================
    >>> 
    Initial  : Counter()
    Sequencel: Counter({'a': 3, 'c': 2, 'b': 1, 'd': 1})
    Dict     : Counter({'d': 7, 'a': 4, 'c': 2, 'b': 1})
 
访问计数
一旦填充了Counter，可以使用字典API获取它的值。

    import collections
    c = collections.Counter('abcdccca')
    for letter in 'abcde':
        print '%s : %d' % (letter, c[letter])
    对于未知元素，Counter不会产生KerError，如果没有找到某个值，其计数为0。
    >>> ================================ RESTART ================================
    >>> 
    a : 2
    b : 1
    c : 4
    d : 1
elements()方法返回一个迭代器，将生产Counter知道的所有元素

    import collections
    c = collections.Counter('abcdccca')
    c['e'] = 0
    print c
    print list(c.elements())
    不能保证元素顺序不变，另外计数小于或等于0的元素不包含在内。
    >>> ================================ RESTART ================================
    >>> 
    Counter({'c': 4, 'a': 2, 'b': 1, 'd': 1, 'e': 0})
    ['a', 'a', 'c', 'c', 'c', 'c', 'b', 'd']

使用most_common()可以生成一个序列，其中包含n个最常遇到的输入值及其相应计数。

    import collections
    c = collections.Counter()
    with open(r'd:\check_traffic.sh', 'rt') as f:
              for line in f:
                  c.update(line.rstrip().lower())
    print 'Most common:'
    for letter, count in c.most_common(5):
              print '%s: %6d' % (letter, count)
    统计系统所有单词中出现的字母，生成一个频率分布，然后打印5个最常见的字母。
    >>> ================================ RESTART ================================
    >>> 
    Most common:
     :   6535
    e:   3435
        :   3202
    t:   3141
    i:   3100
 
算术操作
Counter实例支持算术和集合操作来完成结果的聚集。

    import collections
    c1 = collections.Counter(['a', 'a', 'c', 'b' ,'a'])
    c2 = collections.Counter('alphabet')
    print 'c1:', c1
    print 'c2:', c2
    print '\nCombined counts:'
    print c1 + c2
    print '\nSubtraction:'
    print c1 - c2
    print '\nIntersection:'
    print c1 & c2
    print '\nUnion:'
    print c1 | c2
    每次通过操作生成一个新的Counter时，计数为0或者负的元素都会被删除。
     
    >>> ================================ RESTART ================================
    >>> 
    c1: Counter({'a': 3, 'c': 1, 'b': 1})
    c2: Counter({'a': 2, 'b': 1, 'e': 1, 'h': 1, 'l': 1, 'p': 1, 't': 1})
     
    Combined counts:
    Counter({'a': 5, 'b': 2, 'c': 1, 'e': 1, 'h': 1, 'l': 1, 'p': 1, 't': 1})
     
    Subtraction:
    Counter({'a': 1, 'c': 1})
     
    Intersection:
    Counter({'a': 2, 'b': 1})
     
    Union:
    Counter({'a': 3, 'c': 1, 'b': 1, 'e': 1, 'h': 1, 'l': 1, 'p': 1, 't': 1})
 
## defaultdict

标准字典包括一个方法setdefault()来获取一个值，如果值不存在则建立一个默认值。
defaultdict初始化容器是会让调用者提前指定默认值。

    import collections
    def default_factory():
        return 'default value'
    d = collections.defaultdict(default_factory, foo = 'bar')
    print 'd:', d
    print 'foo =>', d['foo']
    print 'var =>', d['bar']
     
    只要所有键都有相同的默认值，就可以使用这个方法。
    >>> ================================ RESTART ================================
    >>> 
    d: defaultdict(<function default_factory at 0x0201FAB0>, {'foo': 'bar'})
    foo => bar
    var => default value
 
## deque

deque(两端队列)支持从任意一端增加和删除元素。常用的两种结果，即栈和队列，就是两端队列的退化形式，其输入和输出限制在一端。
    
    import collections
    d = collections.deque('abcdefg')
    print 'Deque:', d
    print 'Length:', len(d)
    print 'Deft end', d[0]
    print 'Right end', d[-1]
    d.remove('c')
    print 'remove(c):', d
    deque是一种序列容器，支持list操作，可以通过匹配标识从序列中间删除元素。
    >>> ================================ RESTART ================================
    >>> 
    Deque: deque(['a', 'b', 'c', 'd', 'e', 'f', 'g'])
    Length: 7
    Deft end a
    Right end g
    remove(c): deque(['a', 'b', 'd', 'e', 'f', 'g'])
     
填充
deque可以从任意一端填充，在python实现称为“左端”和“右端”。

    import collections
    d1 = collections.deque()
    d1.extend('abcdefg')
    print 'extend:', d1
    d1.append('h')
    print 'append:', d1
    d2 = collections.deque()
    d2.extendleft(xrange(6))
    print 'extendleft', d2
    d2.appendleft(6)
    print 'appendleft', d2
    extendleft()迭代处理其输入，对每个元素完成与appendleft()相同的处理。
    >>> ================================ RESTART ================================
    >>> 
    extend: deque(['a', 'b', 'c', 'd', 'e', 'f', 'g'])
    append: deque(['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'])
    extendleft deque([5, 4, 3, 2, 1, 0])
    appendleft deque([6, 5, 4, 3, 2, 1, 0])
 
利用
可以从两端利用deque元素，取决于应用的算法。

    import collections
    print "From the right:"
    d = collections.deque('abcdefg')
    while True:
        try:
            print d.pop(),
        except IndexError:
            break
    print
    print "\nFrom the left:"
    d = collections.deque(xrange(6))
    while True:
        try:
            print d.popleft(),
        except IndexError:
            break
    print
    使用pop()可以从deque右端删除一个元素，使用popleft()可以从deque左端删除一个元素。
    >>> ================================ RESTART ================================
    >>> 
    From the right:
    g f e d c b a
     
    From the left:
    0 1 2 3 4 5

由于双端队列是线程安全的，可以在不同的线程中同时从两端利用队列的内容。

    import collections
    import threading
    import time
    candle = collections.deque(xrange(5))
    def burn(direction, nextSource):
        while True:
            try:
                next = nextSource()
            except IndexError:
                break
            else:
                print '%8s: %s' % (direction, next)
                time.sleep(0.1)
        print '%8s done' % direction
        return
    left = threading.Thread(target=burn, args=('Left', candle.popleft))
    right = threading.Thread(target=burn, args=('Right', candle.pop))
    left.start()
    right.start()
    left.join()
    right.join()
    线程交替处理两端，删除元素，知道这个deque为空。
    >>> ================================ RESTART ================================
    >>> 
        Left: 0   Right: 4
     
       Right: 3    Left: 1
     
       Right: 2    Left done
     
       Right done
 
旋转
deque另外一个作用可以按照任意一个方向旋转，而跳过一些元素。

    import collections
    d = collections.deque(xrange(10))
    print 'Normal:', d
    d= collections.deque(xrange(10))
    d.rotate(2)
    print 'Right roration:', d
    d = collections.deque(xrange(10))
    d.rotate(-2)
    print 'Left roration:', d
     
    >>> ================================ RESTART ================================
    >>> 
    Normal: deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
    Right roration: deque([8, 9, 0, 1, 2, 3, 4, 5, 6, 7])
    Left roration: deque([2, 3, 4, 5, 6, 7, 8, 9, 0, 1])
 
## namedtuple

标准tuple使用数值索引来访问其成员。
nametuple实例与常规元祖在内存使用方面同样高效，因为它们没有各实例的字典
。各种nametuple都是由自己的类表示，使用nametuple()工厂函数来创建。
参数就是一个新类名和一个包含元素名的字符串。
    
    import collections
    Person = collections.namedtuple('Persion', 'name age gender')
    print 'Type of Person:', type(Person)
    bob = Person(name='Bob', age=30, gender='male')
    print '\nRepresentation:', bob
    jane = Person(name='Jane', age=28, gender='female')
    print '\nField by name:', jane.name
    print '\nField by index:'
    for p in [bob, jane]:
        print '%s is a %d year old %s' %p
 
## OrderedDict
OrderedDict是一个字典子类，可以记住其内容增加的顺序。

    import collections
    print 'Regular dictionary:'
    d = {}
    d['a'] = 'A'
    d['b'] = 'B'
    d['c'] = 'C'
    for k, v in d.items():
        print k, v
    print '\nOrderDict:'
    d = collections.OrderedDict()
    d['a'] = 'A'
    d['b'] = 'B'
    d['c'] = 'C'
    for k, v in d.items():
        print k, v
    常规dict并不跟踪插入顺序，迭代处理会根据键在散列表中存储的顺序来生成值。在OrderDict中则相反，它会记住元素插入的顺序，并在创建迭代器时使用这个顺序。
    >>> ================================ RESTART ================================
    >>> 
    Regular dictionary:
    a A
    c C
    b B
    OrderDict:
    a A
    b B
    c C

常规dict在检查相等性是会查看其内容，OrderDict中还会考虑元素增加的顺序。
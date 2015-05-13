---
layout: post
title: "jQuery使用on代替delegate,live 用法区别"
category: "JavaScript"
date: 2014-01-12
---


早期对页面上后期加载的动态元素,赋事件或值的时候,是使用`live`的.
由于效率比较低(其实数据不多也感觉不出来),后面使用`delegate`委托来代替了,再后面,1.7以后使用`on` 来代替`delegate`了.
它们在写法上有差别.

如点击div里的任意一个button时增加一个新button：

页面：

    <div id="panel">
          <input type="button" name="name"  value="clone" class="submit-btn" />
    </div>


<!-- more -->

### 使用live
是通过冒泡的方式来绑定到元素上的。
要等事件冒泡到document之后，才会检查事件是否是click，事件的目标元素是否css选择器绑定元素，若两者都符合，则执行绑定的事件。

jQuery版本1.3+

    $('.submit-btn').live('click', function () {
        $(this).clone().appendTo('#panel');
    });


若直接把live改成on, 没有给范围比如#panel,这对页面上一开始有的按钮有效.
也就是说无法直接这样代替live

    $('.submit-btn').on('click', function () {
        $(this).clone().appendTo('#panel');
    });

### 使用delegate

使用delegate 需要给它一个范围才行，如#panel,让它到里面找.这样可以实现live一样的效果.

jQuery版本1.4.3+

    $('#panel').delegate('.submit-btn', 'click', function () {
        $(this).clone().appendTo('#panel');
    });

相比live，delegate的性能更优。

### 使用on

使用on 给它一个范围才行，如#panel,让它到里面找. 这样可以实现live和delegate一样的效果.

里面的'click', '.submit-btn'跟上面的delegate是相反的.只要记住on click是挨在一起的即可.

jQuery版本1.7+

    $('#panel').on('click', '.submit-btn', function () {
        $(this).clone().appendTo('#panel');
    });

[参考博客](http://www.cnblogs.com/sniper007/archive/2011/11/18/2254260.html)
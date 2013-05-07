---
layout: post
title: "Test Code Highlighting and Photos"
description: ""
category: "test"
tags: [jekyll]
---
{% include JB/setup %}

首先测试代码高亮。

{% highlight javascript %}
/**
  * Demonstration arguments is a special variable.
  */
function sum() {
  var i, x = 0;
  for (i = 0; i < arguments.length; ++i) {
    x += arguments[i];
  }
  return x;
}
sum (1, 2, 3); // returns 6
{% endhighlight %}

再测试图片显示。图片源有很多选择，这里试一下又拍网。

[![celebrities drawn in-pencil 03](http://pic.yupoo.com/seanlv/CJMvdKK2/small.jpg)](http://www.yupoo.com/photos/seanlv/88085316/)

效果不错！

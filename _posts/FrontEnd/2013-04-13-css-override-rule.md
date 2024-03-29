---
layout: post
title: "CSS样式覆盖规则"
date: 2013-04-13
tag: CSS
---



CSS的全称叫做“层叠样式表”，但估计很多人都不知道“层叠”二字的含义。
其实，“层叠”指的就是样式的覆盖，当一个元素被运用上多种样式，并且出现重名的样式属性时，
浏览器必须从中选择一个属性值，这个过程就叫“层叠”。样式覆盖（这种叫法更大众化些）遵循一定的规则，

首先需要明确的是，很多情况都会导致一个元素被运用上多种样式，样式覆盖的规则也需要根据不同的情况来定，具体规则如下。

### 规则一：由于继承而发生样式冲突时，最近祖先获胜。

CSS的继承机制使得元素可以从包含它的祖先元素中继承样式，考虑下面这种情况：


    <html>
    <head>
    <title>rule 1</title>
    <style>
    body {color:black;}
    p {color:blue;}
    </style>
    </head>
    <body>
        <p>welcome to <strong>gaodayue的网络日志</strong></p>
    </body>
    </html>

strong分别从body和p中继承了color属性，但是由于p在继承树上离strong更近，因此strong中的文字最终继承p的蓝色。

<!-- more -->

### 规则二：继承的样式和直接指定的样式冲突时，直接指定的样式获胜。

在上面的例子中，假如还指定了strong元素的样式，如：

    strong {color:red;}

那么根据规则二，strong中的文字最终显示为红色。

### 规则三：直接指定的样式发生冲突时，样式权值高者获胜。

样式的权值取决于样式的选择器，权值定义如下表。

|CSS选择器|权值|
|--------|----|
|标签选择器|1|
|类选择器|10|
|ID选择器|100|
|内联样式|1000|
|伪元素(:first-child等)|1|
|伪类(:link等)|10|

可以看到，内联样式的权值 > ID选择器 > 类选择器 > 标签选择器，除此以外，后代选择器的权值为每项权值之和，
比如`#nav .current a`的权值为100 + 10 + 1 = 111。

### 规则四：样式权值相同时，后者获胜。

考虑下面这种情况


    <p class="byline">Written by <a class="email" href="mailto:jean@cosmofarmer. com">Jean Graine de Pomme</a></p>

    .byline a {color:red;}
    p .email {color:blue;}

`.byline a`与`p .email`都直接指定了上面的a元素，且权值都为11，根据规则四，最终显示蓝色。

由于样式表可以是外部的，也可以是内部的，规则四提醒我们要注意外部样式表引入的顺序（及<link>元素的顺序），
以及外部样式表与内部样式表的出现位置。一般来说，内部样式表出现在所有外部样式表的引入之后，一般是在</head>之前。
所以内部的样式会覆盖外部的样式。

### 规则五：!important的样式属性不被覆盖。

`!important`可以看做是万不得已的时候，打破上述四个规则的”金手指”。如果你一定要采用某个样式属性，而不让它被覆盖的，可以在属性值后加上!important，以规则四的例子为例，`.byline a {color:red !important;}`可以强行使链接显示红色。大多数情况下都可以通过其他方式来控制样式的覆盖，不能滥用!important。


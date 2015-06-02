---
layout: post
title: "Django Form验证"
category: "Django"
date: 2013-11-02
---


Django框架提供了一个forms类，来处理web开发中的表单相关事项。众所周知，form最常做的是对用户输入的内容进行验证，
为此Django的forms类提供了全面的内容验证支持。熟悉之后，使用Django的Form做表单验证，非常方便。

## 完整的Form验证过程

一个完整的form验证过程如下：

1. 函数`full_clean()`依次调用每个field的`clean()`函数，该函数针对field的max_length，unique等约束进行验证，如果验证成功则返回值，否则抛出`ValidationError`错误。如果有值返回，则放入form的cleaned_data字典中。

2. 如果每个field的内置`clean()`函数没有抛出`ValidationError`错误，则调用以`clean_`开头，以field名字结尾的自定义field验证函数。验证成功和失败的处理方式同步骤1。

3. 最后，调用form的clean()函数——注意，这里是form的clean(),而不是field的clean()——如果clean没有错误，那么它将返回`cleaned_data`字典。

4. 如果到这一步没有`ValidationError`抛出，那么cleaned_data字典就填满了有效数据。否则`cleaned_data`不存在，form的另外一个字典`errors`填上验证错误。在template中，每个field获取自己错误的方式是：`{{ form.username.errors }}`。

5. 最后，如果有错误`is_valid()`返回`False`，否则返回`True`。

<!-- more -->

## 验证：
分为多个层次,检测失败会返回 ValidationError

        to_python() 　　　　[forms.Field] 转换成python的类型 使用范例
        validate() 　　　　 [forms.Field] 针对特别字段验证，又不想放到验证器当中，不返回值
        run_validators() 　 运行所有字段级别验证，收集错误，不需要改写
        字段级别clean() 　调用上面3项验证，一旦有错，验证停止，否则返回 clean data 字典
        表单级别clean_<fieldname>() 　验证特别的字段
        表单级别clean() 　负责整个表单的验证，手工返回 self.cleaned_data



Refer to： [Django form][django document form]

[django document form]: https://docs.djangoproject.com/en/1.4/ref/forms/validation/



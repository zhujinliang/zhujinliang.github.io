---
layout: post
title: "代码提交规范"
category: "Project Management"
---


# 代码提交规范

当开发团队中多人合作时，每个人都有自己的代码提交习惯，当合作人员多时，必然会导致混乱，所以代码提交规范的提出势在必 行。本文参考了国内外的一些开发人员的建议，总结了一个简单可行的代码提交规范。

## Commit规范
一个commit 应该包括一个准确的逻辑改动。一个逻辑改动包括增加新的特性或修改特定的Bug，如果不能在较高层次用简短语句描述这个改动，就说明它对于单个commit 来说太复杂了，应该拆分它！把你的代码改动拆分为多个 commit。

每次commit 的差异应该尽可能的简练，每个人都更愿意阅读一串连续的、逻辑清楚、差异精炼的补丁，一般看到那种大段大段复杂改动的commit就会头疼。

基本的法则：其他开发开发人员只看你提交的commit信息（不看具体的代码），应该可以在合理的时间内实现几乎相同的补丁程序。

以下习惯，有则改之，无则加冕：

* 总是在每天下班前commit今天所有的改动，这种commit不会清晰。
* 懒汉式的commit信息，如“修改了几个Bug”，这种信息毫无意义。
* 两个改动放在一次 commit 中。如“修改用户注册不成功的处理和用户登录的验证”，请把它拆开分别 commit。
* 有些动词用现在时，而不是完成时，例如不要说“改好了(fixed)…，解决了(solved)…, 增加了(added)…，修改了(revised)…”，而应该说“增加(add)…，修改(revise)…，解决(solve)…”，用一种已经完成的口气，总是让人感觉有点自大。

<!-- more -->
## Commit信息规范

一个好的提交信息的规范，可以使得我们的提交日志的信息可读性更好，以下是一些基本的约定：

1. 所有的提交信息用现在时语气而不是完成时语气，例如使用“增加(add)…，修改(fix)…，解决(solve)…”，而不是“改好了(fixed)…，解决了(solved)…, 增加了(added)…，修改了(revised)…”。
2. 提交的信息，尽量用英语表述，如果实在用英语表述不清楚的，可以用中文。
3. 第一行是commit信息概述，控制50个字符（或25个汉字）以内。结尾统一都不要使用句号，因为后面通常都会有详细描述。

    在第一行的commit信息中，可以加一些前缀，它可以帮助我们对commit进行分类，同时当我们使用git log来检查，或者查找某一个提交的，非常有用。可用的前缀有：
    
    1) `ADD`：增加新的功能，新的单元测试等。
    
    2) `REV`：修改代码的实现，增强了代码的健壮性，重构代码，优化一些功能的细节等。
    
    3) `FIX`：修复某一个bug。
    
    4) `DEL`：删除一些文件。
    
    5) `FMT`：修改主要是提高代码的规范，格式化原来的代码，例如去掉一些空格、空行，将Tab替换成空格等，没有修改代码的功能。
    
    针对上述的几个前缀，举几个例子：
    
    1) `[REV]` Update all app download qcode picture
    
    2) `[FIX]` Fix sms urls that can not access
    
    3) `[ADD]` Add user register account by email
    
    4) `[DEL]` Delete the duplicate template file
    
    5) `[FMT]` Format the evaluate module code

4. 第二行是空行，一些处理工具会把第一行作为邮件的标题，其它的作为正文，所以需要一个空行来区分标题和正文。如果后面没有信息详述，此空行可以省略。
5. 第三行开始是commit 信息详述，每行的长度控制在 72个字符(或 36个汉字)之内。该信息应该要保证足够的详细，开发人员看了之后可以可以在合理的时间内实现几乎相同的补丁程序。
6. 若有bug跟踪软件，对于每一次的bug修复的提交，commit信息中都应该包含bug的ID，commit信息分为两部分，第一部分是提交的概述，第二部分是对该次bug提交的详细描述，例如：

        [FIX] bugfix <bug_id>
        
        Bug <bug_id> - Something goes wrong




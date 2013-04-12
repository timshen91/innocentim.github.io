---
layout: post
title: "Coursera Prolang总结1(SML)"
date: 2013-03-23 9:33
comments: true
categories: Coursera SML
---

Coursera的[Programming Languages](https://class.coursera.org/proglang-2012-001/class/index)是个难得的好课. 虽然课时较短, 时间仓促, 但是它确实提供了高密度的知识吸收, 并且让我更谦卑. Instructor是University of Washington的Dan Grossman, 一个讲课非常热情的老师(横向地, Compilers的老师就没这么激情:P). 这门课通过分析三门语言 : SML, Racket 和 Ruby, 介绍了很多程序语言概念, 从而让我对语言设计积攒了更多经验值. 这绝对比自己看语言们的tutorials划得来.

下面是我从SML的学习中获得的经验的不完全列表:

### 一种更为神奇的变量绑定方式

函数对非参数变量的捕捉被设计为在定义时捕捉好值, 此后便不再改变(更immutable). 这使得函数在执行时根本不关心外围环境, 更优雅, 更利于分析和优化(当然也更inexpressive).

### Pattern Matching

Pattern Matching很像一个constructor的逆运算, 也许可以称作destructor :). 这里的constructor当然不是主流语言中那么随意的一个函数, 而是产生新类型的一种方式. 一个constructor创建一种新的类型, 并可以用BNF风格的语法定义该类型.

一个典型的例子是定义`List 'a := EmptyList | Cons 'a * ('a List)`. 这样便定义了一个范型类型`List`, 而该类型有两个构造函数, 即`EmptyList`和`Cons`.

这种构造函数本身没什么逻辑可言, 就是负责类型的转换. 但是就是因为足够简单, 他们是可逆的 : `Cons`负责包装一个`'a * ('a List)`(star表示元组), 而在模式匹配中, 轻易地, `Cons (head, tail) => someList`将轻易地拆分someList这个元组并将相应成员填充到指定变量head和tail中.

我们知道BNF可以描述上下文无关文法, 强大到足以描述很多语言; 这恰恰为SML提供了很强的数据结构包装器和解包器.

### 非常赞的类型系统

SML拥有"每个函数体是一个表达式"和"每个表达式有确定的返回类型"这种强约束. 这带来的直接后果是写SML代码比较难编译通过, 但是一旦编译通过就很少有错误.

为了更expressive, SML有范型系统. 不像C++丑陋的模板系统, SML能完美地推断出每个类型参数是什么或者哪类类型, 每个函数调用填入了什么类型, 将哪个类型参数绑定到哪个具体类型上了. 这极大地弥补了强类型系统的completeness而且没怎么降低soundness.

关于在强约束下做函数定义的类型推断, 请[参看这篇](/blog/2013/04/12/coursera-prolang-2/)...

顺带说一句, SML主流的缩进风格让我非常难受 : 他们很容易让我写出很靠右的代码(这种"一句话描述函数体"的语言似乎都有这个毛病). 相比之下C/C++规整的indent level更吸引人. 我还想起了Emacs有个叫SmartTab的插件倡导"用Tab表示indent level, 用空格对齐"噗.

Dan is awesome!

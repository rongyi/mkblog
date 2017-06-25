---
title: "Lisp书籍推荐"
date: 2017-06-25T10:26:32+08:00
draft: false
---
<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. 简介</a></li>
<li><a href="#sec-2">2. 开发环境</a></li>
<li><a href="#sec-3">3. 入门读物</a>
<ul>
<li><a href="#sec-3-1">3.1. 《The Little Lisper》</a></li>
<li><a href="#sec-3-2">3.2. 《ANSI Common Lisp》</a></li>
</ul>
</li>
<li><a href="#sec-4">4. 锻炼提高</a>
<ul>
<li><a href="#sec-4-1">4.1. 《Land of Lisp》</a></li>
<li><a href="#sec-4-2">4.2. 《On Lisp》</a></li>
<li><a href="#sec-4-3">4.3. 《Let Over Lambda》</a></li>
</ul>
</li>
</ul>
</div>
</div>


# 简介<a id="sec-1" name="sec-1"></a>

ESR在他的那本《Unix编程艺术》中提到hackers需要学一下Lisp，他的用词是迟早都要。自己的经验是此言非虚，并且大家都知道YC的创办合伙人PG也是Lisp的死忠，他的随笔《Hackers and Painters》中更是把Lisp的发明人Jhon Macathy看做和欧拉比肩的人物（参考他的paper root of Lisp），所以说看下Lisp没有坏处。

下文列出的书籍顺序也是推荐的阅读顺序，因为这个顺序也包含了自己的经验总结。

# 开发环境<a id="sec-2" name="sec-2"></a>

个人建议在Emacs下配置lisp开发环境。Emacs结合slime是非常好的Lisp学习开发环境。

# 入门读物<a id="sec-3" name="sec-3"></a>

## 《The Little Lisper》<a id="sec-3-1" name="sec-3-1"></a>

学习一下递归思想的入门读物

## 《ANSI Common Lisp》<a id="sec-3-2" name="sec-3-2"></a>

PG写的Lisp介绍类的入门读物

# 锻炼提高<a id="sec-4" name="sec-4"></a>

## 《Land of Lisp》<a id="sec-4-1" name="sec-4-1"></a>

强烈推荐的一本书，作者以开发游戏的方式来循循善诱从小的游戏到一点点把她积攒起来到最后可以用web来玩耍。时间充裕的话是可以读第二遍的，不仅仅是因为内容，更是因为有趣。

## 《On Lisp》<a id="sec-4-2" name="sec-4-2"></a>

重点在这个介词on上面，这也是函数式编程以及SICP中提倡的那种解决问题的方法、工程的方法：打散再组合。打散即分模块切分并实现，组合即模块间通过协议来通信。作者在这本书里提出自底向上实现的好处，一个是对开发morale（士气）的鼓舞，一个是模块测试往上堆叠带来的信心，而这个迭代的过程现在又叫TDD，不管叫它什么，作者用Lisp作为实践语言来强调这个观点。到最后介绍把Lisp和其他语言区分开来的macro。Macro有点编译器的味道执行的时候先展开成一段代码然后再执行。又两层意境。个中体会只有自己去体会。这里文字显得有些苍白。

## 《Let Over Lambda》<a id="sec-4-3" name="sec-4-3"></a>

作者也是个Macro的拥拓，这本书对《On Lisp》大加赞赏，所以开篇就用到《On Lisp》中的一些Macro。所以建议阅读放在《On Lisp》之后，不过他喜欢用Vim，附录对这块有些说明，提到的点我觉得还是蛮中肯的，因为Emacs需要调教，调教的时间如果太长就容易掉进所谓的Emacs trap，我对这个的理解要深入一点，因为我掉进去过。所以现在不再一味的折腾，而是要满足开发需求，刀再锋利不用就没发挥它的价值。


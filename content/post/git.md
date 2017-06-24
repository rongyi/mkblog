---
title: "Git"
date: 2017-06-24T19:48:04+08:00
draft: false
---

<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">Git简介</a></li>
<li><a href="#sec-2">历史</a></li>
<li><a href="#sec-3">Git设计原则</a></li>
<li><a href="#sec-4">内部实现</a>
<ul>
<li><a href="#sec-4-1">Content 保存</a></li>
<li><a href="#sec-4-2">存储模型</a></li>
</ul>
</li>
<li><a href="#sec-5">快速上手</a>
<ul>
<li><a href="#sec-5-1">标记自己</a></li>
<li><a href="#sec-5-2">搞到repo</a></li>
<li><a href="#sec-5-3">修改/提交</a></li>
<li><a href="#sec-5-4">显示日志</a></li>
<li><a href="#sec-5-5">分支相关</a></li>
</ul>
</li>
<li><a href="#sec-6">Git带来的开发模型</a></li>
<li><a href="#sec-7">git碎片整理</a>
<ul>
<li><a href="#sec-7-1">找出一个committer修改过的所有的文件</a></li>
</ul>
</li>
<li><a href="#sec-8">emacs中的magit</a></li>
<li><a href="#sec-9">from </a></li>
</ul>
</div>
</div>


# Git简介<a id="sec-1" name="sec-1"></a>

# 历史<a id="sec-2" name="sec-2"></a>

05年由Linux本人发明，伴随着Kernel的发展而产生。Kernel的头十年一直用 `diff/patch` 进行代码管控，后由商业公司BitMove赞助，将其公司产品BitKeeper供Kernel免费使用，后由于社区有人对BitKeeper进行逆向工程惹怒其母公司从而终止了BitKeeper对Kernel的授权。Linus因为不想回到 `diff/patch` 时代遂在当时寻找一种替代方案，他寻找的条件有两个：
-   分布(distributed)
-   可靠(reliable)

遗憾的是当时没有一款管控工具入他的法眼，于是，他写了一个，命名为Git。

# Git设计原则<a id="sec-3" name="sec-3"></a>

上述的筛选条件就是Git的设计原则：
-   Distributed
-   Reliable
-   High Performance

Linux在Google的Tech Talk上讲Git时提到：

> If you are not distributed, you are not woth using
>
> &#x2013; Linus

可见他把分布式看得有多重，因为要支持Kernel开发所以必须要支持可以在offline仍然可以开发并且在联网时可以方便提交的一种机制，而且在传输过程中保证数据的完整性，在这两个前提下保证对版本操作的高速。

# 内部实现<a id="sec-4" name="sec-4"></a>

以本人学习Git的经验在了解Git内部数据存储模型后会更好的掌握上层的操作命令。

需要说明以下几个基本概念：
-   repository

    一个版本信息的集合，往往对应于一个project
-   Git directory

    Git用来保存内部数据的文件夹，一般为 `.git` 文件夹
-   Working directory

    工作目录，一个工程所在的目录，一般指 `.git` 文件夹同级的位置。

Git的数据类型有：
-   blob
-   tree
-   tag
-   commit

需要记住两个大前提：
1.  Git Never see files, only the content.
2.  No Delta

## Content 保存<a id="sec-4-1" name="sec-4-1"></a>

Git管控的最小基本单位是blob，而这个最小单位 **不包括** 文件名信息，仅仅与文件内容相关，也即如果有多份文件，而每份文件的内容都相同，这份内容在Git内部仅被保存一次。文件名保存在文件所在目录信息里。目录对应Git的存储结构是tree，如果你熟悉 `ext fs` 的话，对这样的说明一定不陌生。

那怎么保证reliable？Git从最基本的文件做起，它保存文件内容的步骤：

以被保存的文件内容为输入生成SHA1序列，一次SHA1序列为key将内容通过zlib压缩后作为此key的value保存，映射到文件系统中就是以key作为文件名，value写入此文件中，具体实现对40位的前两位建文件夹后38位建文件(在前两位的文件夹中)，将内容写入这38位的文件中。

SHA1计算的具体实现：
可以在Git源码中 `sha1_file.c:write_sha1_file_prepare` 中找到具体计算的过程。之前有讲过只跟文件内容相关，但不全是文件内容，详细的计算输入是：

    |类型String|空格|文件内容长度|\0|文件内容|

这里有个问题：为什么要区分类型？如果只有一种类型不是更简洁更优雅吗？问题的答案在理解一下介绍的存储模型就清楚了。

## 存储模型<a id="sec-4-2" name="sec-4-2"></a>

Git保存的是snapshot而不是delta，这样的好处是不必关心中间的任何状态。以svn为例，从revision 1234 到2345的diff操作需要从1234到2345的所有delta patch过来然后通过网络传输过来，对于Git而言每次提交都完整保存stage中的状态，提交与提交之前有联系通过这个联系穿梭与版本之间。

举个示例：

    mkdir demo && cd demo && git init
    echo "Amour" > demo.txt
    git add .
    git commit -m "just a demo"

上命令就完成了第一次提交，执行下列命令查看输出：

    find .git/objects/ -type f |sort
    .git/objects/71/688ac8b8587aaf8818dc0c55d62e22e75b22ef
    .git/objects/c5/df312b024f98725ca13e4135225c640c67da24
    .git/objects/cd/6f44469f436230a1b6285b7b545d0738386ae6

    git cat-file -t 7168
    tree
    git cat-file -t c5df
    commit
    git cat-file -t cd6f
    blob

    git cat-file -p  7168
    100644 blob cd6f44469f436230a1b6285b7b545d0738386ae6 demo.txt

    git cat-file -p  c5df
    tree 71688ac8b8587aaf8818dc0c55d62e22e75b22ef
    author rongyi <yi.rong@yamutech.com> 1360044012 +0800
    committer rongyi <yi.rong@yamutech.com> 1360044012 +0800

    just a demo

    git cat-file -p  cd6f
    Amour

可以看见提交过程中的一些信息都保存在了Git object中。而各object之间的联系可以通过下图来描述。

![img](/images/git_first.png)

第二此提交些内容：

    echo "new year" > 2013.txt
    git add 2013.txt
    git commit -m "happy new year"

存储模型如下：

![img](/images/git_second.png)

这张图解释了为什么之前Git要分类型的问题，因为Git内部的存储模型为单向无环图(Directed Acyclic Graph)，分类型的原因即在于要维系好这个模型！如果在计算SHA1序列的过程中 **仅仅** 依赖文件内容的话，那从理论上可以构造出任意和tree，commit等SHA1序列一样的序列，这样这个模型一旦存有环就跨掉了。所以Git内部在计算SHA1序列时会在头部硬塞些内容来保正从用户的输入永远都不可能构造出存在环的情况，保重了Git的reliable。可以理解为我们在使用Git时永远都处于Git的用户态。

# 快速上手<a id="sec-5" name="sec-5"></a>

## 标记自己<a id="sec-5-1" name="sec-5-1"></a>

    git config --global user.name "大名"
    git config --global user.email "邮箱"
    git config --global color.ui true
    git config --global alias.ls status
    git config --global merge.tool meld
    git config --global core.editor vim

## 搞到repo<a id="sec-5-2" name="sec-5-2"></a>

自己造？

    git init || git init --bare

别人的？

    git clone git@github.com/someone/somerepo.git

## 修改/提交<a id="sec-5-3" name="sec-5-3"></a>

    git add file.txt
    git commit -m "fix XXX"

## 显示日志<a id="sec-5-4" name="sec-5-4"></a>

    git log -1
    git log -p
    git log -1 -p
    git show HEAD
    git show HEAD~1
    git log --grep="keystring" #查询某次提交含有"keystring"的log
    git blame file.txt #same as svn blame

## 分支相关<a id="sec-5-5" name="sec-5-5"></a>

分支在Git里是强烈推荐使用的，一般书上对Git中的分支都有两个字形容： **extreamly lightweight** ，本质就是在 `refs/heads` 中添加个文件，然后文件内容写上某次提交的SHA1序列。

    git branch newbranch
    git checkout -b newbranch
    git checkout newbranch
    #make change and commit
    git checkout develop
    git merge --no-ff newbranch

# Git带来的开发模型<a id="sec-6" name="sec-6"></a>

[这里](http://nvie.com/posts/a-successful-git-branching-model/)介绍的很详细，不赘述。

# git碎片整理<a id="sec-7" name="sec-7"></a>

## 找出一个committer修改过的所有的文件<a id="sec-7-1" name="sec-7-1"></a>

    git log --no-merges --stat --author="rongyi" --name-only --pretty=format:""""

# emacs中的magit<a id="sec-8" name="sec-8"></a>

不得不说magit大大提高的git的使用速度. Howard Abrams作了一个非常好的 [demo](https://www.youtube.com/watch?v%3DvQO7F2Q9DwA). 推荐.

# from <https://csswizardry.com/2017/05/little-things-i-like-to-do-with-git/><a id="sec-9" name="sec-9"></a>

    # show commit summary
    git shortlog -sn

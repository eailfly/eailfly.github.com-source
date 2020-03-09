Title: 将hexo项目置与版本控制下
Date: 2016-10-16 17:29:29
Tags:


[转自][Hexo 博客搭建教程（三）：Hexo 博客代码版本控制](http://www.iamsuperman.cn/2016/02/22/hexo/hexo-guide-3/)
由于 Hexo 只会将生成后的 public 文件夹部署到 github 上，导致无法对博客进行代码版本控制。同时如果需要备份代码的话，只能通过其他手段来实现。
本文介绍了如何利用 github 分支对代码进行版本控制，同时起到备份代码的作用。

解决思路
实现 Hexo 博客代码版本控制以及备份的思路如下：
通过新建一个source分支用于专门存放 hexo 代码。原先的master分支依然不变，作为 hexo 部署的分支。
每次在部署后，再把代码提交到source分支。


1、本地创建 git 仓库

``` bash
git init
```

2、添加远程库

``` bash
git remote add origin <git repository url>
```

3、创建 source 分支

``` bash
git checkout -b source
```

4、提交文件及分支，并 push 到远程仓库

``` bash
git add *
git commit -m 'message'
git push origin source
```

其中source为分支名称。

这样就建立了代码版本控制分支。之后只要将博客在部署到 github 之后，将代码 push 到source分支上。代码如下：

``` bash
git add *
git commit -m "udpate site"
git push origin source
```

问题记录
如果你使用了第三方主题，在进行代码提交的时候，是无法将第三方主题提交到你的 github repository 中，会出现 untracked content的提示。
这是因为第三方主题本身也是一个 git 项目。你无法将别人的 git 项目直接通过 add 和 commit 的方式提交到你自己的 git repository。
也就说，你无法提交处于 untracked状态的文件。
解决办法：

添加 submodule的方式，将主题作为 submodule 提交到你的 git repository
删除主题文件夹下的.git文件夹。如果这时候还不能提交，可以新建个文件夹，随便命名，将主题文件夹内的东西复制到新建的文件夹。再通过git add提交就可以了。

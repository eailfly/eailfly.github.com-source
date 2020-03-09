Title: 使用alternatives管理多版本软件
Date: 2014-03-09


今天装you-get的时候发现需要python3，安装python3的时候发现了python的路径是这样的

``` bash
> ll /usr/bin/python
lrwxrwxrwx 1 root root 24 3月   9 13:28 /usr/bin/python -> /etc/alternatives/python
> ll /etc/alternatives/python
lrwxrwxrwx 1 root root 18 3月   9 15:25 /etc/alternatives/python -> /usr/bin/python2.7
```

不知道为什么linux要这样处理，好像以前也碰到过这个现象，但是当时都是直接做的链接，这次google了一下才知道这是一个多版本软件管理的工具。

将python2和python3注册到alternatives中

``` bash
> update-alternatives --install /usr/bin/python python /usr/bin/python2.7 100
> update-alternatives --install /usr/bin/python python /usr/bin/python3.3 101
```

然后看看注册结果：

``` bash
> sudo update-alternatives --config python
There are 2 choices for the alternative python (providing /usr/bin/python).

Selection    Path                Priority   Status
0            /usr/bin/python3.3   6         auto mode
* 1            /usr/bin/python2.7   5         manual mode
2            /usr/bin/python3.3   6         manual mode

Press enter to keep the current choice[*], or type selection number:
```

这里输入你要的版本的序号就可以了。

Title: 忘记RHEL7 root密码的找回办法
Date: 2016-12-16 13:46:02
Tags:


RHEL7引导时采用grub2，与以前的RHEL版本采用grub1，在引导时差别较大。
进行单用户模式的方式也有一定差异。

在RHEL7中，进入单用户模式的方法如下：

    1. 开机到grub2时，用上下键移到用上下键移到菜单中所要使用的启动项，按e（注意不是回车）
    2. 在"linux16 /vmlinuz-3.10.0 ..."这一行的最后面加入"rw single init=/bin/bash",然后按Ctl+X或F10, 就可以进入 单用户模式，进去干什么都行了。可以改普通用户密码，也可以改root密码。
    3. 输入Ctl+Alt+Del重启。

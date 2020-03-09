Title: 自动挂载U盘
Date: 2015-06-06


## 安装

``` bash
pacman -S udisks
pacman -S udevil
systemctl enable devmon@eailfly.service
```

注意：devmon 后的启动参数为用户名。
U盘将被装载到 /media 文件夹。

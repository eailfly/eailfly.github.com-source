Title: Arch使用ApricityOS主题
Date: 2016-06-13


## 安装 ApricityOS 源

``` bash
wget http://apricityos.com/apricity-core-signed/apricity-icons-1.0.0-1-any.pkg.tar.xz
yaourt -U apricity-icons-1.0.0-1-any.pkg.tar.xz
```

## 编辑 `/etc/pacman.conf` 文件
``` conf
[apricity-core]
SigLevel = Required
Server = http://apricityos.com/apricity-core-signed/
```

更新源数据库

``` bash
yaourt -Syu
```

## 安装主题

``` bash
yaourt -S apricity-themes-gnome apricity-icons apricity-keyring apricity-wallpapers
```

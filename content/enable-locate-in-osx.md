Title: 在 OSX 中使用 locate 命令
Date: 2018-07-18 14:58:27
Tags:


从 OS X Lion 开始 OS X 系统内置了 `locate` 命令，如果没有启用的话在第一次执行 `locate` 时系统会提示如何启用：

```sh
  $ locate python

  WARNING: The locate database (/var/db/locate.database) does not exist.
  To create the database, run the following command:

  sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.locate.plist

  Please be aware that the database can take some time to generate; once
  the database has been created, this message will no longer appear.
```
执行
```sh
  sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.locate.plist
```
即可启用

如果需要执行 `updatedb`，需执行 `sudo /usr/libexec/locate.updatedb`，或创建符号链接
```sh
  ln -s /usr/local/bin/updatedb /usr/libexec/locate.updatedb
```

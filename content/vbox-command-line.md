Title: vbox 命令行
Date: 2013-11-03 16:49:23
Tags:

## 创建虚拟机

``` bash
VBoxManage createvm –name "openSUSE" –register
```
创建一个名为openSUSE的虚拟机

## 查看虚拟机

``` bash
VBxoManage list vms
```

## 查看虚拟机状态

``` bash
VBoxManage showvminfo suse
```

## 修改虚拟机设置

### 创建磁盘

``` bash
VBoxManage createhd --filename /home/virtualbox/suse.vdi --size 8000 --remember
```

### 修改操作系统类型

``` bash
VBoxManage modifyvm "suse" --ostype "suse"
```

### 设置内存以及显存大小

``` bash
VBoxManage modifyvm "suse" --memory "1024" --vram "64"
```

### 添加一个IDE接口 (SATA之类的也可以)

``` bash
VBoxManage storagectl winxp --name "IDE Controller" --add ide
```

### 设置启动顺序及挂载一个磁盘

``` bash
VBoxManage modifyvm "suse" --boot1 dvd --hda "/home/virtualbox/suse.vdi" --sata on
```

### 把磁盘放在设备0的第0个端口

``` bash
VBoxManage storageattach winxp --storagectl "IDE Controller" --port 0 --device 0 --type hdd --medium /home/virtualbox/suse.vdi
```

### 挂载ISO

``` bash
VBoxManage storageattach winxp --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium /home/virtualbox/suse.iso
```

## 启动系统

``` bash
startvm <name> [--type gui|sdl|headless]
```

三种模式，gui不用说了，sdl这个没装qt环境的时候用，跟gui差不多，headless这个是不用图形界面（这个是最爽的）

## 共享剪切板

``` bash
modifyvm <name> [--clipboard disabled|hosttoguest|guesttohost|bidirectional]

disabled 不共享剪贴板
hosttoguest 将宿主机的剪贴板共享给虚拟机
guesttohost 将虚拟机的剪贴板共享给宿主机
bidirectional 宿主机和虚拟机共使用一个剪贴板
```

## 共享文件夹

``` bash
VBoxManage sharedfolder add "suse" -name "shared" -hostpath "/home/xxx/shared"
```

进入系统后

``` bash
mount -t vboxsf share mount_point
```

## 删除共享（虚拟机关闭状态）

``` bash
VBoxManage sharedfolder remove "suse" -name "shared"
```

## 虚拟机控制

``` bash
VBoxManage controlvm <name> pause|resume|reset|poweroff|savestate|

pause 暂停，这时虚拟机窗口显示灰色
resume 恢复暂停的虚拟机
reset 复位
poweroff 强行关闭
acpipowerbutton 关机
acpisleepbutton 使虚拟机处于睡眠状态
savestate 保存状态然后关闭，相当于休眠
```

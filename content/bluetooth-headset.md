Title: Arch GNOME 下配置蓝牙耳机
Date: 2016-05-23


## 安装配置

首先安装包

``` bash
pulseaudio-alsa pulseaudio-bluetooth bluez bluez-libs bluez-utils bluez-firmware
```

启动蓝牙进程

``` bash
systemctl start bluetooth
systemctl enable bluetooth
```

然后进入 `bluetoothctl`

``` bash
power on
agent on
default-agent
scan on
```

耳机在配对模式，很快就能发现设备

``` bash
[NEW] Device B8:AD:3E:5C:43:6E LG HBS750
```

使用MAC地址配对

``` bash
pair B8:AD:3E:5C:43:6E
```

使用MAC地址连接

``` bash
connect B8:AD:3E:5C:43:6E
```

## Troubleshooting

配对成功, 但连接失败

你可能在 bluetoothctl 里面看到下面的错误:

``` bash
[bluetooth]# connect 00:1D:43:6D:03:26
Attempting to connect to 00:1D:43:6D:03:26
Failed to connect: org.bluez.Error.Failed
```

这是因为没有安装pulseaudio-bluetooth 包导致的。 如果确实没有安装，安装一下这个包，然后重启一下PulseAudio

Title: Android消除叹号
Date: 2016-10-14 18:03


更新
Android 7 开始地址变了，使用以下命令：

``` bash
adb shell "settings put global captive_portal_https_url https://www.noisyfox.cn/generate_204"
```

使用adb将/generate204地址设置一下就可以了，可以使用如下命令：

``` bash
adb shell "settings put global captive_portal_server captive.v2ex.co"
```

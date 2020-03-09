Title: Vim分屏操作
Date: 2013-11-18 17:03:22
Tags:


本文摘自[酷壳](http://coolshell.cn/articles/1679.html)

### 分屏启动Vim
1.用大写O垂直分屏

``` bash
vim -On file1 file2
```

2.用小写o水平分屏

``` bash
vim -on file1 file2
```

注：n表示分几屏

### 关闭分屏

1.关闭当前窗口

``` bash
Ctrl+w c
```

2.关闭当前窗口，如果只剩最后一个了，则退出Vim。

``` bash
Ctrl+w q
```

### 分屏

1.上下分割当前打开的文件。

``` bash
Ctrl+W s
```

2.上下分割，并打开一个新的文件。

``` bash
:sp filename
```

3.左右分割当前打开的文件。

``` bash
Ctrl+W v
```

4.左右分割，并打开一个新的文件。

``` bash
:vsp filename
```

### 移动光标

Vi中的光标键是h, j, k, l，要在各个屏间切换，只需要先按一下Ctrl+W

1.把光标移到右边的屏。

``` bash
Ctrl+W l
```

2.把光标移到左边的屏中。

``` bash
Ctrl+W h
```

3.把光标移到上边的屏中。

``` bash
Ctrl+W k
```

4.把光标移到下边的屏中。

``` bash
Ctrl+W j
```

5.把光标移到下一个的屏中。.

``` bash
Ctrl+W w
```

### 移动分屏

这个功能还是使用了Vim的光标键，只不过都是大写。当然了，如果你的分屏很乱很复杂的话，这个功能可能会出现一些非常奇怪的症状。

1.向右移动。

``` bash
Ctrl+W L
```

2.向左移动

``` bash
Ctrl+W H
```

3.向上移动

``` bash
Ctrl+W K
```

4.向下移动

``` bash
Ctrl+W J
```

### 屏幕尺寸

下面是改变尺寸的一些操作，主要是高度，对于宽度你可以使用Ctrl+W <或是>，但这可能需要最新的版本才支持。

1.让所有的屏都有一样的高度。

``` bash
Ctrl+W =
```

2.增加高度。

``` bash
Ctrl+W +
```

3.减少高度。

``` bash
Ctrl+W -
```

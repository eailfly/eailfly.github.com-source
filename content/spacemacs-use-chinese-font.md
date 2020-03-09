Title: 给Spacemacs添加中文字体
Date: 2017-03-01 14:37:46
Tags:


不知道为什么Spacemacs突然不显示中文了，必须得设置一个中文字体才可以，但是中文字体的西文都烂的可以，所以想能不能给中文单独设置一个字体，结果找到了以下方法。
将以下代码添加到 `dotspacemacs/user-config`：

``` lisp
(dolist (charset '(kana han cjk-misc bopomofo))
  (set-fontset-font (frame-parameter nil 'font) charset
                    (font-spec :family "Sans Mono CJK SC" :size 20)))
```

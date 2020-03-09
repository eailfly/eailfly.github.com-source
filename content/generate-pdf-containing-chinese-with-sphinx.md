Title: 用 Sphinx 制作中文 pdf
Date: 2016-05-30


[转自] [用 Sphinx 制作中文 pdf](https://guts.me/2013/06/21/generate-pdf-containing-chinese-with-sphinx/)

最近一直在玩些少众的东西，感觉都是很棒的东西，可惜关于它们的资源比较少，中文的就更少，所以打算翻译其中一些觉得比较好的文档。

其中有些文档是用 Sphinx 制作的，有了文档的源码，翻译好再用 Sphinx 再次生成中文版的就可以。然而在生成 pdf 文档的过程却遇到很多问题。虽然 Sphinx 是用 Python 写的，Python 对 UTF8 编码的支持毋庸置疑，但生成 pdf 的过程是由 LaTeX 套件完成。问题在于 LaTeX 套件从开发之日起就没有考虑过支持如中文这样使用 Unicode 字符的语言。

google 了很久，经过了很多尝试，也在 stackoverflow.com 上问了一些高手，最终得到两个能够要达到这个目的方案。两个方案的效果都不太完美，不过还算差强人意了。

在 Arch Linux 上需要安装的软件包有：

``` bash
pacman -S make python-sphinx texlive-langextra texlive-langcjk
```

安装时使用默认的选项，其他依赖的软件包也会一同被安装上了。安装完后可以使用

``` bash
latex -v
```

来查看具体安装到的 LaTeX 套件版本。

若是 Ubuntu，也需要安装同样的软件包：

``` bash
apt-get install make python-sphinx texlive-full
```

两个方案所述的 pdf 制作方法，不仅适用于中文，也适用于日文和韩文。其实是因为上面安装软件包时选择了 CJK 组件，CJK 就是 Chinese、Japanese、Korean 的缩写

## 方案1、使用 LaTeX 及 CJK 自带的字体

上面安装 LaTeX 套件的时候也安装了 CJK 字体。其中有 4 种中文字体：gbsn(简体宋体)、gkai(简体楷体)、bsmi(繁体明体)和 bkai(繁体楷体)。选择字体的时候一定要简体选简体的字体，繁体选繁体的字体，否则生成的 pdf 文档会出现文字缺失。选择好字体就可以在 conf.py 的 latex_elements 选项中添加相关的配置内容：

``` python
latex_elements = {
    ...
    'preamble': '''
    \\hypersetup{unicode=true}
    \\usepackage{CJKutf8}
    \\AtBeginDocument{\\begin{CJK}{UTF8}{gbsn}}
    \\AtEndDocument{\\end{CJK}}
    '''
}
```

其中 `\hypersetup` 是用于设置生成的 pdf 的书签使用 Unicode 字符，否则会出现乱码。

修改完 conf.py 后，就可以执行 Sphinx 项目的 `make latexpdf` 命令生成 pdf 文档了。

如果在 conf.py 的 latex_document 选项中文档标题的内容或作者的内容里包含有 Unicode 字符，可能会出现下面这些莫名其妙的错误：

``` bash
...
(/usr/share/texmf-dist/tex/latex/cjk/texinput/UTF8/UTF8.bdg)
(/usr/share/texmf-dist/tex/latex/cjk/texinput/UTF8/UTF8.enc)
(/usr/share/texmf-dist/tex/latex/cjk/texinput/UTF8/UTF8.chr)
! Extra \else.
\CJK@XXX ...\number `#1\endcsname {`#2}{`#3}\else
                                              \csname u8:\string #1\stri...
l.32 \tableofcontents

?
```

这是由于 Sphinx 在生成 LaTeX 文档时会用 CJK 字符直接替换掉相应的占位符。而 CJK 字符本身就是宏，不应该直接写入 pdf 流中。因为不想修改 Sphinx 的文档类文件 sphinxmanual.cls，所以采取一个投机取巧的方法来处理。用 \\unexpanded 将 Unicode 字符括起来：

``` python
latex_documents = [
  ('index', 'AnIntroductiontolibuv.tex', u'title \\unexpanded{中文内容}',
    u'\\unexpanded{中文作者}', 'manual'),
]
```

## 方案2、使用 ReTeX 及系统字体

ReTeX 是专门为生成 Unicode pdf 文档而生 LaTeX 组件，使用上与 LaTeX 差不多。区别是 XeTeX 使用的是系统已安装的字体，这可以解决 LaTeX 字体缺乏的问题。但在测试过程中，只在 Ubuntu 上成功生成 pdf 文档，在 Arch Linux 上则出现错误。google 了一下原因大概是 XeCJK 与新版的 LaTeX 不兼容。

使用 ReTeX 的第一步也是确定要使用的字体。使用

``` bash
fc-list :lang=zh
```

来查看系统中已安装的中文字体。

选择好字体后，同样需要修改 conf.py 文件中的 latex_elements 选项：

``` python
latex_elements = {
    ...
    'preamble': '''
    \\usepackage{xeCJK}
    \\usepackage{indentfirst}
    \\setlength{\\parindent}{2em}
    \\setCJKmainfont{WenQuanYi Zen Hei Sharp}
    \\setCJKmonofont[Scale=0.9]{WenQuanYi Zen Hei Mono}
    \\setCJKfamilyfont{song}{WenQuanYi Zen Hei}
    \\setCJKfamilyfont{sf}{WenQuanYi Zen Hei}
    '''
}
```

主要是在 \setCJKmainfont、\setCJKmonofont、\setCJKfamilyfont{song} 和 \setCJKfamilyfont{sf} 最后的大括号里填入想要使用的字体名称，即 fc-list 命令列出的名称。

而使用 XeTeX 时，conf.py 里的 latex_document 选项中的标题及作者内容不需要用 \unexpanded 括起 Unicode 字符。

生成 pdf 时，首先执行 Sphinx 项目下的

``` bash
make latex
```

得到需要的 .tex 文件，如 mybook.tex，进入其中的 build/latex 目录中，执行两次下面的命令

``` bash
xelatex mybook.tex
```

至于为什么要执行两次，在网上能找到的文章都说为了正确生成书签。而我的实验结果是，第一次执行生成的 pdf 文档中没有书签，第二次执行后才生成书签。可能是 XeTeX 具体实现机制的问题吧，不去深究了。

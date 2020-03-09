Title: "使用Spacemacs管理hexo"
Date: 2016-12-27 11:56:08
Tags:


安装
====================
在private layer － package.el 中添加

```lisp
    (hexo :location(recipe
                    :fetcher github
                    :repo "kuanyui/hexo.el"
                    ))
```

melpa 中的版本好像有问题，需要使用github中的版本

在private layer - func.el 中添加

```lisp
    (defun hexo-my-blog ()
        (interactive)
        (hexo "~/my-blog/"))
```

Usage
===========
`M-x hexo` 可以进入 `hexo—mode`

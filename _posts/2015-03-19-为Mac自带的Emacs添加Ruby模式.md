---
layout: post
category : 翻译
tagline: "Supporting tagline"
tags : [Emacs, Ruby mode, 代码高亮, 编辑器]
---
Mac自带的Emacs还是v22，实在是有点老，连现在流行的Package Manager都没有。由于它没有自带的Ruby mode，因此也不会为Ruby提供语法高亮了。不过如果是熟悉Emacs和elisp的大神，自己写个插件也不错。但是对于我这种玩票性质的小白用户来说，实在是太难了。不过幸亏这是一个广为人知的问题，Google上有许多答案。其中Diamond的这篇![Adding Ruby Mode to Emacs in Mac OS X](http://biztech.sheprador.com/?p=49)操作比较简单。下面是它的中文版。THX to Diamond...

1. 下载Ruby源码中的Ruby model el文件。原文为![http://svn.ruby-lang.org/cgi-bin/viewvc.cgi/trunk/misc/ruby-mode.el?view=markup](http://svn.ruby-lang.org/cgi-bin/viewvc.cgi/trunk/misc/ruby-mode.el?view=markup)，不过不好复制，建议大家上Github上找![https://raw.githubusercontent.com/ruby/ruby/fe2440261c6dd0cb0429ec99d1befb53d6c249b9/misc/ruby-mode.el](https://raw.githubusercontent.com/ruby/ruby/fe2440261c6dd0cb0429ec99d1befb53d6c249b9/misc/ruby-mode.el)。
2. 将刚才的el文件保存到`/usr/share/emacs/site-lisp/ruby-mode.el`。
3. 在`/usr/share/emacs/site-lisp/site-start.el`中添加一下代码。

```lisp
;; Load ruby mode when needed
(autoload 'ruby-mode "ruby-mode" "Ruby mode" t )
;; Assign .rb and .rake files to use ruby mode
(setq auto-mode-alist (cons '("\\.rb\\'" . ruby-mode) auto-mode-alist))
(setq auto-mode-alist (cons '("\\.rake\\'" . ruby-mode) auto-mode-alist))
;; Show syntax highlighting when in ruby mode
(add-hook 'ruby-mode-hook '(lambda () (font-lock-mode 1)))
```

如果需要修改默认的Tab键缩进宽度，可以继续添加下面的代码。

```lisp
;; Set the default tab width to 4
(setq-default tab-width 4)
```

下面就可以愉快的使用Emacs练习Ruby了。

> 本文档由**[长沙戴维营教育](http://www.diveinedu.cn)**整理。


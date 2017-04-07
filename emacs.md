# emacs

----

## emacs的基本操作键

    1. 缩写键的对照关系

        + M(eta)，在 Mac 下为 Option 键
        + s(uper)，在 Mac 环境下为左 Command 键
        + S(Shift)
        + C(trl)

    2. 光标的移动是编辑器中最常用的操作所以必须熟知

        + C-f 为前移一个字符， f 代表 forward。
        + C-b 为后移一个字符， b 代表 backward。
        + C-p 为上移至前一行， p 代表 previous。
        + C-n 为上移至下一行， n 代表 next。
        + C-a 为移至行首， a 代表 ahead。
        + C-e 为移至行尾， e 代表 end。

    3. 常用的文件操作快捷键组合也必须熟记。

        + C-x C-f 为打开目标文件， f 代表 find/file
        + C-x C-s 为保存当前缓冲区（Buffer）， s 代表 save

        C-x 是 Emacs 的快捷键中常用的前缀命令。这些前缀命令常常代表了一系列有关联的指 令，十分重要，请特别牢记。其它常见的还有 C-c, C-h 。打断组合键为 C-g ，它 用于终端取消之前的指令。快捷键就是用预先绑定好的方式告诉 Emacs 去执行指定的命令。 （之后会介绍到更多有关绑定的内容） 

    4. 获取帮助

        + C-h k 寻找快捷键的帮助信息
        + C-h v 寻找变量的帮助信息
        + C-h f 寻找函数的帮助信息

    5. 一些基本快捷键

        + C-d 移除光标后一个字符
        + M-d 移除光标后的一个词
        + C-x h 全部选中
        + C-M-\ 对选中区域，按照某种格式(比如C程序)进行格式化
        + C-x C-c 退出Emacs
        + C-g 取消未完成的命令
        + C-z (redefined): Undo；原来C-z是挂起Emacs（然后用fg命令调出）；C-x u 是默认的命令； 移动一下光标，再C-z就可以redo
        + C-x u 再C-z就可以redo
        + C-w (redefined) : 剪切一块区域；如果没有设置mark，则是剪切一行
        + M-w (redefined) : 拷贝一块区域；如果没有设置mark, 则是拷贝一行
        + C-k : 从当前位置剪切到行尾
        + C-y : 粘贴
        + M-y : 用C-y拉回最近被除去的文本后，换成 M-y可以拉回以前被除去的文本。键入多次的M-y可以拉回更早以前被除去的文本。



----


## 简单的编辑器自定义

```lisp
;; 关闭工具栏，tool-bar-mode 即为一个 Minor Mode
(tool-bar-mode -1)

;; 关闭文件滑动控件
(scroll-bar-mode -1)

;; 显示行号
(global-linum-mode 1)

;; 更改光标的样式
(setq-default cursor-type 'bar)

;; 关闭启动帮助画面
(setq inhibit-splash-screen 1)

;; 关闭缩进 (第二天中被去除)
;; (electric-indent-mode -1)

;; 更改显示字体大小 16pt
;; http://stackoverflow.com/questions/294664/how-to-set-the-font-size-in-emacs
(set-face-attribute 'default nil :height 160)

;; 快速打开配置文件
(defun open-init-file()
  (interactive)
  (find-file "~/.emacs.d/init.el"))
;; 这一行代码，将函数 open-init-file 绑定到 <f2> 键上
(global-set-key (kbd "<f2>") 'open-init-file)
;; 关闭备份文件
(setq make-backup-files nil)
;; 选择删除
(delete-selection-mode 1)
;; 启动后全屏
(setq initial-frame-alist (quote ((fullscreen . maximized))))
;; 高亮当前行
(global-hl-line-mode 1)
```

## 配置插件源

```lisp
(when (>= emacs-major-version 24)
    (require 'package)
    (package-initialize)
    (setq package-archives '(("gnu"   . "http://elpa.emacs-china.org/gnu/")
            ("melpa" . "http://elpa.emacs-china.org/melpa/"))))

;; 注意 elpa.emacs-china.org 是 Emacs China 中文社区在国内搭建的一个 ELPA 镜像

;; cl - Common Lisp Extension
(require 'cl)

;; Add Packages
(defvar henry/packages '(
        ;; --- Auto-completion ---
        company
        ;; --- Better Editor ---
        hungry-delete
        swiper
        counsel
        smartparens
        ;; --- Major Mode ---
        js2-mode
        ;; --- Minor Mode ---
        nodejs-repl
        exec-path-from-shell
        ;; --- Themes ---
        monokai-theme
        ;; solarized-theme
        ) "Default packages")

(setq package-selected-packages henry/packages)

(defun henry/packages-installed-p ()
    (loop for pkg in henry/packages
        when (not (package-installed-p pkg)) do (return nil)
       finally (return t)))

(unless (my/packages-installed-p)
    (message "%s" "Refreshing package database...")
    (package-refresh-contents)
    (dolist (pkg henry/packages)
    (when (not (package-installed-p pkg))
        (package-install pkg))))

;; Find Executable Path on OS X
(when (memq window-system '(mac ns))
   (exec-path-from-shell-initialize))
```


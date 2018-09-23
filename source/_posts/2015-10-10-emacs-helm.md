title: emacs helm
tags:
  - emacs
  - helm
date: 2015-10-10 14:07:48
categories:
  - emacs
---


`Helm` has been released a long time, but I am always get used to `ido`.

But based on some article recently, and I am really like new things, so I decided to use the this `helm` package.

So, let's start.

<!--more-->

## Install Helm
[Helm Github](https://github.com/emacs-helm/helm) already tell many things about the install, here I want to memo down what I do.

I always use the subtree to manage the package by myself, so
> &gt; git subtree add --prefix .emacs.d/plugins-subtree/helm-suite/helm --squash https://github.com/emacs-helm/helm.git master

> &gt; git subtree add --prefix .emacs.d/plugins-subtree/helm-suite/emacs-async --squash https://github.com/jwiegley/emacs-async.git master

go the helm folder:
> &gt; make

Before the config step, you can simply use the following command to have a test:
> &gt; ./emacs-helm.sh

This would be a simple test environment, and it's very handy.
![helm-preview.jpg](https://winterttrgithubio.blob.core.windows.net/images/2015-10-10-emacs-helm/helm-preview.jpg)


## Basic Config Helm
Add the package to load path and active the `helm`
```lisp
;; -*- coding: utf-8 -*-
(wttr/plugin:prepend-to-load-path "helm-suite/emacs-async")
(wttr/plugin:prepend-to-load-path "helm-suite/helm")
(require 'helm-config)
```


## Some handy config
Enable global `M-x`
```lisp
(global-set-key (kbd "M-x") 'helm-M-x)
```

The default behavior of `Tab` key is not handy enough, so change it:
```lisp
(define-key helm-map (kbd "<tab>") 'helm-execute-persistent-action) ; rebind tab to run persistent action
(define-key helm-map (kbd "C-i") 'helm-execute-persistent-action) ; make TAB works in terminal
(define-key helm-map (kbd "C-z")  'helm-select-action) ; list actions using C-z
```


## Some issue
Still, grep under windows, here is my solution, please not that, your `grep` under windows may not support `--include` well, so remove `%e` will fix it.
```lisp
(if (executable-find "perl")
    (setq helm-grep-default-command "perl ~/.emacs.d/extra-bin/ack/ack-standalone.pl -Hn --no-group --no-color %p %f"
          helm-grep-default-recurse-command "perl ~/.emacs.d/extra-bin/ack/ack-standalone.pl -H --no-group --no-color %p %f")
  (setq helm-grep-default-command "grep -a -d skip -n -e %p %f"
        helm-grep-default-recurse-command "grep -a -d recurse -n -e %p %f"))
```


## Some handy key
- You can insert marked candidates into current buffer with `C-c C-i`
- you can always switch it to vertical window with `C-t`. Running `C-t` again returns the Helm window back to horizontal and so on
- You can mark candidates with `C-SPC`; this is useful when you need to perform an action on many candidates of your choice. `M-a` to select all.

## further more
Read this article [helm-intro](http://tuhdo.github.io/helm-intro.html)


---

**References**:

- [Helm Emacs Github](https://github.com/emacs-helm/helm)
- [A Package in a league of its own: Helm](http://tuhdo.github.io/helm-intro.html)
- [Helm wiki](https://github.com/emacs-helm/helm/wiki#25-developping-using-helm-framework)
- [Exploring large projects with Projectile and Helm Projectile](http://tuhdo.github.io/helm-projectile.html)

And also:

- [Markdown Check and cross character](http://stackoverflow.com/questions/712132/in-html-i-can-make-a-checkmark-with-x2713-is-there-a-corresponding-x-mark)




---

title: emacs exec-path is not synced with $PATH under MacOS
date: 2015-09-12 16:33:16
tags:
  - emacs
  - MacOS
---

I do not notice this issue when I try to use the `npm` from emacs eshell.
I found that the `npm` can be found from bash in `iTerm2` but cannot be found from emacs eshell.
Then I notice that the emacs do not sync the interal `PATH` with what you can check from `bash`.


<!--more-->

I fact the wiki already mentioned those issues [here](http://emacswiki.org/emacs/EmacsApp).
And also, you can find from the [stackoverflow](http://stackoverflow.com/questions/16676826/making-the-path-and-other-environment-variables-available-in-emacs) with the same question.

Here I want to tell a good package for emacs from stackoverflow:
[exec-path-from-shell](https://github.com/purcell/exec-path-from-shell)

With simple setup, you path could be right:
```lisp
(require 'exec-path-from-shell)
(when (memq window-system '(mac ns))
  (exec-path-from-shell-initialize))
```

Enjoy emacs.


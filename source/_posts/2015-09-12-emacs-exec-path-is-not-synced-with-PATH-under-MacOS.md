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

---
{% raw %}
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
Please refer to the author's name `winterTTr` when you remix, transform, and build upon this material. 
{% endraw %}
本作品由`winterTTr`创作，采用[知识共享署名-非商业性使用 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。修改，参照或者转载请注明作者`winterTTr`



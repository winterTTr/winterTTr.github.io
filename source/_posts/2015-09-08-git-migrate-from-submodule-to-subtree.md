title: git migrate from submodule to subtree
date: 2015-09-08 13:21:24
tags:
  - git
  - git-submodule
  - git-subtree
---

My emacs is mainly using `git submodule` to organise the packages. I do not use any package management for my emacs currently. But recently, i lose the `evil` package git repository access. That makes me to really the potential issue about `git-submodule`. Meanwhile, I found the `git-subtree`, so i decide to move all my packages from `git-submodule` to `git-subtree`.


<!--more-->


# Submodule vs Subtree
First, let's talk about some differences between `submodule` and `subtree`.

## submodule = main repository + SHA1 of sub repositories

When you clone you main repository, you have all you files in your main repository and the submodule record file `.gitmoduless`.

You need to use `git submodule init` to initialize the submodle configuration and `git submodule update` will check out the exact version of the sub repository *from their remote git service*. Of course, you can do all of these in one command, `git clone --recursive (repository)`.

In fact, `.gitmoduless` only store the information of  remote repositories, such as 
```
[submodule ".emacs.d/plugins/ace-jump-mode"]
    path = .emacs.d/plugins/ace-jump-mode
    url = https://github.com/winterTTr/ace-jump-mode.git
    pushurl = git@github.com:winterTTr/ace-jump-mode.git
[submodule ".emacs.d/plugins/evil"]
    path = .emacs.d/plugins/evil
    url = git://gitorious.org/evil/evil.git
[submodule ".emacs.d/plugins/expand-region"]
    path = .emacs.d/plugins/expand-region
    url = https://github.com/magnars/expand-region.el.git
```


And git stores SHA1 of commit for all the sub repositories directly in Git's object database. You can check the content with
> git ls-tree master:[path-to-directory-containing-submodule]

For example:

> ~ $ git ls-tree master .emacs.d/plugins/ace-jump-mode
> 160000 commit 8351e2df4fbbeb2a4003f2fb39f46d33803f3dac	.emacs.d/plugins/ace-jump-mode

So here is one of the potential issue for `git-submodule`, when you lose the access of other submodule remote git server or your remote server delete the commits with that SHA1 you want to reference, you cannot give your environment setup correctly again if you clone your main repository in new environment.

This is exact issue i met now.




## subtree = main repository + snapshots of sub repositoies
When you clone your repository, you will get all the code no matter you main repository and sub repositoies. In fact, all the sub repository code will be commmitted to your main repository as normal files. So, this would be like a local snapshot of the sub repository. You clone your repository and you got all you should have for your whole environment.

Even though you cannot access some of the sub repository git service, you do not need to care about that because you already have the workable snapshot in your repository. The only case you need to access those remote service is when you need to pull their latest change and merge to your local snapshots, or you want to push your local change back to sub repositories remote git server.

This gives us several benifits:
1. you will not depends on other git service when you clone yourself. And the other contributors do not even need to know if there is a upstream repository existing.
3. you can make changes to your snapshot as if it is your code and you can track all the history.

And what `git-subtree` do is to help you check out this snapshots and merge upstream changes and etc.

So personally, I would consider `git-subtree` as a way to manage your sub repositories.


# Detail Steps

## Remove submodule

It's a pity that we do not have a `git submodule rm`, so remove a submodule will be manual steps.

For a submodule, there are many places that record the relevant information, we need to clean them as expected to remove submodule gracefully:
- `.gitmodules`
Where your git folder and remote server is stored. Manually edit to remove submodule information.
- `.git/config`
After you do the `git submodule init`, git read the information from `.gitmodules` and save it here to track local submodule. You can use `git submoudle deinit [submodule-path]` to remove that module from this file. One thing need to pay attention to, if you use `git module deinit`, it will also remove submodule from working tree.
- `.git/modules/[submodule path]`
Here is where git fetch and save the remote sub module tree to. Need manually delete. If you do not delete it, it would report error when you want to add the same module as submodule again.
- `submodule file in working tree`
You can simply do this by `rm -rf [submodule path]`

So the steps are:

1. clean `.gitmodules`
> Manually edit this file to remove relevant git submodule section
> &gt; git add .gitmodules

2. clean `.git/config`
> manually delete submodule section in `.git/config`

3. clean files in `index/staged`
> git rm --cached [submodule path]

4. clean `.git/modules/[submodule path]`
> &gt; rm -rf .git/modules/[submodule path]

5. commit your change
> &gt; git commit -m "remove submodule xxxx"

6. clean local if necessary
> &gt; rm -rf [submodule path]

Here is my example:

> ~ $ ff .gitmodules
>
>     #<buffer .gitmodule>
>
> ~ $ git add .gitmodules
> ~ $ ff .git/config
>
>     #<buffer config>
>
> ~ $ git rm --cached .emacs.d/plugins/ace-jump-mode
>
>     rm '.emacs.d/plugins/ace-jump-mode'
>
> ~ $ git status
>
>     On branch master
>     Your branch is ahead of 'origin/master' by 1 commit.
>       (use "git push" to publish your local commits)
> 
>     Changes to be committed:
>       (use "git reset HEAD <file>..." to unstage)
> 
>       deleted:    .emacs.d/plugins/ace-jump-mode
>       modified:   .gitmodules
> 
> ~ $ git commit -m "remove submodule ace-jump-mode"
> 
>     [master 24a9e7d] remove submodule ace-jump-mode
>      Committer: winterTTr <winterTTr@gmail.com>

## Use subtree to obtain snapshot

The common process to get remote repository into subtree:
1. add remote branch to local track
> &gt; git remote add ace-jump-mode https://github.com/winterTTr/ace-jump-mode.git

2. get the latest version of the code into working tree
> &gt; git subtree add --prefix .emacs.d/plugins-subtree/ace-jump-mode --squash ace-jump-mode master
>
>     git fetch ace-jump-mode master
>     From https://github.com/winterTTr/ace-jump-mode
>      * branch            master     -> FETCH_HEAD
>     Added dir '.emacs.d/plugins-subtree/ace-jump-mode'

3. for further update from remote server
> &gt; git subtree pull --prefix .emacs.d/plugins-subtree/ace-jump-mode --squash ace-jump-mode



---

**References**:

- [GIT SUBMODULES EXPLAINED](http://longair.net/blog/2010/06/02/git-submodules-explained/)
- [Git Submodule Tutorial](https://git.wiki.kernel.org/index.php/GitSubmoduleTutorial)
- [How to use Git submodules](http://blog.joncairns.com/2011/10/how-to-use-git-submodules/)
- [“git rm --cached x” vs “git reset head — x”?](http://stackoverflow.com/questions/5798930/git-rm-cached-x-vs-git-reset-head-x)
- [Where does Git store the SHA1 of the commit for a submodule?](http://stackoverflow.com/questions/5033441/where-does-git-store-the-sha1-of-the-commit-for-a-submodule)
- [How do I remove a Git submodule?](http://stackoverflow.com/questions/1260748/how-do-i-remove-a-git-submodule)
- [Git Tools - Subtree Merging](https://git-scm.com/book/en/v1/Git-Tools-Subtree-Merging)
- [Alternatives To Git Submodule: Git Subtree](http://blogs.atlassian.com/2013/05/alternatives-to-git-submodule-git-subtree/)
- [Mastering Git subtrees](https://medium.com/@porteneuve/mastering-git-subtrees-943d29a798ec)
- [Git Subtree](http://git-memo.readthedocs.org/en/latest/subtree.html)

**Markdown Tips**:
- If you want to use `<` in your markdown block, you need to escape it as `&lt;`

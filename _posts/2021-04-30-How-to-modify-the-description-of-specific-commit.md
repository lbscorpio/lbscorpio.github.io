---
title:  How to modify the description of a specific commit after pushing
date: 2021-04-30 14:35:21
categories:
  - Procedure
tags:
  - github
  - opensource
  - git
---

Sometimes I found that some commit using the wrong description after the commit already pushed to remote repository, we can use command `git commit --amend` to update the last commit. But if not the last one?

## Locate the commit you want to update

To update the description of some specific commit, you need to find it firstly.

    $ git log

## Rebase the commit

If the specific commit was the last `n` commit, issuing the following command:

    $ git rebase -i HEAD~n

Update the command from `pick` to `edit` on the specific commit, then hit `Esc` and enter `:wq` to exit git(vim) environment.

    Example:
    edit 24dbd75 Minor changes on the 18th article
    pick 1a40b2c Added one reference URL on the 19th article
    pick c1c55d6 Added the 21th article
    
    # Rebase d8b5c20..77b6cee onto d8b5c20 (5 commands)
    #
    # Commands:
    # p, pick <commit> = use commit
    # r, reword <commit> = use commit, but edit the commit message
    # e, edit <commit> = use commit, but stop for amending
    # s, squash <commit> = use commit, but meld into previous commit
    # f, fixup <commit> = like "squash", but discard this commit's log message
    # x, exec <command> = run command (the rest of the line) using shell
    # b, break = stop here (continue rebase later with 'git rebase --continue')
    # d, drop <commit> = remove commit
    # l, label <label> = label current HEAD with a name
    # t, reset <label> = reset HEAD to a label
    # m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
    # .       create a merge commit using the original merge commit's
    # .       message (or the oneline, if no original merge commit was
    # .       specified). Use -c <commit> to reword the commit message.
    #
    # These lines can be re-ordered; they are executed from top to bottom.
    #
    # If you remove a line here THAT COMMIT WILL BE LOST.
    #
    # However, if you remove everything, the rebase will be aborted.
    #

## Amend the description of the specific commit

Update the new description by issuing the following command:

    $ git commit --amend

Once you are satisfied with your changes, run:

    $ git rebase --continue

## Push the latest update to remote repository

    $ git push origin master --force

> For the shared repository, have better to make sure no new commits from others before you force pushing.

## Reference

<a href="https://www.jianshu.com/p/098d85a58bf1" target="_blank">Git 修改已提交的commit注释</a>  

> **Copyright Notice**: Original work by this blog’s author, licensed under <a href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank">CC BY-NC-ND 4.0</a>. When reposting, please include a link to the original source and this copyright notice.
---
title: Git 存储与恢复
toc: true
tags:
    - git
categories:

---


## <div id="_git_checkout">git checkout</div>
TODO


## <div id="_git_reset">git reset</div>
TODO


## <div id="_git_stash_command">git stash</div>
在开发中经常遇到这种情况，在 `dev` 分支上开发新功能时，有新的 bug 需要马上解决，此时新功能尚未开发完，不想增加一个脏提交，这时可以使用 `git stash` 命令。`git stash` 用于保存和恢复工作进度。

### <div id="_git_stash_list">查看已保存进度列表</div>
使用 `git stash list` 查看已保存的进度的列表：
```shell
$ git stash list
stash@{0}: WIP on master: bb764d6 Add locale youtube download shell
stash@{1}: On master: third stash
stash@{2}: On master: second stash
stash@{3}: On master: first stash
```

### <div id="_git_stash_save">保存进度</div>
`git stash` 保存所有 **已跟踪** 文件的修改，但不保存 **未跟踪** 的文件（git 文件状态请参考[记录每次更新到仓库](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%AE%B0%E5%BD%95%E6%AF%8F%E6%AC%A1%E6%9B%B4%E6%96%B0%E5%88%B0%E4%BB%93%E5%BA%93)）。

```shell
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test1.md
        new file:   test2.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        test.md

$ git stash
Saved working directory and index state WIP on master: bb764d6 Add locale youtube download shell

$ git status
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        test.md

nothing added to commit but untracked files present (use "git add" to track)

$ git stash
No local changes to save
```

从以上示例可以看到，未跟踪的文件 test.md 不会被存储起来，使用 `git stash` 保存未追踪的文件系统会提示 “No local changes to save”。因此在保存进度前最好用 `git add .` 命令将所有文件添加到暂存区，避免保存进度时丢失新文件。

实际开发时，**禁止** 直接使用 `git stash` 命令，而是使用 `git stash save "message"` 保存进度并添加进度说明。禁止理由如下：有时保存了工作进度，但相关功能暂时不需要了，添加进度说明可以方便日后快速查看指定进度的目标，不用再查看代码反推当时为何添加这些修改。相信我，如果你时隔几个月再去查看以前保存的某个进度，一定会赞同此说法。

因此，一次典型保存工作进度的流程如下：
```shell
$ git add .
$ git stash save "We must add stash message"
Saved working directory and index state On master: We must add stash message
$ git stash list
stash@{0}: On master: We must add stash message
```

### <div id="_git_stash_restore">恢复进度</div>

#### <div id="_git_stash_pop">恢复并删除进度</div>
恢复进度并且删除进度： <code>git stash pop [--index] [<stash\>]</code>

```shell
$ git stash list
stash@{0}: On master: save test2.md
stash@{1}: On master: save test1.md
stash@{2}: On master: save test.md

$ git stash pop
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test2.md

Dropped refs/stash@{0} (bf0fba50dd944507668419b6502acceb935eb262)

$ git stash list
stash@{0}: On master: add test1.md
stash@{1}: On master: add test.md

$ git stash pop stash@{1}
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test.md
        new file:   test2.md

Dropped stash@{1} (ecee645f4fff9e493fa56dd6df9eccd579e90114)

$ git stash list
stash@{0}: On master: add test1.md
```

#### <div id="_git_stash_apply">恢复但不删除进度</div>
恢复进度但不删除进度： <code>git stash apply [--index] [<stash\>]</code>
```shell
$ git stash list
stash@{0}: On master: add test2.md
stash@{1}: On master: add test1.md
stash@{2}: On master: add test.md

$ git stash apply
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test2.md

$ git stash list
stash@{0}: On master: add test2.md
stash@{1}: On master: add test1.md
stash@{2}: On master: add test.md

$ git stash apply stash@{1}
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test1.md
        new file:   test2.md

$ git stash list
stash@{0}: On master: add test2.md
stash@{1}: On master: add test1.md
stash@{2}: On master: add test.md
```

### <div id="_git_stash_drop">删除进度</div>
删除指定进度： <code>git stash drop [<stash\>]</code>
```shell
$ git stash list
stash@{0}: On master: add test2.md
stash@{1}: On master: add test1.md
stash@{2}: On master: add test.md

$ git stash drop
Dropped refs/stash@{0} (579fcf40727d31a15c76065ff465ef66d8ba4805)

$ git stash list
stash@{0}: On master: add test1.md
stash@{1}: On master: add test.md

$ git stash drop stash@{1}
Dropped stash@{1} (561d3c994a0d1e4415483776fcaa0d5db42929ff)

$ git stash list
stash@{0}: On master: add test1.md
```

删除所有进度： <code>git stash clear</code>
```shell
$ git stash list
stash@{0}: On master: add test2.md
stash@{1}: On master: add test1.md
stash@{2}: On master: add test.md

$ git stash clear

$ git stash list
```

**注意：** 删除进度时，修改的内容也会被删除。
